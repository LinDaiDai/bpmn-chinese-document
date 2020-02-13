## Properties-panel篇(下)

在上一章节主要介绍了如何在原有`properties-panel`的基础上进行扩展, 但是有很多小伙伴就会说我太嫌弃原有属性栏的样式了 😅...我是一名成熟的前端了, 我要有自己的想法...

OK... 我尊重你...这一章节霖呆呆就来教教大家怎样美化我们的`properties-panel`😊.

通过这一章节的阅读你可以学习到:

- 修改属性栏的默认样式
- 自定义`properties-panel`
- 修改节点名称`label`属性
- 修改节点颜色`color`属性
- 修改`event`节点类型
- 修改`Task`节点的类型
- 初始化`properties-panel`并设置一些默认值

## 修改属性栏的默认样式

先来看看我们通过修改属性栏的默认样式可以实现什么样的效果🤔️吧!

![绯红主题](https://user-gold-cdn.xitu.io/2020/1/20/16fc1809d604ecd6?w=1640&h=848&f=png&s=290951)


![科技蓝主题](https://user-gold-cdn.xitu.io/2020/1/20/16fc180b306f95c3?w=1650&h=786&f=png&s=295338)


![极客黑主题](https://user-gold-cdn.xitu.io/2020/1/20/16fc1814030dae00?w=1640&h=774&f=png&s=204151)


如上👆所示, 你可以给属性栏定制不同的主题颜色, 来美化它原本的样子.

其实想要修改默认属性栏的样式, 非常简单, 只要打开控制台(Window: F12, Mac : option + command + i)通过审查元素, 找到各个元素的`class`, 然后在代码里覆盖它原有的属性就可以了.

还记得我们之前在项目的`main.js`中引用了`properties-panel`的样式吗?

```javascript
// main.js

import 'bpmn-js-properties-panel/dist/assets/bpmn-js-properties-panel.css' // 右边工具栏样式
```

现在让我们在项目中创建一个`styles`文件夹, 同时创建一个`bpmn-properties-theme-red.css`文件, 里面将用来编写我们需要自定义修改的属性栏样式.

之后在`main.js` 中引用它, 最好是放在原有样式的后面:

```javascript
// main.js

import 'bpmn-js-properties-panel/dist/assets/bpmn-js-properties-panel.css' // 右边工具栏样式
import './styles/bpmn-properties-theme-red.css' // 绯红主题
```

比如现在我想要修改一下属性栏头部的字体颜色:

![](https://user-gold-cdn.xitu.io/2020/1/20/16fc1822e4b5698d?w=1960&h=842&f=png&s=1084506)

通过审查元素找到这个类, 然后在`bpmn-properties-theme-red.css`中修改它:

```css
.bpp-properties-header>.label {
    color: rgb(239, 112, 96);
    font-size: 16px;
}
```

保存再次打开页面就可以看到效果了.

当然我这里只是演示一下可以怎样去修改默认的样式, 所以只是用了最简单的`css`来演示. 这里其实有很大的扩展空间, 你可以用`less`或者`sass`来编写, 也可以自己实现一下主题切换等等的功能. 抛砖引玉希望能给你启发 😊...

如果你想偷会懒...直接取霖呆呆的样式也行...

上面👆案例的`github`地址: 


## 自定义`properties-panel`

有时候你可能不满足用官方提供的`properties-panel`, 而是想要自定义一个属性栏, 这也是可以实现的.

比如我想要根据不同的节点类型, 在右边显示不同的属性配置, 并且编辑完之后可以同步更新到`xml`上.

![自定义properties-panel](https://user-gold-cdn.xitu.io/2020/1/20/16fc1cdb97617377?w=3200&h=1692&f=png&s=1833510)

其实实现的原理在之前的 [《全网最详bpmn.js教材-properties篇》](https://juejin.im/post/5e19f17e6fb9a02fea372b37)中也有提过了, 主要是利用`updateProperties()`这个方法来修改元素节点上的属性.

现在就让我们来看看如何封装一个这样的自定义属性栏吧😊.

### 前期准备

由于自定义属性栏的代码可能会很多, 而且可能还会涉及到很多复杂的业务组件, 所以我建议你将其从`引入bpmn.js`的地方给抽离出来, 也就是封装成一个通用的自定义属性栏组件.

**组件的`props`**

既然决定将其抽离成组件了, 那么这个组件的`props`应该设置成什么呢?

(`props`即父组件向子组件传递的值, 在这里父元素就是引入`bpmn.js`的地方, 子元素为自定义属性栏组件)

先来让我们理理我们的需求, 我们需要点击不同的元素来呈现不同的配置, 那么可以将单个`element`作为`props`传递进去.

不过后来在编写的过程中, 我发现有很多事件的绑定都是要涉及到`modeler`的, 若是将这些绑定事件都在父组件中完成不就违背了我们抽离出单独组件的意愿了吗🤔️?

所以在这里, 我是将整个`modeler`作为`props`来编写.这样不管是给`modeler`绑定事件还是给`element`绑定事件都很好做了.

OK...考虑好`props`, 让我们在`components`文件夹下创建一个`custom-properties-panel`的文件夹, 并在其中创建一个名为`PropertiesView.vue`的文件, 用来编写我们的自定义属性栏组件.

我们期望的这个组件是能够这样在`html`中使用:

```html
<div class="containers" ref="content">
    <div class="canvas" ref="canvas"></div>
    <properties-view v-if="bpmnModeler" :modeler="bpmnModeler"></properties-view>
</div>
```

(`bpmnModeler`是你使用`new BpmnModeler`创建的`modeler`对象)

### 编写自定义属性栏组件

#### 1. 组件结构

先来将这个组件的基础结构给搭好:

```html
<!--PropertiesView.vue-->
<template>
    <div class="custom-properties-panel"></div>
</template>
<script>
export default {
    name: 'PropertiesView',
    props: {
        modeler: {
          type: Object,
          default: () => ({})
        }
    },
    data () {
        return {
            selectedElements: [], // 当前选择的元素集合
            element: null // 当前点击的元素
        }
    },
    created () {
        this.init()
    },
    methods: {
       init () {} 
    }
}
</script>
<style scoped></style>
```

#### 2. 组件`html`代码

先让我给这个组件里添加点东西:
```html
<template>
  <div class="custom-properties-panel">
    <div class="empty" v-if="selectedElements.length<=0">请选择一个元素</div>
    <div class="empty" v-else-if="selectedElements.length>1">只能选择一个元素</div>
    <div v-else>
      <fieldset class="element-item">
        <label>id</label>
        <span>{{ element.id }}</span>
      </fieldset>
      <fieldset class="element-item">
        <label>name</label>
        <input :value="element.name" @change="(event) => changeField(event, 'name')" />
      </fieldset>
      <fieldset class="element-item">
        <label>customProps</label>
        <input :value="element.name" @change="(event) => changeField(event, 'customProps')" />
      </fieldset>
    </div>
  </div>
</template>
```

如上👆, 我增加了三个属性, `id, name, customProps`. 同时, 有一个`selectedElements`的判断.

这是因为我们在操作图形的时候, 如果你使用`command + 左键`(window上应该是`Ctrl`?)是可以选择多个节点的, 这时候就需要做一个判断.

#### 3. 组件的`js`代码

如果你看多了霖呆呆写的代码, 你会发现我比较喜欢将一些初始化的代码提到一个叫做`init()`的函数中来, 这个是个人编码习惯哈...

在这里, 我们的初始化函数主要做以下几件事:

- 使用`selection.changed`监听选中的元素;
- 使用`element.changed`监听发生改变的元素.

```javascript
init () {
 const { modeler } = this // 父组件传递进来的 modeler
  modeler.on('selection.changed', e => {
    this.selectedElements = e.newSelection // 数组, 可能有多个
    this.element = e.newSelection[0] // 默认取第一个
  })
  modeler.on('element.changed', e => {
    const { element } = e
    const { element: currentElement } = this
    if (!currentElement) {
      return
    }
    // update panel, if currently selected element changed
    if (element.id === currentElement.id) {
      this.element = element
    }
  })
}
```

另外, 我们可以写一个公用的属性更新方法, 用来更新元素上的属性:

```javascript
/**
 * 更新元素属性
 * @param { Object } 要更新的属性, 例如 { name: '', id: '' }
 */
updateProperties(properties) {
  const { modeler, element } = this
  const modeling = modeler.get('modeling')
  modeling.updateProperties(element, properties)
}
```

然后给属性栏上的`input`或者其它的控件, 增加一个`@change`事件, 当控件内的内容发生改变时, 同步更新`element`.

```javascript
/**
* 改变控件触发的事件
* @param { Object } input的Event
* @param { String } 要修改的属性的名称
*/
changeField (event, type) {
  const value = event.target.value
  let properties = {}
  properties[type] = value
  this.element[type] = value
  this.updateProperties(properties) // 调用属性更新方法
}
```

#### 4. 完整的组件代码

将上面👆的所有代码组合起来:

```html
<template>
  <div class="custom-properties-panel">
    <div class="empty" v-if="selectedElements.length<=0">请选择一个元素</div>
    <div class="empty" v-else-if="selectedElements.length>1">只能选择一个元素</div>
    <div v-else>
      <fieldset class="element-item">
        <label>id</label>
        <span>{{ element.id }}</span>
      </fieldset>
      <fieldset class="element-item">
        <label>name</label>
        <input :value="element.name" @change="(event) => changeField(event, 'name')" />
      </fieldset>
      <fieldset class="element-item">
        <label>customProps</label>
        <input :value="element.name" @change="(event) => changeField(event, 'customProps')" />
      </fieldset>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PropertiesView',
  props: {
    modeler: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      selectedElements: [],
      element: null
    }
  },
  created() {
    this.init()
  },
  methods: {
    init() {
      const { modeler } = this
      modeler.on('selection.changed', e => {
        this.selectedElements = e.newSelection
        this.element = e.newSelection[0]
      })
      modeler.on('element.changed', e => {
        const { element } = e
        const { element: currentElement } = this
        if (!currentElement) {
          return
        }
        // update panel, if currently selected element changed
        if (element.id === currentElement.id) {
          this.element = element
        }
      })
    },
    /**
    * 改变控件触发的事件
    * @param { Object } input的Event
    * @param { String } 要修改的属性的名称
    */
    changeField(event, type) {
      const value = event.target.value
      let properties = {}
      properties[type] = value
      this.element[type] = value
      this.updateProperties(properties)
    },
    updateName(name) {
      const { modeler, element } = this
      const modeling = modeler.get('modeling')
      // modeling.updateLabel(element, name)
      modeling.updateProperties(element, {
        name
      })
    },
    /**
     * 更新元素属性
     * @param { Object } 要更新的属性, 例如 { name: '' }
     */
    updateProperties(properties) {
      const { modeler, element } = this
      const modeling = modeler.get('modeling')
      modeling.updateProperties(element, properties)
    }
  }
}
</script>

<style scoped>
/** 更多代码在git上有 **/
.custom-properties-panel {
  position: absolute;
  right: 0;
  top: 0;
  width: 300px;
  background-color: #fff9f9;
  border-color: rgba(0, 0, 0, 0.09);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.09);
  padding: 20px;
}
</style>
```

## 修改节点名称`label`属性

在上面的例子中, 我们演示了如果修改元素属性的, 如果你想要修改一个元素的`label`, 一种方式是像上面👆一样, 修改`name`这个属性, 或者用`modeling.updateLabel`这个方法更新也是一样的:

```javascript
updateName(name) {
  const { modeler, element } = this
  const modeling = modeler.get('modeling')
  modeling.updateLabel(element, name)
  // 等同于 modeling.updateProperties(element, { name })
},
```

## 修改节点颜色`color`属性

如何让用户手动修改节点的颜色呢?

![设置fill](https://user-gold-cdn.xitu.io/2020/1/20/16fc2131b589075a?w=1766&h=724&f=png&s=275738)

可以利用`modeling.setColor`这个方法.

比如我在代码中添加一行属性:

```html
<fieldset class="element-item">
    <label>节点颜色</label>
    <input type="color" :value="element.color" @change="(event) => changeField(event, 'color')" />
  </fieldset>
```

然后改造以下`changeField`方法:

```javascript
/**
 * 改变控件触发的事件
 * @param { Object } input的Event
 * @param { String } 要修改的属性的名称
 */
changeField(event, type) {
  const value = event.target.value
  let properties = {}
  properties[type] = value
  if (type === 'color') { // 若是color属性
    this.onChangeColor(value)
  }
  this.element[type] = value
  this.updateProperties(properties)
},
onChangeColor(color) {
  const { modeler, element } = this
  const modeling = this.modeler.get('modeling')
  modeling.setColor(element, {
    fill: color,
    stroke: null
  })
},
```

`setColor`这个方法接收两个属性:

- `fill`: 节点的填充色
- `stroke`: 节点边框的颜色和节点`label`的颜色

在上面我演示的是修改节点的填充色, 也就是`fill`, 当然你也可以改变`stroke`, 效果是这样的:

![设置stroke](https://user-gold-cdn.xitu.io/2020/1/20/16fc2169e37c817a?w=1750&h=726&f=png&s=294068)

有意思的是, 如果你把`fill`和`stroke`都设置成了`color`:

```javascript
modeling.setColor(element, {
    fill: color,
    stroke: color
})
```
那么`label`标签就看不到了... 这是因为`stroke`也会改变`label`的颜色, 让它变得和`fill`一样.

![设置fill和stroke](https://user-gold-cdn.xitu.io/2020/1/20/16fc219b3c70aea5?w=1736&h=774&f=png&s=278033)

不过一般你也不会将`边框和填充内容`设置成一个色吧...没必要...

如果你实在是想要解决这个问题的话, 这里有个不靠谱的做法, 就是在全局的`css`中, 将`label`的样式强行修改一下:

```css
.djs-label {
    fill: #000!important;
}
```

## 修改`event`节点类型

有些时候, 我们可能还需要在自定义属性栏中修改这个节点的类型, 比如在开始节点, 点击`contextPad`上的小扳手:

![](https://user-gold-cdn.xitu.io/2020/1/20/16fc279b6c6ccde7?w=1796&h=1094&f=png&s=447650)

实现这个功能我们需要用到`bpmnReplace.replaceElement`这个方法.

首先让我们看看`event`里这个属性是放在哪里的.

如下图: 我修改了一下开始节点的类型, 将它改为`MessageEventDefinition`

![](https://user-gold-cdn.xitu.io/2020/1/20/16fc27cceb8051de?w=2446&h=1162&f=png&s=1056960)

它对应是放在`element.businessObject.eventDefinitions`这个数组中的, 若是`StartEvent`和`EndEvent`, 则这个数组为`undefinded`.

让我们来看看这个功能怎么实现哈 😄.

首先在`html`加上`修改event节点类型`的下拉框:

```html
<!--PropertiesView.vue-->
<template>
    <fieldset class="element-item" v-if="isEvent">
        <label>修改event节点类型</label>
        <select @change="changeEventType" :value="eventType">
          <option
            v-for="option in eventTypes"
            :key="option.value"
            :value="option.value"
          >{{ option.label }}</option>
        </select>
  </fieldset>
</template>
<script>
export default {
    data () {
        return {
            eventTypes: [
                { label: '默认', value: '' },
                { label: 'MessageEventDefinition', value: 'bpmn:MessageEventDefinition' },
                { label: 'TimerEventDefinition', value: 'bpmn:TimerEventDefinition' },
                { label: 'ConditionalEventDefinition', value: 'bpmn:ConditionalEventDefinition' }
          ],
          eventType: ''
        }
    },
    methods: {
        verifyIsEvent (type) { // 判断类型是不是event
            return type.includes('Event')
        },
        changeEventType (event) {}
    },
    computed: {
        isEvent() { // 判断当前点击的element类型是不是event
          const { element } = this
          return this.verifyIsEvent(element.type)
        }
    }
}
</script>
```
好了, 完成上面👆的基础代码, 主要逻辑就是在改变下拉框值的时候了:

```javascript
changeEventType(event) { // 改变下拉框
  const { modeler, element } = this
  const value = event.target.value
  const bpmnReplace = modeler.get('bpmnReplace')
  this.eventType = value
  bpmnReplace.replaceElement(element, {
    type: element.businessObject.$type,
    eventDefinitionType: value
  })
},
```

现在改变下拉框的值, 就可以改变`eventDefinitionType`的值了, 不过还有一个问题, 就是你点击了其它的节点, 然后再次点回开始节点的时候, 下拉框的默认值就不对了, 也就是说我们还需要获取到这个开始节点本身的`eventDefinitionType`值.

这时候, 我们可以在`selection.changed`监听事件中做这类初始化`properties-panel`的事情.

```javascript
init () {
    modeler.on('selection.changed', e => {
        this.selectedElements = e.newSelection
        this.element = e.newSelection[0]
        console.log(this.element)
        this.setDefaultProperties() // 设置一些默认的值
      })
}
setDefaultProperties() {
  const { element } = this
  if (element) {
    const { type, businessObject } = element
    if (this.verifyIsEvent(type)) { // 若是event类型
      // 获取默认的 eventDefinitionType
      this.eventType = businessObject.eventDefinitions ? businessObject.eventDefinitions[0]['$type'] : ''
    }
  }
}
```

## 修改`Task`节点的类型

`event`类型的节点我们已经知道怎么修改了, 那么对于`Task`类型的节点呢 🤔️? 

其实做法都差不多.

![](https://user-gold-cdn.xitu.io/2020/1/20/16fc29ab2b9a0304?w=1516&h=860&f=png&s=288623)

同样, 让我们在`html`中加上针对`Task`类型的属性下拉框:

```html
<!--PropertiesView.vue-->
<template>
    <fieldset class="element-item" v-if="isTask">
        <label>修改Task节点类型</label>
        <select @change="changeTaskType" :value="taskType">
          <option
            v-for="option in taskTypes"
            :key="option.value"
            :value="option.value"
          >{{ option.label }}</option>
        </select>
    </fieldset>
</template>
<script>
export default {
    data () {
        return {
            taskTypes: [
            { label: 'Task', value: 'bpmn:Task' },
            { label: 'ServiceTask', value: 'bpmn:ServiceTask' },
            { label: 'SendTask', value: 'bpmn:SendTask' },
            { label: 'UserTask', value: 'bpmn:UserTask' }
          ],
          taskType: ''
        }
    },
    methods: {
        verifyIsTask(type) {
            return type.includes('Task')
        },
        changeTaskType (event) {}
    },
    computed: {
        isTask() { // 判断当前点击的element类型是不是task
          const { element } = this
          return this.verifyIsTask(element.type)
        }
    }
}
</script>
```

然后在改变`Task`下拉框的时候:

```javascript
changeTaskType(event) {
  const { modeler, element } = this
  const value = event.target.value // 当前下拉框选择的值
  const bpmnReplace = modeler.get('bpmnReplace')
  bpmnReplace.replaceElement(element, {
    type: value // 直接修改type就可以了
  })
}
```


## 初始化`properties-panel`并设置一些默认值

我们在设置自己的自定义属性栏的时候, 可能要根据不同的节点类型来做不同的业务逻辑判断, 并对`properties-panel`做一些默认值的设置, 比如上面👆的`修改event类型`, 这时候我们可以怎么样做呢 🤔️?

和`修改event类型`一样, 我们可以在`selection.changed`监听事件中完成这个功能.

```javascript
init () {
    modeler.on('selection.changed', e => {
        this.selectedElements = e.newSelection
        this.element = e.newSelection[0]
        console.log(this.element)
        this.setDefaultProperties() // 设置一些默认的值
      })
}
setDefaultProperties() {
  const { element } = this
  if (element) {
    // 这里可以拿到当前点击的节点的所有属性
    const { type, businessObject } = element
    // doSomeThing
  }
}
```

其实就是和上面👆介绍`修改event类型`的初始化一样, 不过我怕有的小伙伴直接跳过了`修改event类型`没有看到这一部分, 所以单独拎出来说下.

比如我们想要从`Shape`里获取到`label`然后同步到右侧的自定义属性栏里可以这样做:

在`setDefaultProperties`里我们可以通过`this.element`拿到当前点击的这个元素, 将这个元素打印出来会发现, `label`实际上是`businessObject`对象中的`name`属性, 所以我们只需要做一下处理:

```javascript
element['name'] = businessObject.name
```

这样你不管在修改图上面的`label`还是修改自定义属性栏里的`name`都会同步更新了, 具体可以看github中的代码.



## `replace`的类型

在上面👆我们介绍了关于`Event`和`Task`类型的元素是如何转化类型的, 案例中也仅仅演示了几种类型, 那么全部的类型到哪里看呢 🤔️?

你可以在`bpmn.js`的源码这里找到:

```
https://github.com/bpmn-io/bpmn-js/blob/develop/lib/features/replace/ReplaceOptions.js
```

你甚至可以直接到代码中将里面你要的内容导出:

```javascript
import { START_EVENT } from 'bpmn-js/lib/features/replace/ReplaceOptions.js'
```

## 后语

上面👆教材案例的代码地址: [LinDaiDai/bpmn-vue-properties-panel](https://github.com/LinDaiDai/bpmn-vue-properties-panel)

截止到本章节, `properties-panel`算是介绍大概了, 不管是要使用原有的`properties-panel`还是使用自定义`properties-panel`我相信你都已经掌握了 😄...

在后续霖呆呆可能会根据`bpmn.js`源码来列举一些常用的属性和方法, 以便你更好的了解`bpmn.js`.

马上要过年了🧨了...

码完了这一章节, 霖呆呆也要开始整理回家的行李了 😄 ...

再次祝大家新年快乐呀~ 🔥 🎆

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注霖呆呆的公众号, 选择“其它”菜单中的“bpmn.js群”即可😊.

![LinDaiDai公众号二维码.jpg](../resource/LinDaiDai公众号二维码.jpg)


