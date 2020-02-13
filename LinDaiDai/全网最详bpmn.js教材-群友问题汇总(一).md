
# 全网最详bpmn.js教材-群友问题汇总(一)

这一章节主要是将近段时间[前端bpmn.js交流群](https://juejin.im/post/5e15b149e51d45238744d3d0)中群友提的一些问题做一个汇总...

后面有碰到同样问题的小伙伴希望能帮到你们...

问题的解答有的是群友给出的方案有些是我自己想的方案, 可能不是最优解, 如果有更好解决办法的小伙伴还希望能够提出来呀 😁.

## 目录

- `palette`左侧工具栏
    - 如何给工具栏的每一项都加上标题
    - `palette`和`renderer`中的图片如何用本地图片
    - 自定义`palette`中如何使用它本身的图标样式
- `contextPad`
    - `contextPad`中的内容根据元素类型不同显示不同
- 文件
    - 如何加载本地`bpmn`或者`xml`文件
- 属性
    - 每个元素的`id`是否能够修改
- 其它
    - 如何创建线节点
    - 右下角的绿色`logo`能否隐去


## palette左侧工具栏

### 1. 如何给工具栏的每一项都加上标题

实现类似于下面这张图的效果:


![](https://user-gold-cdn.xitu.io/2020/2/12/1703966e2fd77fe3?w=990&h=1472&f=png&s=71814)

原先我们实现自定义palette的时候只考虑到了显示图片的情况, 有一些业务场景可能需要将每种元素的标题显示出来.

这里我提供了两种解决方案:

1. 给每个类定义一个伪类, 将title写到这个伪类里
2. 额...要UI设计师将每个title画到每个元素图表的下面, 也就是将title作为图标的一部分

这里我主要讲解一下第一种实现方式.

首先我们知道在`customPalette`中是有这么一个东西的:

```javascript
'append.lindaidai-task': {
    group: 'model',
    className: 'icon-custom lindaidai-task',
    title: translate('创建一个类型为lindaidai-task的任务节点'),
    action: {
        click: appendTask,
        dragstart: appendTaskStart
    }
}
```
主要看`className`.

之前我教材中的css代码是这样写的:

```css
.icon-custom {
    border-radius: 50%;
    background-size: 65%;
    background-repeat: no-repeat;
    background-position: center;
}
.icon-custom.lindaidai-task {
    position: relative;
    background-image: url('https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/rules.png');
}
```

现在我想在它下面加一个标题:

```css
.icon-custom.lindaidai-task::after {
    font-size: 12px;
    content: 'LinDaiDai'; /* 这里放的就是标题 */
    position: absolute;
    top: 17px;
    left: 0;
}
```

这样就简单的实现了这么一个显示标题的功能.

具体案例可以看这里: [bpmn-vue-basic](https://github.com/LinDaiDai/bpmn-vue-basic)

### 2. palette和renderer中的图片如何用本地图片

palette上想要用本地图片很简单, 因为自定义palette主要是依靠className, 而className肯定是写在css文件中的, 我们只需要找到图片对应的相对路径就可以了:

例如项目目录为:

```
/src
    |- /assets
        |- rules.png
    |- css
        |- app.css

```
它对应的引用:

```css
/*app.css*/
.icon-custom.lindaidai-task {
    position: relative;
    /* background-image: url('https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/rules.png'); */
    background-image: url('../assets/rules.png');
}
```

我们知道自定义renderer里想要实现自定义效果主要是靠`svgCreate`方法创建出一个`image`元素然后添加到返回值中, 这个图片的url我原先一直用的是网络图片, 那肯定没什么问题.

而如果你想要用一张本地图片的话, 你开始想到的可能是这样使用相对路径:

```javascript
// customRenderer.js
const imageConfig = {
    'url': '../../assets/rules.png',
    // 'url': 'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/rules.png',
    'attr': { x: 0, y: 0, width: 48, height: 48 }
}

const { attr, url } = imageConfig;
const customIcon = svgCreate('image', {
    ...attr,
    href: url
})
```
但是保存打开页面之后发现不尽人意...

在这里你需要使用`CommonJS`的引入方式才可以, 将它转换为`base64`的`Data URL`:

```javascript
// customRenderer.js
const imageConfig = {
    'url': require('../../assets/rules.png'),
    // 'url': 'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/rules.png',
    'attr': { x: 0, y: 0, width: 48, height: 48 }
}

const { attr, url } = imageConfig;
const customIcon = svgCreate('image', {
    ...attr,
    href: url
})
```

保存打开页面发现是可以的.

但是在这里我不推荐你使用相对路径的方式, 因为配置文件的位置可能随时会变, 一变的话相对路径也得更这边, 所以如果你是使用以`webpack`打包工具为基础的脚手架的话, 我建议你配置一个`alias`(别名), 那样也能方便你开发.

配置`alias`的方式很简单, 如果你和我一样是用`vue`开发项目的话, 请检查一下你的根目录有没有一个叫`vue.config.js`的文件, 如果没有的话, 创建一个, 并在其中写上:

```javascript
// customRenderer.js
const path = require('path')

const resolve = dir => path.join(__dirname, dir)

module.exports = {
    chainWebpack: config => {
        config.resolve.alias
            .set('@', resolve('src'))
            .set('@assets', resolve('src/assets'))
    }
}
```
(其它框架请自行度娘...)

是不是看着也很简单, 和它的英文一样, 其实也就是给某个文件夹配置一个别名.

比如我这里就是给`src`和`src/assets`配置了别名.

这样你在代码里写`@/views/xxx.vue`就当于写`src/views/xxx.vue`.

现在让我们来修改一下前面的路径:

```javascript
// customRenderer.js
const imageConfig = {
    'url': require('@assets/rules.png'),
    // 'url': require('../../assets/rules.png'),
    // 'url': 'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/rules.png',
    'attr': { x: 0, y: 0, width: 48, height: 48 }
}

const { attr, url } = imageConfig;
const customIcon = svgCreate('image', {
    ...attr,
    href: url
})
```

现在无论你如何移动你的`customRenderer.js`文件, 图片的路径都不会错了.

案例GitHub地址: [bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom)

(该问题解决方案来自简书网友 [梦想还是要有的_bfc7](https://www.jianshu.com/u/6cf0ac48d650))

### 3. 自定义palette中如何使用它本身的图标样式

我们之前的自定义palette一直都是使用我们自己找的一写图片图标...

而如果你某一个元素的样式就想要它官方提供的怎么办 🤔️?

例如我要实现这样的效果:

前两个元素是我自定义的, 最后一个网关用官方提供的原始样式, 如下图:


![](https://user-gold-cdn.xitu.io/2020/2/12/17039def35b021ea?w=1354&h=624&f=png&s=54845)

想要做到这一点其实很简单, 还记得我们自定义palette的时候是依赖着一个`className`属性的吗?

你只需要将这个`className`设置成它官方提供的就可以了.

那有人就要问了,这个官方原始的`className`我该到哪找呢 😂?


![](https://user-gold-cdn.xitu.io/2020/2/12/17039e10758400bc?w=2722&h=1670&f=png&s=409407)

审查元素, 找到对应的类名, 比如这里是`bpmn-icon-gateway-none`

然后在将`customPalette`中的网关设置成这个`className`:

```javascript
PaletteProvider.prototype.getPaletteEntries = function(element) {
    ...
    return {
        ...
        'create.exclusive-gateway': {
            group: 'gateway',
            className: 'bpmn-icon-gateway-none', // 重点是这个
            title: '创建一个网关',
            action: {
                dragstart: createGateway(),
                click: createGateway()
            }
        }
    }
}
```
现在左侧的工具栏就已经可以将原始的网关样式显示出来了.

但是有一个问题了, 那就是此时你想要用你定义好的这个网关在右边画图, 也就是进入`renderer`阶段, 如果你是完全自定义`renderer`的话, 控制台可能就会报错了...

先让我们来回顾一下`customRenderer.js`是怎么写的:

```javascript
export default function CustomRenderer(eventBus, styles, textRenderer) {
    this.drawCustomElements = function(parentNode, element) {
        if (customElements.includes(type)) { // or customConfig[type]
            // 这里是自定义的元素
        }
    }
}

CustomRenderer.prototype.drawShape = function(p, element) {
    return this.drawCustomElements(p, element)
}

```

如果你和我一样是将**是否是自定义的元素**这个判断放到`drawCustomElements`这个方法里写的话你可能就会报错了...因为它会告诉你找不到这个类型的渲染方式.

解决办法是这层判断放到`CustomRenderer.prototype.drawShape`里:

```javascript
export default function CustomRenderer(eventBus, styles, textRenderer) {
    this.drawCustomElements = function(parentNode, element) {
        // 这里是自定义的元素
    }
}

CustomRenderer.prototype.drawShape = function(p, element) {
    if (customElements.includes(element.type)) { // 放到这里判断
        return this.drawCustomElements(p, element)
    }
}
```

这样修改之后, 在执行`drawShape`方法的时候, 它就会判断是否是自定义元素, 如果是自定义元素的话才有返回值, 否则就没有返回值.

没有返回值时它就会根据原始的样式进行渲染了.

这是因为我们在设计自定义modeler的时候将原始的modeler也引用进来了:


![](https://user-gold-cdn.xitu.io/2020/2/12/17039eea4a987c2f?w=3234&h=2048&f=png&s=863469)

关于上述案例可查看: [bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom) 中的**自定义modeler**那一个tab项.

##  contextPad

### 1. contextPad中的内容根据元素类型不同显示不同

不同类型的节点出现的contextPad的内容可能是不同的.
比如:

StartEvent会出现edit、delete、Task、BusinessRuleTask、ExclusiveGateway等等;

EndEvent只能出现edit、delete;SequenceFlow只能出现edit、delete.

也就是说我们需要根据节点类型来返回不同的contextPad.

这个其实我在[《全网最详bpmn.js教材-封装组件篇》](https://juejin.im/post/5dfecb3de51d45581b11efac) 这里面已经提到过该如何处理了, 具体可以看那篇文章:


![](https://user-gold-cdn.xitu.io/2020/2/13/1703d41e8229cb24?w=1356&h=1060&f=png&s=152447)

## 文件

### 1. 如何加载本地bpmn或者xml文件

在`http篇`那一章节, 我向大家演示的是通过一个远程的文件链接(可能是后台传递过来的), 然后通过`axios`解析获取的文件, 从而得到`xml`的字符串再调用`importXML`方法显示出图形.

那么如何加载一个本地的`bpmn`文件或者`xml`文件呢.

#### 方案一: 使用raw-loader

我首先想到的是通过`xml-loader`解析这两类文件, 但是不知道能不能成, 于是试了试.

**(项目案例基于: [bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom))**

首先在项目中安装`xml-loader`:

```
$ npm i --save-dev xml-loader
```

然后配置一下`vue.config.js`这个文件(这个文件在上面👆`palette和renderer中的图片如何用本地图片`已经提到过了, 没有的话就在根目录创建一个)

**vue.config.js**:

```javascript
const path = require('path')

const resolve = dir => path.join(__dirname, dir)

module.exports = {
    chainWebpack: config => {
        config.resolve.alias
            .set('@', resolve('src'))
            .set('@assets', resolve('src/assets'))
            .end()
        config.module // 主要是看这部分
            .rule('xml-loader')
            .test(/.(bpmn|xml)$/)
            .use('xml-loader')
            .loader('xml-loader')
            .end()
    }
}
```
这里的意思就是以`bpmn或者xml`为后缀的文件会被`xml-loader`处理.

现在让我们在`custom-renderer.vue`这个页面中来试试:

```html
<script>
    const bpmnXml = require('../mock/diagram.bpmn')
    
    console.log(bpmnXml)
</script>
```

打印出来的`bpmnXml`却是一个对象, 而不是字符串:


![](https://user-gold-cdn.xitu.io/2020/2/13/1703d1375215f219?w=1202&h=784&f=png&s=159880)

而且使用`importXML`想要转换这个对象显然是不行的.

这可怎么办呢...


![](https://user-gold-cdn.xitu.io/2020/2/13/1703d15dad6137ef?w=295&h=221&f=png&s=33684)

等等, 既然`importXML`解析只需要一个字符串的话, 让我想到了前几天刚学到的`raw-loader`, 它可以获取`txt`中的文本内容, 那是不是也能获取`bpmn和xml`呢 🤔️?

说干就干, 继续安装`raw-loader`:

```
$ npm i --save-dev raw-loader
```

然后修改`vue.config.js`:

```
const path = require('path')

const resolve = dir => path.join(__dirname, dir)

module.exports = {
    chainWebpack: config => {
        config.resolve.alias
            .set('@', resolve('src'))
            .set('@assets', resolve('src/assets'))
            .end()
        config.module // 将xml-loader替换成raw-loader
            .rule('raw-loader')
            .test(/.(bpmn|xml)$/)
            .use('raw-loader')
            .loader('raw-loader')
            .end()
    }
}
```
修改完之后记得重启项目...

然后让我们来看看效果:

```html
<script>
    const bpmnXml = require('../mock/diagram.bpmn')
    
    console.log(bpmnXml)
    console.log(typeof bpmnXml) // object
    console.log(bpmnXml.default)
</script>
```

此时打印出来的虽然也是个对象, 但是里面有个`default`属性, 它存储的就是xml字符串


![](https://user-gold-cdn.xitu.io/2020/2/13/1703d516ea5f3e68?w=3078&h=1356&f=png&s=486497)

所以我们取`default`属性就可以了:

```javascript
this.bpmnModeler.importXML(bpmnXml.default, err => {
    if (err) {
        
    } else {
        // 这里是成功之后的回调, 可以在这里做一系列事情
        this.success()
    }
})
```

不知道是不是版本的原因, 有些通过`raw-loader`转换的`bpmn`文件就直接是字符串, 而不是这个对象, 大家在使用的时候注意一下.

注意⚠️:

关于上面`vue.config.js`是`vue-cli3`中`webpack`的配置, 如果你的项目的构建方式是使用原始`webpack`的话, 它就相当于`webpack.config.js`中的:

```javascript
module.exports = {
    ...
    module: {
        rules: [
            {
                test: /.(bpmn|xml)$/,
                use: 'raw-loader'
            }
        ]
    }
}
```

其它打包方式我这里就不说了.

#### 方案二: 使用new FileReader()

这个方案是群里的群友**火莲**提出来的, 他已经实现了, 我就没去试了, 不过应该是可以的.

![](https://user-gold-cdn.xitu.io/2020/2/13/1703d22e192e74cb?w=942&h=512&f=png&s=120495)


![](https://user-gold-cdn.xitu.io/2020/2/13/1703d2354f70edd1?w=868&h=310&f=png&s=81929)

```javascript
var reader = new FileReader();
reader.readAsText(file);
reader.onload = function(oFREvent){
    var xmlDoc = oFREvent.target.result;
    openDiagram(xmlDoc);
}
```

## 属性

### 1. 每个元素的id是否能够修改

其实每个元素的id也是一个属性而已, 但是它并不会随着元素类型的改变而改变, 也就是说正常情况下它是不会变动的.

不过既然它是一个属性, 那么我们就能通过`modeling.updateProperties()`修改它:

```javascript
const properties = { id: 'id0001' } 
const { modeler, element } = this
const modeling = modeler.get('modeling')
modeling.updateProperties(element, properties)
```


## 其它

### 1. 如何创建线节点

创建线节点在[《全网最详bpmn.js教材-封装组件篇》](https://juejin.im/post/5dfecb3de51d45581b11efac) 这里面也提到过该如何处理, 具体可以看那篇文章.

### 2. 右下角的绿色logo能否隐去

关于右下角logo能否隐去这个问题, 群里产生了激烈的讨论, 因为大家都怕吃官司侵权...

用官网的话来说就是不能:


![](https://user-gold-cdn.xitu.io/2020/2/13/1703d263a4c2826c?w=1668&h=1044&f=png&s=170082)

不过群友**zaw**也提供了一种解决方案😂:

找到那个类名, 然后样式设置 `display : none`.

我认为你能不隐就不要隐去了吧, 虽然人家这东西是开源的, 但是也说了不要去掉, 就遵从作者的意愿吧(**就像我在这里求大家一键三连一样: 点赞, 收藏, Star 呀 哈哈哈...**)

## 后语

全部教材目录: [《全网最详bpmn.js教材》](https://juejin.im/post/5def372af265da33c84a4818)

GitHub教材地址: [bpmn-chinese-document](https://github.com/LinDaiDai/bpmn-chinese-document)   求Star 🌟 求Fork 📓...

疫情四溢, 足不出户, 霖呆呆从大年初二到今天就只出过一次门 😂...

不知道你们那边情况怎么样, 反正我家后面300米处的那户人家夫妻俩已经被感染隔离起来了...

所以我们小镇也被全面封锁了, 还不知道啥时候能返深...

不过在家呆着挺好的, 难得有和家人相处的机会, 要好好珍惜呀, 而且能趁着假期恶补一下自己薄弱的知识点就很好, 哈哈😄.

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注霖呆呆的公众号, 选择“其它”菜单中的“bpmn.js群”即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)