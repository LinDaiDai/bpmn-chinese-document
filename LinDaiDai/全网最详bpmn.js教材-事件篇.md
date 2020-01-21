## 前言
> Q: bpmn.js是什么? 🤔️

[bpmn.js](https://bpmn.io/)是一个BPMN2.0渲染工具包和web建模器, 使得画流程图的功能在前端来完成.

> Q: 我为什么要写该系列的教材? 🤔️

因为公司业务的需要因而要在项目中使用到`bpmn.js`,但是由于`bpmn.js`的开发者是国外友人, 因此国内对这方面的教材很少, 也没有详细的文档. 所以很多使用方式很多坑都得自己去找.在将其琢磨完之后, 决定写一系列关于它的教材来帮助更多`bpmn.js`的使用者或者是期于找到一种好的绘制流程图的开发者. 同时也是自己对其的一种巩固.

由于是系列的文章, 所以更新的可能会比较频繁, **您要是无意间刷到了且不是您所需要的还请谅解**😊.

不求赞👍不求心❤️. 只希望能对你有一点小小的帮助.

## 事件篇

上一章节我们介绍了利用`bpmn.js`与后台进行交互, 要是对`bpmn.js`不了解的小伙请移步:

[《全网最详bpmn.js教材-http请求篇》](https://juejin.im/post/5def468c6fb9a01622778a03)

这一章节要讲解是关于`bpmn.js`的一些事件, 通过学习此章节你可以学习到:

- [监听modeler并绑定事件](#监听modeler并绑定事件)
- [监听element并绑定事件](#监听element并绑定事件)
- [通过监听事件判断操作方式](#通过监听事件判断操作方式)



### 监听modeler并绑定事件

很多时候你期望的是在用户在进行不同操作的时候能够监听到他操作的是什么, 从而做想要做的事情.

是进行了`shape`的新增还是进行了线的新增.

比如如下的一些监听事件:

- shape.added 新增一个`shape`之后触发;
- shape.move.end 移动完一个`shape`之后触发;
- shape.removed 删除一个`shape`之后触发;



继续在项目案例[bpmn-vue-basic](https://github.com/LinDaiDai/bpmn-vue-basic)的基础上创建一个`event.vue`文件:

并在`success()`函数中添加上监听事件的函数:

```vue
// event.vue
<script>
...
success () {
  this.addModelerListener()
},
// 监听 modeler
addModelerListener() {
  const bpmnjs = this.bpmnModeler
  const that = this
  // 这里我是用了一个forEach给modeler上添加要绑定的事件
  const events = ['shape.added', 'shape.move.end', 'shape.removed', 'connect.end', 			'connect.move']
  events.forEach(function(event) {
    that.bpmnModeler.on(event, e => {
      console.log(event, e)
      var elementRegistry = bpmnjs.get('elementRegistry')
      var shape = e.element ? elementRegistry.get(e.element.id) : e.shape
      console.log(shape)
    })
  })
},
```

如图所示, 在这里你就可以获取到相关节点的所有信息了:

![img1](https://user-gold-cdn.xitu.io/2019/12/10/16eeeb1268d45fe9?w=3246&h=1816&f=jpeg&s=482230)



案例Git地址: [LinDaiDai-bpmn.js案例event.vue](https://github.com/LinDaiDai/bpmn-vue-basic/blob/master/src/components/event.vue)

> 其实具体有哪些事件我在官网上都没有找到说明, 以上只是我在查找到[bpmn.io/diagram.js/modeling](https://github.com/bpmn-io/diagram-js/blob/master/lib/features/modeling/Modeling.js)文件之后, 取的一些我项目里有用到的事件.



### 监听element并绑定事件

上面介绍的是监听`modeler`并绑定事件, 可能你也需要监听用户点击图形上的`element`或者监听某个`element`改变:

- element.click 点击元素;
- element.changed 当元素发生改变的时候(包括新增、移动、删除元素)

继续在`success()`上添加监听事件:

```
// event.vue
<script>
...
success () {
	...
	this.addEventBusListener()
},
addEventBusListener () {
	let that = this
  const eventBus = this.bpmnModeler.get('eventBus') // 需要使用eventBus
  const eventTypes = ['element.click', 'element.changed'] // 需要监听的事件集合
  eventTypes.forEach(function(eventType) {
    eventBus.on(eventType, function(e) {
      console.log(e)
    })
  })
}
</script>
```

配置好`addEventBusListener()`函数后, 在进行元素的点击、新增、移动、删除的时候都能监听到了.

但是有一点很不好, 你在点击“画布”的时候, 也就是**根元素**也可能会触发此事件, 我们一般都不希望此时会触发, 因此我们可以在`on`回调中添加一些判断, 来避免掉不需要的情况:

```javascript
eventBus.on(eventType, function(e) {
  if (!e || e.element.type == 'bpmn:Process') return // 这里我的根元素是bpmn:Process
  console.log(e)
})
```

此时我们可以把监听到返回的节点信息打印出来看看:

![img2](https://user-gold-cdn.xitu.io/2019/12/10/16eeeb1269133514?w=2310&h=1814&f=jpeg&s=335137)



如上图, 它会打印出该节点的`Shape`信息和`DOM`信息等, 但我们可能只关注于`Shape`信息(也就是该节点的`id、type`等等信息), 此时我们可以使用`elementRegistry`来获取`Shape`信息:

```javascript
eventBus.on(eventType, function(e) {
  if (!e || e.element.type == 'bpmn:Process') return // 这里我的根元素是bpmn:Process
  console.log(e)
  var elementRegistry = this.bpmnModeler.get('elementRegistry')
  var shape = elementRegistry.get(e.element.id) // 传递id进去
  console.log(shape) // {Shape}
  console.log(e.element) // {Shape}
  console.log(JSON.stringify(shape)===JSON.stringify(e.element)) // true
})

```

或者你也可以直接就用`e.element`获取到`Shape`的信息, 我比较了一下它们两是一样的. 但是官方是推荐使用`elementRegistry`的方式.



### 通过监听事件判断操作方式

上面我们已经介绍了`modeler`和`element`的监听绑定方式, 在事件应用中, 你更多的需要知道用户要进行什么操作, 好写对应的业务逻辑. 

这里我就以我工作中要用到的场景为案例进行讲解.

- 新增了shape
- 新增了线(connection)
- 删除了shape和connection
- 移动了shape和线

```javascript
// event.vue
    ...
    success () {
      this.addModelerListener()
      this.addEventBusListener()
    },
    // 添加绑定事件
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
    addModelerListener() {
      // 监听 modeler
      const bpmnjs = this.bpmnModeler
      const that = this
      // 'shape.removed', 'connect.end', 'connect.move'
      const events = ['shape.added', 'shape.move.end', 'shape.removed']
      events.forEach(function(event) {
        that.bpmnModeler.on(event, e => {
          var elementRegistry = bpmnjs.get('elementRegistry')
          var shape = e.element ? elementRegistry.get(e.element.id) : e.shape
          // console.log(shape)
          if (event === 'shape.added') {
            console.log('新增了shape')
          } else if (event === 'shape.move.end') {
            console.log('移动了shape')
          } else if (event === 'shape.removed') {
            console.log('删除了shape')
          }
        })
      })
    },
    addEventBusListener() {
      // 监听 element
      let that = this
      const eventBus = this.bpmnModeler.get('eventBus')
      const eventTypes = ['element.click', 'element.changed']
      eventTypes.forEach(function(eventType) {
        eventBus.on(eventType, function(e) {
          if (!e || e.element.type == 'bpmn:Process') return
          if (eventType === 'element.changed') {
            that.elementChanged(eventType, e)
          } else if (eventType === 'element.click') {
            console.log('点击了element')
          }
        })
      })
    },
    elementChanged(eventType, e) {
      var shape = this.getShape(e.element.id)
      if (!shape) {
        // 若是shape为null则表示删除, 无论是shape还是connect删除都调用此处
        console.log('无效的shape')
        // 由于上面已经用 shape.removed 检测了shape的删除, 因此这里只判断是否是线
        if (this.isSequenceFlow(shape.type)) {
          console.log('删除了线')
        }
      }
      if (!this.isInvalid(shape.type)) {
        if (this.isSequenceFlow(shape.type)) {
          console.log('改变了线')
        }
      }
    },
    getShape(id) {
      var elementRegistry = this.bpmnModeler.get('elementRegistry')
      return elementRegistry.get(id)
    },
    isInvalid (param) { // 判断是否是无效的值
      return param === null || param === undefined || param === ''
    },
    isSequenceFlow (type) { // 判断是否是线
      return type === 'bpmn:SequenceFlow'
    }
```

案例Git地址: [LinDaiDai-bpmn.js案例event.vue](https://github.com/LinDaiDai/bpmn-vue-basic/blob/master/src/components/event.vue) 喜欢的小伙伴请给个`Star`🌟呀, 谢谢😊


### 后语

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注**霖呆呆(LinDaiDai)的公众号**, 选择 **其它** 菜单中的 **bpmn.js群** 即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

