## 前言
> Q: bpmn.js是什么? 🤔️

[bpmn.js](https://bpmn.io/)是一个BPMN2.0渲染工具包和web建模器, 使得画流程图的功能在前端来完成.

> Q: 我为什么要写该系列的教材? 🤔️

因为公司业务的需要因而要在项目中使用到`bpmn.js`,但是由于`bpmn.js`的开发者是国外友人, 因此国内对这方面的教材很少, 也没有详细的文档. 所以很多使用方式很多坑都得自己去找.在将其琢磨完之后, 决定写一系列关于它的教材来帮助更多`bpmn.js`的使用者或者是期于找到一种好的绘制流程图的开发者. 同时也是自己对其的一种巩固.

由于是系列的文章, 所以更新的可能会比较频繁, **您要是无意间刷到了且不是您所需要的还请谅解**😊.

不求赞👍不求心❤️. 只希望能对你有一点小小的帮助.

## 自定义Palette篇
经过前面几章的基础教程相信大家对`bpmn.js`的基本使用已经有了一个很好的掌握.

从这一章节开始我会讲解一些关于`bpmn.js`中自定义的部分, 包括自定义左侧工具栏、自定义渲染、自定义`contextPad`等等.

还是先来看一张图了解一下我们的绘图页面都有哪些东西:

![](https://user-gold-cdn.xitu.io/2019/12/12/16ef82459a8e9ffd?w=2028&h=1560&f=png&s=791765)

这一章我要介绍的时候如何自定义左侧的工具栏(`Palette`, 也叫调色板), 通过阅读你可以学习到:

- [在默认的Palette基础上修改](#在默认的Palette基础上修改)
- [完全自定义Palette](#完全自定义Palette)

对于上面👆的目录, 其实隐含意思就是自定义`Palette`包括两种方式:
- 在`bpmn.js`默认提供的`Palette`上进行修改(或者新增新的项)
- 完全覆盖`Palette`中有的所有项, 自定义一个全新的`Palette`

## 在默认的Palette基础上修改

先来看看第一种最简单的, 我们在官方提供的调色板里新增一个自定义的项.
- 元素类型: `bpmn:Task`
- 元素名称: `lindaidai-task`
- 样式: 沿用`bpmn:Task`原有的样式, 只不过将边框变为红色
- 作用: 创建一个类型为`lindaidai-task`的任务节点

效果是这样的:

![bpmnCustom1.png](https://user-gold-cdn.xitu.io/2019/12/12/16ef84b06e189b74?w=1770&h=1586&f=png&s=228988)

如上所示, 只改变了任务框的颜色为红色, 所以效果不是很明显, 你甚至可以直接给它换一个样貌:

![bpmnCustom2.png](https://user-gold-cdn.xitu.io/2019/12/12/16ef95af4a7ae842?w=1832&h=1598&f=png&s=244441)

接下来让我们看看该怎么实现它吧!🐶

### 前期准备

因为是新的章节, 这里我也新建一个项目:
```
$ vue create bpmn-vue-custom
$ npm i vue-router axios bpmn-js-properties-panel bpmn-js --save-D
```
按照之前的案例[LinDaiDai/bpmn-vue-basic](https://github.com/LinDaiDai/bpmn-vue-basic)配置好相应的路由之类的东西.

在`components`文件夹下新建一个名为`custom-palette.vue`的文件, 并将[provider.vue](https://github.com/LinDaiDai/bpmn-vue-basic/blob/master/src/components/provider.vue)(之前的一个基础案例) 的内容复制进去.

继续在`components`文件夹下新建文件夹`custom`用于盛放我们后面要写的一些自定义的东西.

来看看我们现在的项目结构:

![](https://user-gold-cdn.xitu.io/2019/12/12/16ef8aa413fdae75?w=492&h=792&f=png&s=76499)

我已经在`custom`文件夹新建立了一个`CustomPalette.js`, 接下来就是要在这里面写上我们要自定义的项.

### 编写`CustomPalette.js`代码

首先这个`js`是导出一个类(类的名称你可以随意取, 但是在引用的时候不能随意取, 后面会说到):

这里我就取为`CustomPalette`:
```javascript
// CustomPalette.js
export default class CustomPalette {
    constructor(bpmnFactory, create, elementFactory, palette, translate) {
        this.bpmnFactory = bpmnFactory;
        this.create = create;
        this.elementFactory = elementFactory;
        this.translate = translate;

        palette.registerProvider(this);
    }
    // 这个函数就是绘制palette的核心
    getPaletteEntries(element) {}
}

CustomPalette.$inject = [
    'bpmnFactory',
    'create',
    'elementFactory',
    'palette',
    'translate'
]
```
上面👆的代码很好理解:
- 定义一个类
- 使用`$inject`注入一些需要的变量
- 在类中使用`palette.registerProvider(this)`指定这是一个`palette`

定义完`CustomPalette.js`之后, 我们需要在其同级的`index.js`中将它导出:
```javascript
// custom/index.js
import CustomPalette from './CustomPalette'

export default {
    __init__: ['customPalette'],
    customPalette: ['type', CustomPalette]
}
```
**注:️**
这里`__init__`中的名字就必须是`customPalette`了, 还有下面的属性名也必须是`customPalette`, 不然就会报错了.

同时要在页面中使用它:
```html
<!--custom-palette.vue-->
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

</script>
```

### 编写核心函数`getPaletteEntries`代码

抛开这些不看, 重点就是如何构造这个`getPaletteEntries`函数

函数的名称你不能变, 不然会报错, 首先它返回的是一个对象, 对象中指定的就是你要自定义的项, 它大概长成这样:
```javascript
// CustomPalette.js
getPaletteEntries(element) {
    return {
        'create.lindaidai-task': {
            group: 'model', // 分组名
            className: 'bpmn-icon-task red', // 样式类名
            title: translate('创建一个类型为lindaidai-task的任务节点'),
            action: { // 操作
                dragstart: createTask(), // 开始拖拽时调用的事件
                click: createTask() // 点击时调用的事件
            }
        }
    }
}
```
可以看到我定义的一项的名称就是: `create.lindaidai-task`. 它会有几个固定的属性:
- group: 属于哪个分组, 比如`tools、event、gateway、activity`等等,用于分类
- className: 样式类名, 我们可以通过它给元素修改样式
- title: 鼠标移动到元素上面给出的提示信息
- action: 用户操作时会触发的事件

接下来我们要做的无非就是:

1. 通过`className`来设置样式
2. 通过`action`来定义要触发的事情

### 编写`className`代码
我在`scr`的目录下新建了一个`css`文件, 里面用来盛放一些全局的样式, 并在`main.js`中引用这个全局样式:
```javascript
// main.js
// 引入全局的css
import './css/app.css'
```
然后在其中加上一下样式:
```css
/* app.css */
.bpmn-icon-task.red {
    color: #cc0000 !important;
}
```
上面👆的`className`我之所以要用`bpmn-icon-task`, 是因为这个类是`bpmn.js`中自带的一个`iconfont`类, 使用它就可以实现一个`task`的图标的效果:

![](https://user-gold-cdn.xitu.io/2019/12/12/16ef8f79d1bc43cc?w=1388&h=1258&f=png&s=1609172)

由于`iconfont`是一个字体, 所以这里我使用`color`来改变它的颜色.

如果你想要给它完全换一张图片的话也可以用`className`来实现:
```css
/* app.css */
.icon-custom { /* 定义一个公共的类名 */
    border-radius: 50%;
    background-size: 65%;
    background-repeat: no-repeat;
    background-position: center;
}

.icon-custom.lindaidai-task { /* 加上背景图 */
    background-image: url('https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/rules.png');
}
```
然后修改`create.lindaidai-task`中的`className`:
```javascript
// CustomPalette.js
 'create.lindaidai-task': {
    className: 'icon-custom lindaidai-task' 
 }
```
这样页面上显示的就是你定义的那张背景图了:

![bpmnCustom4.png](https://user-gold-cdn.xitu.io/2019/12/12/16ef9636c778fb2c?w=744&h=490&f=png&s=49564)
### 编写`action`代码
完成了上面的操作, 其实页面已经能正常渲染出一个我们自定义的元素了, 但是你在点击或者拖拽它的时候是没有效果的💦.

此时我们期望的是点击或者拖拽它能在画布中画出一个`lindaidai-task`, 因此你得给它加上事件,
也就是编写一个函数用来创建`bpmn:Task`这个元素:
```
// CustomPalette.js
function createTask() {
    return function(event) {
        const businessObject = bpmnFactory.create('bpmn:Task');
        const shape = elementFactory.createShape({
            type: 'bpmn:Task',
            businessObject
        });
        console.log(shape) // 只在拖动或者点击时触发
        create.start(event, shape);
    }
}
```
这里的核心其实就是利用`bpmn.js`提供的一些方法创建`shape`然后将其添加到画布上.

(我这里演示的是创建一个类型为`bpmn:Task`的元素, 你还可以用来创建`bpmn:StartEvent、bpmn:ServiceTask、bpmn:ExclusiveGateway`等等...)

此时你拖动或者点击`lindaidai-task`就可以在页面上创建一个`Task`元素了.


![](https://user-gold-cdn.xitu.io/2019/12/12/16ef982d03ea6b1c?w=1784&h=1600&f=png&s=331804)

我们看到虽然`lindaidai-task`在左侧工具栏中是金黄金黄的, 但是实际画到页面却还是呈现“裸体”状态😅, 这就和自定义渲染有关系了, 不要着急, 这些在后面的章节中会讲到.

### 完整的`CustomPalette.js`代码
让我们将上面的所有代码整合一下:
```javascript
// CustomPalette.js
export default class CustomPalette {
    constructor(bpmnFactory, create, elementFactory, palette, translate) {
        this.bpmnFactory = bpmnFactory;
        this.create = create;
        this.elementFactory = elementFactory;
        this.translate = translate;

        palette.registerProvider(this);
    }

    getPaletteEntries(element) {
        const {
            bpmnFactory,
            create,
            elementFactory,
            translate
        } = this;

        function createTask() {
            return function(event) {
                const businessObject = bpmnFactory.create('bpmn:Task'); // 其实这个也可以不要
                const shape = elementFactory.createShape({
                    type: 'bpmn:Task',
                    businessObject
                });
                console.log(shape) // 只在拖动或者点击时触发
                create.start(event, shape);
            }
        }

        return {
            'create.lindaidai-task': {
                group: 'model',
                className: 'icon-custom lindaidai-task',
                title: translate('创建一个类型为lindaidai-task的任务节点'),
                action: {
                    dragstart: createTask(),
                    click: createTask()
                }
            }
        }
    }
}

CustomPalette.$inject = [
    'bpmnFactory',
    'create',
    'elementFactory',
    'palette',
    'translate'
]
```
项目案例Git地址: [LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom)

**注意: 项目案例里我为了方便演示, 在`custom-palette`中引入的是`ImportJS/onlyPalette.js`, 而上面的案例是以引入`custom/index.js`为讲解的, 这个自己要明白如何区分.**

## 完全自定义Palette
可以看到, 上面👆的那种实现方式实际上就是定义了一个`CustomPalette`然后在`new BpmnModeler`生成的对象中引用进去.

但是这样做有一点不好👎, 那就是如果你不想要它提供的默认的这些项, 比如开始节点、结束节点、任务节点, 而是全都是自己自定义的, 就不能满足了. 比如这样:

![bpmnCustom6.png](https://user-gold-cdn.xitu.io/2019/12/12/16ef9d3a15d9f87e?w=1798&h=1698&f=png&s=345521)

此时你就需要重写`BpmnModeler`这个类了, 实现自己独有的一套`modeler`.

### 前期准备

继续在上面👆的项目的基础上创建一个`customModeler`文件夹和一个`custom-modeler.vue`文件.
然后在`customModeler`中创建一个`index.js`和一个`custom`文件夹.

- `customModeler`文件夹下的文件就是用来放自定义的`modeler`
- `custom-modeler.vue`作为页面展示(记得配置页面的路由)

此时项目结构变成了:

![bpmnCustom7.png](https://user-gold-cdn.xitu.io/2019/12/12/16ef9f604274be18?w=496&h=928&f=png&s=263285)

### 编写`CustomPalette.js`代码
这里的`CustomPalette.js`的编写方式就和第一种的有所不同了:
```javascript
/**
 * A palette that allows you to create BPMN _and_ custom elements.
 */
export default function PaletteProvider(palette, create, elementFactory, globalConnect) {
    this.create = create
    this.elementFactory = elementFactory
    this.globalConnect = globalConnect

    palette.registerProvider(this)
}

PaletteProvider.$inject = [
    'palette',
    'create',
    'elementFactory',
    'globalConnect'
]

PaletteProvider.prototype.getPaletteEntries = function(element) { // 此方法和上面案例的一样
    const {
        create,
        elementFactory
    } = this;

    function createTask() {
        return function(event) {
            const shape = elementFactory.createShape({
                type: 'bpmn:Task'
            });
            console.log(shape) // 只在拖动或者点击时触发
            create.start(event, shape);
        }
    }

    return {
        'create.lindaidai-task': {
            group: 'model',
            className: 'icon-custom lindaidai-task',
            title: '创建一个类型为lindaidai-task的任务节点',
            action: {
                dragstart: createTask(),
                click: createTask()
            }
        }
    }
}
```
在这里是直接重写了`PaletteProvider`这个类, 同时覆盖了其原型上的`getPaletteEntries`方法, 从而达到覆盖原有的工具栏的效果.

(别看上面👆写的东西好像很多的样子, 但是其实静下心来看发现也没啥😊)

### 编写`custom/index.js`代码
接下来还是和第一种方式一样, 需要将我们自定义的`Palette`导出:
```javascript
// custom/index.js
import CustomPalette from './CustomPalette'

export default {
    __init__: ['paletteProvider'],
    paletteProvider: ['type', CustomPalette]
}
```
这不过这里我们就不是用`customPalette`了, 而是直接用`paletteProvider`.

### 编写`customModeler/index.js`代码
最重要的一步, 就是编写`CustomModeler`这个类了:
```javascript
import Modeler from 'bpmn-js/lib/Modeler'

import inherits from 'inherits'

import CustomModule from './custom'

export default function CustomModeler(options) {
    Modeler.call(this, options)

    this._customElements = []
}

inherits(CustomModeler, Modeler)

CustomModeler.prototype._modules = [].concat(
    CustomModeler.prototype._modules, [
        CustomModule
    ]
)
```
导出的类继承了`Modeler`这个核心的类, 这样就保证了其他功能的实现.

### 在页面上引用
最后一步, 是需要将我们原本通过`BpmnModeler`创建的对象改为通过我们自定义的`CustomModeler`来创建, 编写`custom-modeler.vue`.
```html
<!--custom-modeler.vue-->
<script>
...
import CustomModeler from './customModeler'
...
this.bpmnModeler = new CustomModeler({ // 原本是用BpmnModeler
    ...
    additionalModules: [] // 可以不用引用任何东西
})

</script>
```
快来打开页面看看效果:

![bpmnCustom8.png](https://user-gold-cdn.xitu.io/2019/12/12/16efa06eb8035cab?w=1840&h=1134&f=png&s=215893)

## 后语
上面👆两个案例用的都是同一个项目🦐

项目案例Git地址: [LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom) 喜欢的小伙伴请给个`Star`🌟呀, 谢谢😊

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注**霖呆呆(LinDaiDai)的公众号**, 选择 **其它** 菜单中的 **bpmn.js群** 即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

