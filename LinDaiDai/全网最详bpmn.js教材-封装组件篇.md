## 前言
> Q: bpmn.js是什么? 🤔️

[bpmn.js](https://bpmn.io/)是一个BPMN2.0渲染工具包和web建模器, 使得画流程图的功能在前端来完成.

> Q: 我为什么要写该系列的教材? 🤔️

因为公司业务的需要因而要在项目中使用到`bpmn.js`,但是由于`bpmn.js`的开发者是国外友人, 因此国内对这方面的教材很少, 也没有详细的文档. 所以很多使用方式很多坑都得自己去找.在将其琢磨完之后, 决定写一系列关于它的教材来帮助更多`bpmn.js`的使用者或者是期于找到一种好的绘制流程图的开发者. 同时也是自己对其的一种巩固.

由于是系列的文章, 所以更新的可能会比较频繁, **您要是无意间刷到了且不是您所需要的还请谅解**😊.

不求赞👍不求心❤️. 只希望能对你有一点小小的帮助.

## 封装组件篇

在进入这一章节的学习之前, 我希望你能先掌握前面几节的知识点: **自定义`palette`、自定义`renderer`、自定义`contextPad`、编辑删除节点**.

因为这一章节会将前面几节的内容做一个汇总, 然后提供一个可用的`bpmn`组件解决方案.

通过阅读你可以学习到:

- [创建线节点](#创建线节点)
- [自定义modeler](#自定义modeler)
- [将bpmn封装成组件](#将bpmn封装成组件)

## 创建线节点

首先让我们先来了解一下线节点是如何创建的.

我以`CustomPalette.js`为例子🌰, 还记得在之前讲的`createTask`吗, 创建线和它差不多:
```javascript
// CustomPalette.js
PaletteProvider.$inject = [
    ...
    'globalConnect'
]
PaletteProvider.prototype.getPaletteEntries = function(element) {
    const { globalConnect } = this
    
    function createConnect () {
        return {
          group: 'tools',
          className: 'icon-custom icon-custom-flow',
          title: '新增线',
          action: {
            click: function (event) {
              globalConnect.toggle(event)
            }
          }
        }
    }
    
    return {
        'create.lindaidai-task': {...},
        'global-connect-tool': createConnect()
    }
}
```

这样就可以画出线了:

![bpmnModeler.png](https://user-gold-cdn.xitu.io/2019/12/22/16f2b9bf57bc899f?w=788&h=446&f=png&s=67316)

## 自定义modeler

经过了上面那么的例子, 其实我们不难发现, 在每个关键的函数中, 都是将自己想要自定义的东西通过函数返回值传递出去. 

而且返回值的内容都大同小异, 无非就是`group、className`等等东西, 那么这样的话, 我们是不是可以将其整合一下, 减少许多代码量呢?

我们可以构建这样一个函数:

```javascript
// CustomPalette.js
function createAction (type, group, className, title, options) {
    function createListener (event) {
      var shape = elementFactory.createShape(assign({ type }, options))
      create.start(event, shape)
    }

    return {
      group,
      className,
      title: '新增' + title,
      action: {
        dragstart: createListener,
        click: createListener
      }
    }
}
```
 它接收所有元素不同的属性, 然后返回一个自定义元素.

 但是线的创建可能有些不同:

 ```javascript
 // CustomPalette.js
 function createConnect (type, group, className, title, options) {
    return {
      group,
      className,
      title: '新增' + title,
      action: {
        click: function (event) {
          globalConnect.toggle(event)
        }
      }
    }
  }
 ```

 因此我这里把创建元素的函数分为两类: `createAction`和`createConnect`.

 接下来我们只需要构建一个这样的数组:
 ```javascript
 // utils/util.js
 const flowAction = { // 线
    type: 'global-connect-tool',
    action: ['bpmn:SequenceFlow', 'tools', 'icon-custom icon-custom-flow', '连接线']
}
const customShapeAction = [ // shape
    {
        type: 'create.start-event',
        action: ['bpmn:StartEvent', 'event', 'icon-custom icon-custom-start', '开始节点']
    },
    {
        type: 'create.end-event',
        action: ['bpmn:EndEvent', 'event', 'icon-custom icon-custom-end', '结束节点']
    },
    {
        type: 'create.task',
        action: ['bpmn:Task', 'activity', 'icon-custom icon-custom-task', '普通任务']
    },
    {
        type: 'create.businessRule-task',
        action: ['bpmn:BusinessRuleTask', 'activity', 'icon-custom icon-custom-businessRule', 'businessRule任务']
    },
    {
        type: 'create.exclusive-gateway',
        action: ['bpmn:ExclusiveGateway', 'activity', 'icon-custom icon-custom-exclusive-gateway', '网关']
    },
    {
        type: 'create.dataObjectReference',
        action: ['bpmn:DataObjectReference', 'activity', 'icon-custom icon-custom-data', '变量']
    }
]
const customFlowAction = [
    flowAction
]

export { customShapeAction, customFlowAction }
 ```

 同时构建一个方法来循环创建出上面👆的元素:
 ```javascript
 // utils/util.js
/**
 * 循环创建出一系列的元素
 * @param {Array} actions 元素集合
 * @param {Object} fn 处理的函数
 */
 export function batchCreateCustom(actions, fn) {
    const customs = {}
    actions.forEach(item => {
        customs[item['type']] = fn(...item['action'])
    })
    return customs
}
 ```

 ### 编写`CustomPalette.js`代码

 之后就可以在`CustomPalette.js`中来引用它们了:
 ```javascript
 // CustomPalette.js
 import { customShapeAction, customFlowAction, batchCreateCustom } from './../../utils/util'
 PaletteProvider.prototype.getPaletteEntries = function(element) {
    var actions = {}
    const {
        create,
        elementFactory,
        globalConnect
    } = this;

    function createConnect(type, group, className, title, options) {
        return {
            group,
            className,
            title: '新增' + title,
            action: {
                click: function(event) {
                    globalConnect.toggle(event)
                }
            }
        }
    }

    function createAction(type, group, className, title, options) {
        function createListener(event) {
            var shape = elementFactory.createShape(Object.assign({ type }, options))
            create.start(event, shape)
        }

        return {
            group,
            className,
            title: '新增' + title,
            action: {
                dragstart: createListener,
                click: createListener
            }
        }
    }
    Object.assign(actions, {
        ...batchCreateCustom(customFlowAction, createConnect), // 线
        ...batchCreateCustom(customShapeAction, createAction)
    })
    return actions
}
 ```
这样看来代码是不是精简很多了呢😊.

让我们来看看页面的效果:

![bpmnModeler2.png](https://user-gold-cdn.xitu.io/2019/12/22/16f2c02305333033?w=932&h=716&f=png&s=115945)

此时左侧的工具栏就已经全部被替换成我们想要的图片了.

### 编写`CustomRenderer.js`代码

然后就到了编写`renderer`代码的时候了, 在编写之前, 同样的, 我们可以做一些配置项.

因为我们注意到在渲染自定义元素的的时候, 靠的就是`svgCreate('image', {})`这个方法.

它里面也是接收的一个图片的地址`url`和样式配置`attr`.

那么`url`的前缀我们就可以提取出来:

```javascript
 // utils/util.js
const STATICPATH = 'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/' // 静态文件路径
const customConfig = { // 自定义元素的配置
    'bpmn:StartEvent': {
        'field': 'start',
        'title': '开始节点',
        'attr': { x: 0, y: 0, width: 40, height: 40 }
    },
    'bpmn:EndEvent': {
        'field': 'end',
        'title': '结束节点',
        'attr': { x: 0, y: 0, width: 40, height: 40 }
    },
    'bpmn:SequenceFlow': {
        'field': 'flow',
        'title': '连接线',
    },
    'bpmn:Task': {
        'field': 'rules',
        'title': '普通任务',
        'attr': { x: 0, y: 0, width: 48, height: 48 }
    },
    'bpmn:BusinessRuleTask': {
        'field': 'variable',
        'title': 'businessRule任务',
        'attr': { x: 0, y: 0, width: 48, height: 48 }
    },
    'bpmn:ExclusiveGateway': {
        'field': 'decision',
        'title': '网关',
        'attr': { x: 0, y: 0, width: 48, height: 48 }
    },
    'bpmn:DataObjectReference': {
        'field': 'score',
        'title': '变量',
        'attr': { x: 0, y: 0, width: 48, height: 48 }
    }
}
const hasLabelElements = ['bpmn:StartEvent', 'bpmn:EndEvent', 'bpmn:ExclusiveGateway', 'bpmn:DataObjectReference'] // 一开始就有label标签的元素类型

export { STATICPATH, customConfig, hasLabelElements }
```

然后只需要在编写`drawShape`方法的时候判断一下就可以了:

```javascript
// CustomRenderer.js
import inherits from 'inherits'
import BaseRenderer from 'diagram-js/lib/draw/BaseRenderer'
import {
    append as svgAppend,
    create as svgCreate
} from 'tiny-svg'
import { customElements, customConfig, STATICPATH, hasLabelElements } from '../../utils/util'
/**
 * A renderer that knows how to render custom elements.
 */
export default function CustomRenderer(eventBus, styles, bpmnRenderer) {
    BaseRenderer.call(this, eventBus, 2000)
    var computeStyle = styles.computeStyle

    this.drawElements = function(parentNode, element) {
        console.log(element)
        const type = element.type // 获取到类型
        if (type !== 'label') {
            if (customElements.includes(type)) { // or customConfig[type]
                return drawCustomElements(parentNode, element)
            }
            const shape = bpmnRenderer.drawShape(parentNode, element)
            return shape
        } else {
            element
        }
    }
}

inherits(CustomRenderer, BaseRenderer)

CustomRenderer.$inject = ['eventBus', 'styles', 'bpmnRenderer']

CustomRenderer.prototype.canRender = function(element) {
    // ignore labels
    return true
        // return !element.labelTarget;
}

CustomRenderer.prototype.drawShape = function(parentNode, element) {
    return this.drawElements(parentNode, element)
}

CustomRenderer.prototype.getShapePath = function(shape) {
    // console.log(shape)
}

function drawCustomElements(parentNode, element) {
    const { type } = element
    const { field, attr } = customConfig[type]
    const url = `${STATICPATH}${field}.png`
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
            y: attr.y + attr.height + 20,
            "font-size": "14",
            "fill": "#000"
        })
        text.innerHTML = element.businessObject.name
        svgAppend(parentNode, text)
    }
    return customIcon
}
```
关键在于`drawCustomElements`函数中, 利用了`url`的一个字符串拼接.

这样的话, 自定义元素就可以都渲染出来了.

效果如下:

![bpmnModeler3.png](https://user-gold-cdn.xitu.io/2019/12/22/16f2e1fc64441d68?w=1758&h=934&f=png&s=270948)

### 编写`CustomContextProvider.js`代码

完成了`palette`和`renderer`的编写, 接下来让我们看看`contextPad`是怎么编写的.

其实它的写法和`palette`差不多, 只不过有一点需要我们注意的:

**不同类型的节点出现的`contextPad`的内容可能是不同的.**

比如:

- `StartEvent`会出现`edit、delete、Task、BusinessRuleTask、ExclusiveGateway`等等;
- `EndEvent`只能出现`edit、delete`;
- `SequenceFlow`只能出现`edit、delete`.

也就是说我们需要根据节点类型来返回不同的`contextPad`.

那么在编写`getContextPadEntries`函数返回值的时候, 就可以根据`element.type`来返回不同的结果:
```javascript
import { isAny } from 'bpmn-js/lib/features/modeling/util/ModelingUtil'
ContextPadProvider.prototype.getContextPadEntries = function(element) {
    ... // 此处省略的代码可查看项目github源码
    
    // 只有点击列表中的元素才会产生的元素
    if (isAny(businessObject, ['bpmn:StartEvent', 'bpmn:Task', 'bpmn:BusinessRuleTask', 'bpmn:ExclusiveGateway', 'bpmn:DataObjectReference'])) {
        Object.assign(actions, {
            ...batchCreateCustom(customShapeAction, createAction),
            ...batchCreateCustom(customFlowAction, createConnect), // 连接线
            'edit': editElement(),
            'delete': deleteElement()
        })
    }
    // 结束节点和线只有删除和编辑
    if (isAny(businessObject, ['bpmn:EndEvent', 'bpmn:SequenceFlow', 'bpmn:DataOutputAssociation'])) {
        Object.assign(actions, {
            'edit': editElement(),
            'delete': deleteElement()
        })
    }
    return actions
}
```
`isAny`的作用其实就是判断类型属不属于后面数组中, 类似于`includes`.

这样我们的`contextPad`就丰富起来了😊.

![bomnModeler4.png](https://user-gold-cdn.xitu.io/2019/12/22/16f2e36889de43fa?w=1084&h=774&f=png&s=169950)

## 将bpmn封装成组件

有了自定义`modeler`的基础, 我们就可以将`bpmn`封装成一个组件, 在我们需要应用的地方引用这个组件就可以了.

为了给大家更好演示, 我新建了一个项目 [bpmn-custom-modeler](https://github.com/LinDaiDai/bpmn-custom-modeler) , 里面的依赖和配置都和 [bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom)中相同, 只不过在这个新的项目里我是打算用自定义的`modeler`来覆盖它原有的, 并封装一个`bpmn`组件来供页面使用.

### 前期准备

在项目的`components`文件夹下新建一个名为`bpmn`的文件夹, 这里面用来存放封装的`bpmn`组件.

然后我们还可以准备一个空的`xml`作为组件中的默认显示(也就是若是一进来没有任何图形的时候应该显示的是什么内容), 这里我定义了一个`newDiagram.js`.

再在根目录下创建一个`views`文件来放一些页面文件, 这里我就再新建一个`custom-modeler.vue`用来引用封装好的`bpmn`组件来看效果.

### 组件的`props`

首先让我们来思考一下, 既然要把它封装成组件, 那么肯定是需要给这个组件里传递`props`(可以理解为参数). 它可以是一整个`xml`字符串, 也可以是一个`bpmn`文件的地址.

我以传入`bpmn`文件地址为例进行封装. 当然你们可以根据自己的业务需求来定.

也就是在引用这个组件的时候, 我期望的是这样写:
```html
/* views/custom-modeler.vue */
<template>
    <bpmn :xmlUrl="xmlUrl" @change="changeBpmn"></bpmn>
</template>

<script>
import { Bpmn } from './../components/bpmn'
export default {
    components: {
        Bpmn
    },
    data () {
      return {
        xmlUrl: 'https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmnMock.bpmn'
      }  
    },
    methods: {
        changeBpmn ($event) {}
    }
}
</script>
```
只要引用了`bpmn`组件, 然后传递一个`url`, 页面上就可以显示出对应的图形内容.

这样的话, 我们的`Bpmn.vue`中就应该这样定义`props`:

```javascript
// Bpmn.vue
props: {
    xmlUrl: {
      type: String,
      default: ''
    }
}
```

### 编写组件的`hmtl`代码

组件中的`html`代码十分容易, 主要是给画布一个盛放的容器, 再定义了两个按钮用于下载:

```html
<!-- Bpmn.vue -->
<template>
  <div class="containers">
    <div class="canvas" ref="canvas"></div>
    <div id="js-properties-panel" class="panel"></div>
    <ul class="buttons">
      <li>
          <a ref="saveDiagram" href="javascript:" title="保存为bpmn">保存为bpmn</a>
      </li>
      <li>
          <a ref="saveSvg" href="javascript:" title="保存为svg">保存为svg</a>
      </li>
    </ul>
  </div>
</template>
```

### 编写组件的`js`代码

在`js`里, 我就将前面几节[《全网最详bpmn.js教材-http请求篇》](https://juejin.im/post/5def468c6fb9a01622778a03) 和 [《全网最详bpmn.js教材-http事件篇》](https://juejin.im/post/5def47e16fb9a0160376e416)
中的功能都整合了进来.

大体就是:

- 初始化的时候, 对输入进来的`xmlUrl`做判断, 若是不为空的话则请求获取数据,否则赋值一个默认值;
- 初始化成功之后, 在成功的函数中添加`modeler`、`element`的监听事件;
- 初始化下载`xml、svg`的链接按钮.

例如:
```javascript
// Bpmn.vue
async createNewDiagram () {
  const that = this
  let bpmnXmlStr = ''
  if (this.xmlUrl === '') { // 判断是否存在
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
  this.addBpmnListener()
  this.addModelerListener()
  this.addEventBusListener()
},
addBpmnListener () {},
addModelerListener () {},
addEventBusListener () {}
```
整合之后的代码有些多, 这里贴出来有点不太好, 详细代码在`gitHub`上有: [LinDaiDai/bpmn-custom-modeler/Bpmn.vue](https://github.com/LinDaiDai/bpmn-custom-modeler/blob/master/src/components/bpmn/Bpmn.vue)

## 后语

项目案例Git地址: [LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-custom-modeler) 喜欢的小伙伴请给个`Star`🌟呀, 谢谢😊

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注**霖呆呆(LinDaiDai)的公众号**, 选择 **其它** 菜单中的 **bpmn.js群** 即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

