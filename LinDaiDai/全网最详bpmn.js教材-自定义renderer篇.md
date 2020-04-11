## 前言
> Q: bpmn.js是什么? 🤔️

[bpmn.js](https://bpmn.io/)是一个BPMN2.0渲染工具包和web建模器, 使得画流程图的功能在前端来完成.

> Q: 我为什么要写该系列的教材? 🤔️

因为公司业务的需要因而要在项目中使用到`bpmn.js`,但是由于`bpmn.js`的开发者是国外友人, 因此国内对这方面的教材很少, 也没有详细的文档. 所以很多使用方式很多坑都得自己去找.在将其琢磨完之后, 决定写一系列关于它的教材来帮助更多`bpmn.js`的使用者或者是期于找到一种好的绘制流程图的开发者. 同时也是自己对其的一种巩固.

由于是系列的文章, 所以更新的可能会比较频繁, **您要是无意间刷到了且不是您所需要的还请谅解**😊.

不求赞👍不求心❤️. 只希望能对你有一点小小的帮助.

## 自定义Renderer篇
接着上一章节, 我们已经知道了该如何自定义左侧的工具栏(Palette), 不了解的小伙伴可以移步: [《全网最详bpmn.js教材-自定义palette篇》](https://juejin.im/post/5df197c4f265da33bd4976af).

但是同时我们也知道仅仅只改变`Palette`是不够的, 因为绘画出来的图形还是“裸体的”: 

![](https://user-gold-cdn.xitu.io/2019/12/13/16efdcf6c9a313b2?w=1070&h=960&f=png&s=142295)

这一章节我们就来看一下如何自定义画布上的图形, 也就是实现自定义`Renderer`的功能.

通过阅读你可以学习到:

- [在默认的Renderer基础上修改](#在默认的Renderer基础上修改)
- [完全自定义Renderer](#完全自定义Renderer)
- [label标签自定义在元素下方](#label标签自定义在元素下方)

## 在默认的Renderer基础上修改

和自定义`Palette`一样, 先来看看最简单的在原有的元素上进行修改.

### 前期准备

让我们接着在[LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom)案例项目上进行开发.

在`components`文件夹下新建一个`custom-renderer.vue`文件, 同时配置路由“自定义renderer”.

在`components/custom`文件夹下新建一个`CustomRenderer.vue`文件, 用来自定义`renderer`.

在`components`文件夹下新建一个`utils`文件夹同时新建`util.js`文件, 用来放一些公共的方法和配置.

### 编写`CustomRenderer.vue`代码

由于是在`bpmn.js`已有的元素上进行修改, 所以首先我们可以先将`BaseRenderer`这个类引入进来, 然后让我们的自定义`renderer`继承它:

```javascript
import BaseRenderer from 'diagram-js/lib/draw/BaseRenderer' // 引入默认的renderer
const HIGH_PRIORITY = 1500 // 最高优先级
export default class CustomRenderer extends BaseRenderer { // 继承BaseRenderer
    constructor(eventBus, bpmnRenderer) {
        super(eventBus, HIGH_PRIORITY)
        this.bpmnRenderer = bpmnRenderer
    }

    canRender(element) {
        // ignore labels
        return !element.labelTarget
    }

    drawShape(parentNode, element) { // 核心函数就是绘制shape
        const shape = this.bpmnRenderer.drawShape(parentNode, element)
        return shape
    }

    getShapePath(shape) {
        return this.bpmnRenderer.getShapePath(shape)
    }
}

CustomRenderer.$inject = ['eventBus', 'bpmnRenderer']
```
上面👆的代码很简单, 相信大家都可以看的明白.

**注: 这里有个小坑要注意一下, 就是`HIGH_PRIORITY`不能够去掉, 不然的话你会发现它不会执行下面的`drawShpe`函数**

到了这里可能就有小伙伴要问了, 感觉你做了这么多并没有什么用啊, 还是没有看到关于自定义`renderer`的效果呀😅!

没错, 只完成上面的步骤那是不够的, 关键是在于如何编写`drawShape`这个方法.

### 编写`drawShape`代码

我们可以先在前面创建好的`utils/util.js`文件下写下此代码:
```javascript
// util.js
const customElements = ['bpmn:Task']

export { customElements }
```
也就是创建了一个名为`customElements`的数组然后导出, 至于数组里为什么只有一项`bpmn:Task`?🤔️

那是因为在上一个案例中我创建的`lindaidai-task`的类型就是`bpmn:Task`类型的.

所以这个数组的作用就是用来放哪些类型是需要我们自定义的, 从而在渲染的时候就可以与不需要自定义的元素作区分.

甚至你还可以做一些配置:
```javascript
const customElements = ['bpmn:Task'] // 自定义元素的类型
const customConfig = { // 自定义元素的配置(后面会用到)
    'bpmn:Task': {
        'url': 'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/rules.png',
        'attr': { x: 0, y: 0, width: 48, height: 48 }
    }
}

export { customElements, customConfig }
```

让我们在`CustomRenderer.js`中使用并编写它:

```javascript
import { customElements, customConfig } from '../utils/util'

...
    drawShape(parentNode, element) {
      const type = element.type // 获取到类型
      if (customElements.includes(type)) { // or customConfig[type]
        const { url, attr } = customConfig[type]
        const customIcon = svgCreate('image', { // 在这里创建了一个image
          ...attr,
          href: url
        })
        element['width'] = attr.width // 这里我是取了巧, 直接修改了元素的宽高
        element['height'] = attr.height
        svgAppend(parentNode, customIcon)
        return customIcon
      }
      const shape = this.bpmnRenderer.drawShape(parentNode, element)
      return shape
    }
...
```
可以看到,实现让页面渲染出自己想要的效果的做法就是使用`svgCreate`方法创建一个`image`并添加到父节点中.

### 导出并使用`CustomRenderer`
同样的自定义`renderer`需要导出才能使用, 修改`custom/index.js`文件:
```javascript
import CustomPalette from './CustomPalette'
import CustomRenderer from './CustomRenderer'

export default {
    __init__: ['customPalette', 'customRenderer'],
    customPalette: ['type', CustomPalette],
    customRenderer: ['type', CustomRenderer]
}
```
**注意: `__init__`中的属性命名`customRenderer`都是固定的写法不能修改, 不然就会没有效果**

要是你看了之前`custom-palette.vue`的话, 就知道直接在页面上应用就行了:
```html
<!--custom-renderer.vue-->
<script>
...
import customModule from './custom'
...
this.bpmnModeler = new BpmnModeler({
...
    additionalModules: [
        // 左边工具栏以及节点
        propertiesProviderModule,
        // 自定义的节点
        customModule
    ]
})
```
**注意: 项目案例里我为了方便演示, 在`custom-palette`中引入的是`ImportJS/onlyRenderer.js`, 而上面的案例是以引入`custom/index.js`为讲解的, 这个自己要明白如何区分.**

此时打开页面就可以看到效果了, 类型为`bpmn:Task`的节点就被渲染成了自定义的“黄金积木”😝:
![bpmnCustom9.png](https://user-gold-cdn.xitu.io/2019/12/14/16f0509e5df4260d?w=2042&h=1574&f=png&s=211697)

## 完全自定义Renderer
完全自定义`Renderer`的意思就是将原本使用`new BpmnModeler`创建画布的方式改为使用`new CustomModeler`来创建.

这一部分在[《全网最详bpmn.js教材-自定义palette篇》](https://juejin.im/post/5df197c4f265da33bd4976af)中讲解的很详细了, 就不做过多的阐述.

同样是在`customModeler/custom`的文件夹下创建一个`customRender.js`文件, 然后写入以下代码:
```javascript
/* eslint-disable no-unused-vars */
import inherits from 'inherits'

import BaseRenderer from 'diagram-js/lib/draw/BaseRenderer'

import {
    append as svgAppend,
    create as svgCreate
} from 'tiny-svg'

import { customElements, customConfig } from '../../utils/util'
/**
 * A renderer that knows how to render custom elements.
 */
export default function CustomRenderer(eventBus, styles) {
    BaseRenderer.call(this, eventBus, 2000)

    var computeStyle = styles.computeStyle

    this.drawCustomElements = function(parentNode, element) {
        console.log(element)
        const type = element.type // 获取到类型
        if (customElements.includes(type)) { // or customConfig[type]
            const { url, attr } = customConfig[type]
            const customIcon = svgCreate('image', {
                ...attr,
                href: url
            })
            element['width'] = attr.width // 这里我是取了巧, 直接修改了元素的宽高
            element['height'] = attr.height
            svgAppend(parentNode, customIcon)
            return customIcon
        }
        const shape = this.bpmnRenderer.drawShape(parentNode, element)
        return shape
    }
}

inherits(CustomRenderer, BaseRenderer)

CustomRenderer.$inject = ['eventBus', 'styles']

CustomRenderer.prototype.canRender = function(element) {
    // ignore labels
    return !element.labelTarget;
}

CustomRenderer.prototype.drawShape = function(p, element) {
    return this.drawCustomElements(p, element)
}

CustomRenderer.prototype.getShapePath = function(shape) {
    console.log(shape)
}
```
直接修改原型链中的`drawShape`方法就可以了.

然后记得在`customModeler/custom/index.js`中将其导出.

## label标签自定义在元素下方
由于评论区有小伙伴提了问题: 该如何将`label`标签自定义在元素的下方?

因此霖呆呆我回去也是花了点时间研究了一下`label`标签.

首先`label`标签实际上是`xml`中各个标签上的一个名叫`name`的属性, 如下图:

![](https://user-gold-cdn.xitu.io/2019/12/18/16f1998eac7e0bd4?w=1968&h=1706&f=png&s=1923075)

开始节点和`lindaidai-task`中都有`name`属性, 但是在`bpmn:StartEvent`上能将这个`label`显示出来, 是因为在下面有一个`bpmndi:BPMNLabel`的标签.

于是就造成了图形上是这样显示的:

![bpmn11.png](https://user-gold-cdn.xitu.io/2019/12/18/16f199d412a014ef?w=862&h=408&f=png&s=54178)

那么我们该如何将这里的`label`显示出来呢?

首先让我们先将`Shape`打印出来看看:
![bpmn12.png](https://user-gold-cdn.xitu.io/2019/12/18/16f19a7db2bcecca?w=1768&h=754&f=png&s=551001)

可以发现在`businessObject`中有一个`name`属性...

既然这样的话, 我们肯定也能在`drawShape`中拿到这个`name`属性, 之后可以用`svgCreate`方法给父节点中添加一个文本类型的标签.

```javascript
// CustomRenderer.js

import { hasLabelElements } from '../../utils/util'

drawShape(parentNode, element) {
    const type = element.type // 获取到类型
    if (customElements.includes(type)) { // or customConfig[type]
        const { url, attr } = customConfig[type]
        const customIcon = svgCreate('image', {
            ...attr,
            href: url
        })
        element['width'] = attr.width // 这里我是取了巧, 直接修改了元素的宽高
        element['height'] = attr.height
        svgAppend(parentNode, customIcon)
        // 判断是否有name属性来决定是否要渲染出label
        if (!hasLabelElements.includes(type) && element.businessObject.name) {
            const text = svgCreate('text', {
                x: attr.x,
                y: attr.y + attr.height + 20, // y取的是父元素的y+height+20
                "font-size": "14",
                "fill": "#000"
            })
            text.innerHTML = element.businessObject.name
            svgAppend(parentNode, text)
            console.log(text)
        }
        return customIcon
    }
    const shape = this.bpmnRenderer.drawShape(parentNode, element)
    return shape
}
```

因为有些元素本身就带有`label`属性的, 比如`bpmn:StartEvent`, 所以不需要重新渲染, 因此我在`util.js`中加了一个`hasLabelElements`数组:
```javascript
// utils/util.js
const hasLabelElements = ['bpmn:StartEvent', 'bpmn:EndEvent'] // 一开始就有label标签的元素类型
```
之前我是想通过`element.labels.length<=0`来过滤掉开始就有`label`标签的元素的, 但是发现在渲染阶段还获取不到`labels`, 所以长度一直都会是`0`, 就干脆定义一个`hasLabelElements`来判断好了😓...

打开页面效果是这样的:

![bpmn13.png](https://user-gold-cdn.xitu.io/2019/12/18/16f19b839f5eb2ff?w=780&h=408&f=png&s=58346)

**看起来好像成功了 !  good boy ! 😄**

但是当我双击想要去编辑`label`文字的时候, 却出现了这样的效果:


![](https://user-gold-cdn.xitu.io/2019/12/18/16f19baa78817e13?w=892&h=446&f=png&s=52035)

它直接在我原来图形的上面新建了一个输入框...

额😅...其实我也没有想到什么好的办法去解决,在这里我提供一个看起来可行的方案:
**在双击元素的时候, 将`text`给移除, 或者将他的`innerHTML`设置为`''`**.

当然你要是感觉这样也看得下去的话, 咱不捣鼓也行, 毕竟你编辑这里面的内容, 下面的`label`也是会同步的变的.

再不济的话, 你可以全局修改`djs-direct-editing-parent`这个类的样式, 将下面的文字给覆盖上也是可以的... 当然感觉这个不是一个很好的办法.
在`app.css`中写入:
```css
.djs-direct-editing-parent {
    top: 130px!important;
    width: 60px!important;
}
```
**总结**

上面的做法主要是利用`svgCreate`来创建`text`元素添加到`parentNode`中, 其实`bpmn.js`中用到了很多`ting-svg`的东西, 之前也没接触过这些, 然后也是通过查找资料了解到`svgCreate`的用法...

科普一波好了, 哈哈😄:
[SVG基础知识](https://www.jianshu.com/p/54385eb69eec)

## 后语
上面👆案例用的都是同一个项目🦐

项目案例Git地址: [LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom) 喜欢的小伙伴请给个`Star`🌟呀, 谢谢😊

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)



