## http请求篇

上一章节我们介绍了`bpmn.js`的一些基础知识点以及介绍了在`vue`是如何使用的, 要是对`bpmn.js`不了解的小伙请移步:

这一章节主要讲解的是关于`bpmn.js`如何与后台进行交互的问题, 通过学习此章节你可以学习到:

[通过http请求获取数据并渲染](#通过http请求获取数据并渲染)

[将编辑之后的最新bpmn发送给后台](将编辑之后的最新bpmn发送给后台)

[编辑完保存为bpmn文件或svg文件](编辑完保存为bpmn文件或svg文件)

### 通过http请求获取数据并渲染

在之前的案例中使用的一直都是本地写死的一个`xml`字符串, 那么实际使用上肯定不会以这种方式.

我们团队现在采用的做法是:

- 前端发起请求, 获取到一个`bpmn`文件的地址
- 拿到地址之后, 使用`axios`请求这个地址得到`xml`的字符串(这里命名为`bpmnXmlStr`)
- 使用`importXML`方法将字符串转化为图形并渲染.

为了模拟上面的执行环境我接着上一章节的项目案例[bpmn-vue-basic](https://github.com/LinDaiDai/bpmn-vue-basic)在components文件夹下创建一个`axios.vue`的 文件并配置好路由:

```javascript
const routes = [
	...
	{
		path: '/axios',
		component: () => import('./../components/axios')
	}
]
```

同时在项目中安装`axios`以用于前端发送`http`请求:

```javascript
npm i axios --save-D
```

首先在`HTML`代码中作出一个`loading`的效果, 用户前端在获取到`xml`之前的一个展示:

```vue
// axios.vue
<template>
  <div class="containers">
    <div class="loading" v-if="loading">
        Loading...
    </div>
    <template v-else>
        <div class="canvas" ref="canvas"></div>
        <div id="js-properties-panel" class="panel"></div>
    </template>
  </div>
</template>
```

然后在`js`中引入`axios`并定义一个`getXmlUrl`方法模拟获取`bpmn`文件地址:

```vue
// axios.vue
<script>
...
import axios from 'axios'
import { xmlStr } from '../mock/xmlStr' // 引入一个本地的xml字符串, 若是没有获取到后台的数据则用它

export default {
	...
	data () {
	   return {
		...
		loading: true,
        xmlUrl: '',
        defaultXmlStr: xmlStr
		}
	},
	methods: {
  	async init () {
      this.loading = true
      this.xmlUrl = await this.getXmlUrl()
      console.log(this.xmlUrl)
      this.loading = false
      this.$nextTick(() => { // 等待 DOM 更新之后再对工作流进行初始化
      	this.initBpmn()
    	})
    },
    getXmlUrl () { // 该方法模拟请求后台获取bpmn文件地址
        return new Promise(resolve => {
            setTimeout(() => {
                const url = 'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmnMock.bpmn' // 模拟网络请求的一个地址
                resolve(url)
            }, 1000)
        })
    },
    initBpmn () {
      ... // 这里是初始化工作流的代码
      this.createNewDiagram()
    },
    async createNewDiagram () {
        const that = this
        let bpmnXmlStr = ''
        if (this.xmlUrl === '') { // 若是后台没有数据则使用默认的一个xml
            bpmnXmlStr = this.defaultXmlStr
            this.transformCanvas(bpmnXmlStr)
        } else {
            let res = await axios({
                method: 'get',
                timeout: 120000,
                url: that.xmlUrl,
                headers: { 'Content-Type': 'multipart/form-data' }
            })
            console.log(res)
            bpmnXmlStr = res['data']
            this.transformCanvas(bpmnXmlStr)
        }
    },
    transformCanvas(bpmnXmlStr) {
      // 将字符串转换成图显示出来
      this.bpmnModeler.importXML(bpmnXmlStr, (err) => {
        if (err) {
          console.error(err)
        } else {
          this.success()
        }
        // 让图能自适应屏幕
        var canvas = this.bpmnModeler.get('canvas')
        canvas.zoom('fit-viewport')
      })
    },
    success () {
        console.log('创建成功!')
    }
    }
}
</script>
```

你可以直接用我在案例中模拟获取地址的那个路径:

[https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmnMock.bpmn](https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmnMock.bpmn)

案例Git地址: [LinDaiDai-bpmn.js测试案例axios.vue](https://github.com/LinDaiDai/bpmn-vue-basic/blob/master/src/components/axios.vue)



### 将编辑之后的最新bpmn发送给后台

上面我们介绍了如何从后台那里拿到数据并渲染到页面上, 但这样是不够的. 

可能你需要将编辑之后的最新`bpmn`存储到后台.

该功能就涉及到了`bpmn.js`中的事件绑定, 也就是前端需要给图形绑定一个事件来检测到图形的改变, 并获取到最新的`xml` 信息.

新建一个`save.vue`文件并将`axios.vue`里的内容复制进来.

在`success()`方法中添加一个`addBpmnListener()`绑定事件的方法:

```vue
// save.vue
<script>
   success () {
    	console.log('创建成功!')
    	this.addBpmnListener()
  	},
		// 添加绑定事件
    addBpmnListener () {
      const that = this
      // 给图绑定事件，当图有发生改变就会触发这个事件
      this.bpmnModeler.on('commandStack.changed', function () {
        that.saveDiagram(function(err, xml) {
          console.log(xml) // 这里获取到的就是最新的xml信息
        })
      })
    },
    // 下载为bpmn格式,done是个函数，调用的时候传入的
    saveDiagram(done) {
      // 把传入的done再传给bpmn原型的saveXML函数调用
      this.bpmnModeler.saveXML({ format: true }, function(err, xml) {
        done(err, xml)
      })
    }
</script>

```

如图所示:

![img2](https://user-gold-cdn.xitu.io/2019/12/10/16eeeabc3342306d?w=3238&h=1794&f=jpeg&s=664111)

案例Git地址: [LinDaiDai-bpmn.js测试案例save.vue](https://github.com/LinDaiDai/bpmn-vue-basic/blob/master/src/components/save.vue)



### 编辑完保存为bpmn文件或svg文件

在上面我们监听`commandStack.changed`事件就能实时获取到最新的`xml`信息.

拿到这些信息之后你可以选择在每次图形改变的时候就请求给后台传递给他们最新的`xml`; 

也可以选择将其保存到一个变量中, 然后在页面中给一个保存按钮, 当点击按钮的时候再传递给后台.

或许你可能完全不需要再请求给后台, 而是希望本地就能够下载为`bpmn`文件或者`svg`文件.

在上面`save.vue`案例的基础上增加两个保存按钮:

![img3](https://user-gold-cdn.xitu.io/2019/12/10/16eeeabc35c171e2?w=1974&h=1804&f=jpeg&s=166289)



然后修改`HTML`代码:

```vue
// save.vue
<template>
	...
	<ul class="buttons">
    <li>
    	<a ref="saveDiagram" href="javascript:" title="保存为bpmn">保存为bpmn</a>
    </li>
    <li>
    	<a ref="saveSvg" href="javascript:" title="保存为svg">保存为svg</a>
    </li>
  </ul>
</template>
```

在`js`代码中加上:

```vue
// save.vue
<script>
	...
  addBpmnListener () {
      const that = this
      // 获取a标签dom节点
      const downloadLink = this.$refs.saveDiagram
      const downloadSvgLink = this.$refs.saveSvg
        // 给图绑定事件，当图有发生改变就会触发这个事件
      this.bpmnModeler.on('commandStack.changed', function () {
        that.saveSVG(function(err, svg) {
            that.setEncoded(downloadSvgLink, 'diagram.svg', err ? null : svg)
        })
        that.saveDiagram(function(err, xml) {
            that.setEncoded(downloadLink, 'diagram.bpmn', err ? null : xml)
        })
      })
  },
  // 下载为SVG格式,done是个函数，调用的时候传入的
  saveSVG(done) {
      // 把传入的done再传给bpmn原型的saveSVG函数调用
      this.bpmnModeler.saveSVG(done)
  },
  // 下载为bpmn格式,done是个函数，调用的时候传入的
  saveDiagram(done) {
      // 把传入的done再传给bpmn原型的saveXML函数调用
      this.bpmnModeler.saveXML({ format: true }, function(err, xml) {
          done(err, xml)
      })
  },
  // 当图发生改变的时候会调用这个函数，这个data就是图的xml
  setEncoded(link, name, data) {
      // 把xml转换为URI，下载要用到的
      const encodedData = encodeURIComponent(data)
      // 下载图的具体操作,改变a的属性，className令a标签可点击，href令能下载，download是下载的文件的名字
      console.log(link, name, data)
      let xmlFile = new File([data], 'test.bpmn')
      console.log(xmlFile)
      if (data) {
        link.className = 'active'
        link.href = 'data:application/bpmn20-xml;charset=UTF-8,' + encodedData
        link.download = name
      }
  }
</script>
```

案例Git地址: [LinDaiDai-bpmn.js测试案例save.vue](https://github.com/LinDaiDai/bpmn-vue-basic/blob/master/src/components/save.vue)  喜欢的小伙伴请给个`Star`🌟呀, 谢谢😊



## 后语

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注霖呆呆的公众号, 选择“其它”菜单中的“bpmn.js群”即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

