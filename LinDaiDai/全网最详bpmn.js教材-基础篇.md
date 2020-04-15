## 前言
> Q: bpmn.js是什么? 🤔️

[bpmn.js](https://bpmn.io/)是一个BPMN2.0渲染工具包和web建模器, 使得画流程图的功能在前端来完成.

> Q: 我为什么要写该系列的教材? 🤔️

因为公司业务的需要因而要在项目中使用到`bpmn.js`,但是由于`bpmn.js`的开发者是国外友人, 因此国内对这方面的教材很少, 也没有详细的文档. 所以很多使用方式很多坑都得自己去找.在将其琢磨完之后, 决定写一系列关于它的教材来帮助更多`bpmn.js`的使用者或者是期于找到一种好的绘制流程图的开发者. 同时也是自己对其的一种巩固.

由于是系列的文章, 所以更新的可能会比较频繁, **您要是无意间刷到了且不是您所需要的还请谅解**😊.

不求赞👍不求心❤️. 只希望能对你有一点小小的帮助.


## bpmn.js基本的使用

这一章节主要是介绍了`bpmn.js`最基本的几种实用方式, 适合完全没有接触过`bpmn.js`的新手或者想要在`vue`项目中使用它的开发者.

通过这一章节的讲解你可以学习到:

- [bpmn.js最简单的一种使用](#bpmn.js最简单的一种使用)

- [使用npm安装bpmn.js](#使用npm安装bpmn.js)

- [vue中使用bpmn.js](#vue中使用bpmn.js)

为了方便大家对后面的讲解有一个大概认识, 我们先来看一下使用`bpmn.js`画图都有哪些内容:


![](https://user-gold-cdn.xitu.io/2019/12/10/16eeea0f565dccaf?w=2028&h=1560&f=jpeg&s=283375)

### bpmn.js最简单的一种使用

我们可以直接使用`CDN`将`bpmn.js`引入到代码中使用:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>BPMNJS</title>
    <!--CDN加速-->
    <script src="https://unpkg.com/bpmn-js@6.0.2/dist/bpmn-viewer.development.js"></script>  		 <!--引入一个简单的xml字符串-->
    <script src="./xmlStr.js"></script>
    <style>
        #canvas {
            height: 400px;
        }
    </style>
</head>

<body>
    <div id="canvas"></div>
    <script>
        var bpmnJS = new BpmnJS({
            container: '#canvas'
        });
        bpmnJS.importXML(xmlStr, function(err) {
            if (!err) {
                console.log('success!');
                // 让图能自适应屏幕
                var canvas = bpmnJS.get('canvas')
                canvas.zoom('fit-viewport')
            } else {
                console.log('something went wrong:', err);
            }
        });
    </script>
</body>

</html>
```

(上面的`xmlStr.js`以及案例代码在这里[LinDaiDai-bpmn-basic-demo](https://github.com/LinDaiDai/bpmn-basic-demo))

如上面的案例所示, 我们使用`DNS`加速直接引入`bpmn.js`, 然后本地指定一个容器(也就是`id`为`canvas`的那个`div`), 接着用`bpmn.js`提供的方法`importXML`就可以解析`xml`字符串生成对应的工作流图了.

打开页面可以看到

![img1](https://user-gold-cdn.xitu.io/2019/12/10/16eee9ffc37d7bf7?w=3246&h=1598&f=jpeg&s=167906)



### 使用npm安装bpmn.js

上面提供的使用方式是一种最基本的方式,仅仅是将图展示出来,不能自己绘画也不能操作. 所以在工作中使用更多的还是采用`npm`安装到项目中使用. 我们可以使用以下命令进行安装:

```javascript
npm install --save bpmn-js
```

在应用程序中使用:

```javascript
import BpmnViewer from 'bpmn-js';
import testDiagram from './test-diagram.bpmn';

var viewer = new BpmnViewer({
  container: '#canvas'
});

viewer.importXML(testDiagram, function(err) {
  if (!err) {
    console.log('success!');
    viewer.get('canvas').zoom('fit-viewport');
  } else {
    console.log('something went wrong:', err);
  }
});
```

上面的`testDiagram`指的是某个`bpmn` 文件了,而不是第一个案例中的`xml`字符串.

官方这边也提供了一个例子, 可以看一下: [bpmn-js-example-bunding](https://github.com/bpmn-io/bpmn-js-examples/tree/master/bundling)



### vue中使用bpmn.js

在`vue` 中的使用可以分为以下几个部分:

- [vue中使用bpmn.js-基础篇](#vue中使用bpmn.js-基础篇)

- [vue中使用bpmn.js-左侧工具栏](#vue中使用bpmn.js-左侧工具栏)

- [vue中使用bpmn.js-右侧属性栏](#vue中使用bpmn.js-右侧属性栏)

为了方便讲解, 我先创建一个空的`vue`项目(只安装了路由):

```javascript
vue create vue-bpmn-basic
cd vue-bpmn-basic
npm i vue-router --save-D
```



**注⚠️️**

你可以不用本地创建, 此项目地址

[LinDaiDai/bpmn-vue-basic](https://github.com/LinDaiDai/bpmn-vue-basic)



#### vue中使用bpmn.js-基础篇

我在项目的`components`文件夹下创建一个名为`basic.vue`的文件, 且配置好路由:

```javascript
const routes = [
    {
    	path: '/basic',
    	component: () => import('./../components/basic')
    }
]
```

项目结构如图所示:

![img5](https://user-gold-cdn.xitu.io/2019/12/10/16eee9ffc48a825f?w=2048&h=1536&f=jpeg&s=315287)



1. 安装相关依赖

```javascript
npm i bpmn-js --save-D
```

2. 编写`HTML`代码

```html
// basic.vue
<template>
  <div class="containers">
    <div class="canvas" ref="canvas"></div>
  </div>
</template>
```

3. 编写`JS`代码

```
// basic.vue
  <script>
    // 引入相关的依赖
    import BpmnModeler from 'bpmn-js/lib/Modeler'
    import {
      xmlStr
    } from '../mock/xmlStr' // 这里是直接引用了xml字符串
    export default {
      name: '',
      components: {},
      // 生命周期 - 创建完成（可以访问当前this实例）
      created() { },
      // 生命周期 - 载入后, Vue 实例挂载到实际的 DOM 操作完成，一般在该过程进行 Ajax 交互
      mounted() {
        this.init()
      },
      data() {
        return {
          // bpmn建模器
          bpmnModeler: null,
          container: null,
          canvas: null
        }
      },
      methods: {
        init() {
          // 获取到属性ref为“canvas”的dom节点
          const canvas = this.$refs.canvas
          // 建模
          this.bpmnModeler = new BpmnModeler({
            container: canvas
          })
          this.createNewDiagram()
        },
        createNewDiagram() {
          // 将字符串转换成图显示出来
          this.bpmnModeler.importXML(xmlStr, (err) => {
            if (err) {
              // console.error(err)
            } else {
              // 这里是成功之后的回调, 可以在这里做一系列事情
              this.success()
            }
          })
        },
        success() {
          // console.log('创建成功!')
        }
      }
    }
  </script>
```

4. 编写`CSS`

```
// basic.vue
<style scoped>
.containers{
	position: absolute;
	background-color: #ffffff;
	width: 100%;
	height: 100%;
}
.canvas{
	width: 100%;
	height: 100%;
}
.panel{
	position: absolute;
	right: 0;
	top: 0;
	width: 300px;
}
</style>

```

使用命令`npm run start`启动项目, 打开可以看到:

![img2](https://user-gold-cdn.xitu.io/2019/12/10/16eee9ffc4703f65?w=2034&h=1822&f=jpeg&s=106661)



#### vue中使用bpmn.js-左侧工具栏

> 左侧工具栏作用: 给图形添加新的节点

如图所示:

![img3](https://user-gold-cdn.xitu.io/2019/12/10/16eee9ffc6b548c8?w=2028&h=1810&f=jpeg&s=115395)

要想使用左侧工具栏, 需要在项目中引用相应的样式:

1. 在`main.js`中引用`css`:

```javascript
// main.js
import Vue from 'vue'
import App from './App.vue'
import router from './router'
Vue.config.productionTip = false
// 以下为bpmn工作流绘图工具的样式
import 'bpmn-js/dist/assets/diagram-js.css' // 左边工具栏以及编辑节点的样式
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn.css'
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-codes.css'
import 'bpmn-js/dist/assets/bpmn-font/css/bpmn-embedded.css'
new Vue({
    router,
    render: h => h(App),
}).$mount('#app')

```

2. 页面上引入`propertiesProviderModule`:

```vue
// provider.vue
<script>
...
import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda'
...
methods: {
	init () {
    // 获取到属性ref为“canvas”的dom节点
    const canvas = this.$refs.canvas
    // 建模
    this.bpmnModeler = new BpmnModeler({
      container: canvas,
      //添加控制板
      propertiesPanel: {
        parent: '#js-properties-panel'
      },
      additionalModules: [
        // 左边工具栏以及节点
        propertiesProviderModule
      ]
    })
    this.createNewDiagram()
	},
	...
}
</script>

```

`provider.vue`的其他代码片段都和`basic.vue` 相同.

此时打开页面就发现多了左侧的工具栏, 且可以添加节点.



#### vue中使用bpmn.js-右侧属性栏

> 属性栏的作用: 用户在点击图上的节点的时候, 能获取到该节点的属性信息

如图所示:

![img4](https://user-gold-cdn.xitu.io/2019/12/10/16eee9ffc70e9716?w=2252&h=1810&f=jpeg&s=170862)



想要使用右侧的属性栏就得安装上一个名为`bpmn-js-properties-panel`的插件了.

1. 安装插件

```javascript
npm i bpmn-js-properties-panel --save-D

```

2.  在`main.js`中引入相应样式:

```javascript
// main.js
import Vue from 'vue'
...
import 'bpmn-js-properties-panel/dist/assets/bpmn-js-properties-panel.css' // 右边工具栏样式
...

```

3. 在页面中引入`propertiesProviderModule`:

```
// panel.vue
<script>
...
import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda'
...
methods: {
  init() {
    // 获取到属性ref为“canvas”的dom节点
    const canvas = this.$refs.canvas
    // 建模
    this.bpmnModeler = new BpmnModeler({
      container: canvas,
      //添加控制板
      propertiesPanel: {
        parent: '#js-properties-panel'
      },
      additionalModules: [
        // 左边工具栏以及节点
        propertiesProviderModule,
        // 右边的工具栏
        propertiesPanelModule
      ]
    })
    this.createNewDiagram()
  }
}

```



### 后语

上述案例项目Git地址 : [https://github.com/LinDaiDai/bpmn-vue-basic](https://github.com/LinDaiDai/bpmn-vue-basic) 喜欢的小伙伴请给个`Star`🌟呀, 谢谢😊

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注**霖呆呆(LinDaiDai)的公众号**, 选择 **其它** 菜单中的 **bpmn.js群** 即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

