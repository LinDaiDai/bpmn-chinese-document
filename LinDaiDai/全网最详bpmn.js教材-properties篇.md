## Properties篇

哈哈 来了 来了 它终于来了😂.

让大家久等的`Properties`篇🎉.

其实霖呆呆工作上用到的`bpmn.js`的内容也就只局限于之前写的文章了, 算是将实际用到的知识全盘脱出了... 那么就有人会好奇的问了...为什么连`Properties-panel`这样重要的功能都没有用到呢🤔️?

这其实和我们团队的用法有关:

最开始接触使用到`bpmn.js`是因为需要用它来绘制工作流实现决策引擎的这么一个功能. 而我们的做法是前端通过`bpmn.js`来绘制流程图, 图中的`Start`、`UserTask`、`BusinessRuleTask`等等我在这里都称之为节点. 每个节点都对应着`xml`文件中的标签, 传统的做法可能是将各个节点的属性都保存到标签上, 例如我这里有一个开始节点的`xml`标签:

```xml
<startEvent id="StartEvent_1y45yut" name="开始" roles="admin"></startEvent>
```

然后给这个节点增加上一个名为“`权限(roles)`”的自定义属性, 这个属性会保存在`xml`中, 并且导出这个文件的时候也会留在上面.

我们虽然每个节点也都会关联很多信息, 但是这些信息并不是直接保存在` xml` 标签里的. 而是每个节点都会有一个 `id` , 后台有一个表专门用于存放每个节点的附加信息, 所以每次点击节点的时候, 都通过这个`id`来调取后台存储的数据, 从而拿到节点对应的属性, 右边出现一个抽屉将这些属性信息显示在里面可以进行修改. 修改保存之后, 也是调用后台的接口来修改表里的信息. 所以主要的逻辑还是集中在后台上. 因此对于`xml`的操作还真不是太多, 自然的也就没用上`Properties-panel`了.

但是我的这种做法, 你也可以理解为右边出现的“抽屉” 就是我自定义的`Properties-panel`, 因为它确实也起到了与节点关联属性的作用.

OK, 言归正传啦, 虽然我工作中并没有用到它, 但是经过读者给出的意见以及自己对它的一些研究, 还是能用它做一些业务实现的, 希望在你认真看完之后能有所收获😁.

通过这一章节的阅读你可以学习到:

- 什么是`bpmn properties`🤔️?
- 如何读取`bpmn properties`🤔️?
- 如何修改`bpmn properties`🤔️?
- 使用`updateProperties`方法🤔️?

## `bpmn properties`属性介绍以及基本用法

### 1. 什么是`bpmn properties`🤔️?

让我们先来搞懂一下什么是`bpmn properties`🤔️?

我们在用`bpmn.js`画的每一个节点其实都被称之为`diagram element`(图表元素, 是不是很好理解😁)

而在`bpmn`文件中的每个`xml`标签称之为`BPMN element`.

将`diagram element`与`BPMN element`的一些属性关联起来靠的是一个叫做`businessObject`的属性. 从名称上理解你也可以知道它是一个对象(Object), 你可以在这个对象中添加上一些特殊的属性, 并且这些属性是可以直接插到`BPMN element`上的.

举个例子🌰:

我绘制了一个`StartEvent`节点, 它对应的:

- `diagram element`:

  ```javascript
  {
  	id: "StartEvent_1y45yut",
  	type: "bpmn:StartEvent",
  	businessObject: {
  		$type: "bpmn:StartEvent",
  		name: "开始"
  	}
  }
  ```

- `BPMN element`:

  ```xml
  <startEvent id="StartEvent_1y45yut" name="开始"></startEvent>
  ```

像这类属性就是`bpmn properties`, 你可以用它来实现你的业务需要.

### 2. 如何读取`bpmn properties`🤔️?

不知道大家是否还记得我在《事件篇》中用到的一段代码:

```javascript
addModelerListener () {
       // 监听 modeler
      const bpmnjs = this.bpmnModeler
      const that = this
      const events = ['shape.added', 'shape.move.end', 'shape.removed']
      events.forEach(function(event) {
        that.bpmnModeler.on(event, e => {
          var elementRegistry = bpmnjs.get('elementRegistry')
          var shape = e.element ? elementRegistry.get(e.element.id) : e.shape
          if (event === 'shape.added') {
            console.log('新增了shape')
          } else if (event === 'shape.move.end') {
            console.log('移动了shape')
          } else if (event === 'shape.removed') {
            console.log('删除了shape')
          }
        })
      })
    }
```

这个方法是放在 `将字符串转换成图显示出来` 之后, 用于给元素绑定事件.

其中就有用到`elementRegistry`.

所以如果是在`html`中, 你就可以用这种方式来获取`bpmn properties`:

```html
<body>
	<div id="canvas"></div>
<script>
	var bpmnJS = new BpmnJS({
  	container: '#canvas'
  });
  bpmnJS.importXML(xmlStr, function(err) {
  	if (!err) {
  		var elementRegistry = bpmnJs.get('elementRegistry')
      var startEventElement = elementRegistry.get('StartEvent_1y45yut'),
          startEvent = startEventElement.businessObject;
     	console.log(startEvent.name) // 开始
  	}
  }
</script>
```

而在一些`class`里, 比如`CustomRenderer.js`里, 你可以直接用`.`的方式就获取到了:

```javascript
export default class CustomRenderer extends BaseRenderer {
	drawShape (parentNode, element) {
		// element.businessObject
		// or 解构
		// const { businessObject } = element
	}
}
```

### 3. 如何修改`bpmn properties`🤔️?

你在`bpmn`文件中, 可能会看到这样一段代码:

```xml
<bpmn:sequenceFlow id="SequenceFlow_1" sourceRef="StartEvent_1" targetRef="Task_1" name="FOO &gt; BAR?">
      <bpmn:conditionExpression xsi:type="bpmn:tFormalExpression"><![CDATA[${foo > bar}]]></bpmn:conditionExpression>
    </bpmn:sequenceFlow>
```

里面的`xsi:type`、`sourceRef`这些属性是啥啊🤔️? 我怎么知道哪类标签有哪些属性🤔️?

你其实可以在官方给的这个`bpmn.json`中查找到:

[《meta-model descriptor》](https://github.com/bpmn-io/bpmn-moddle/blob/master/resources/bpmn/json/bpmn.json)

设置的话可以根据以下方法:

```javascript
var moddle = bpmnJS.get('moddle');

// 创建一个BPMN element , 并且载入到导出的xml里
var newCondition = moddle.create('bpmn:FormalExpression', {
  body: '${ value > 100 }'
});

// 写入属性, 但是不支持撤销 
sequenceFlow.conditionExpression = newCondition;
```

上面👆的这种方式是不支持撤销的, 如果你想要能够 撤销/重新 的话, 你可以通过以下这种方式:

```javascript
var modeling = bpmnJS.get('modeling');

modeling.updateProperties(sequenceFlowElement, {
  conditionExpression: newCondition
});
```

也就是通过`modeling.updateProperties()`这个方法.

这个`modeling`好像是需要引入的, 反正如果我是使用`DNS` 远程的引入下面的这个`js`好像就会报错`Uncaught Error: No provider for "modeling"!`.

```html
<script src="https://unpkg.com/bpmn-js@6.0.2/dist/bpmn-viewer.development.js"></script>
```

当然如果你是使用`npm` 下载的话就没有这个问题了.

这个方法的第一个参数是一个 `diagram element`, 也就是前面我们提到的用`elementRegistry`获取到的对象.

第二个参数是要修改的属性, 它是一个`Map`结构.

### 4. 使用`updateProperties`方法

例如🌰, 我想在点击某个类型为`bpmn:Task`的节点的时候, 修改它的`name`属性, 我可以这么做:

- 给节点添加点击事件
- 判断节点类型为`bpmn:Task` , 只对这种类型的节点做后续处理
- 使用`updateProperties`更新`name`

```javascript
    createNewDiagram () {
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
    success () {
        this.addModelerListener() // 添加监听事件
    },
    addModelerListener () {
    		const eventBus = this.bpmnModeler.get('eventBus')
      	const modeling = this.bpmnModeler.get('modeling')
      	const elementRegistry = this.bpmnModeler.get('elementRegistry');
      	const eventTypes = ['element.click', 'element.changed'];
      	eventTypes.forEach(function(eventType) {
          	eventBus.on(eventType, function (e) {
              	if (!e || !e.element) {
                  console.log('无效的e', e)
                  return
                }
              	if (eventType === 'element.click') {
                  console.log('点击了element', e)
                  var shape = e.element ? elementRegistry.get(e.element.id) : e.shape
                  if (shape.type === 'bpmn:Task') {
                    modeling.updateProperties(shape, {
                    	name: '我是修改后的Task名称'
                  	})
                  }
                }
            })
        })
    }
```

当然你也可以一次性修改多个属性:

```javascript
modeling.updateProperties(startEventElement, {
	name: '我是修改后的虚线节点',
	isInterrupting: false
})
```

我通过查找前面的 `meta-model descriptor` 知道`StartEvent`还有一个`isInterrupting`属性, 于是试着修改它, 结果竟然成功了, 将开始节点变为了虚线为边框的节点:

![bpmnModeler5.png](https://user-gold-cdn.xitu.io/2020/1/12/16f95581f47db29e?w=582&h=302&f=png&s=24462)


当然你也可以加一些除了`meta-model descriptor`里的一些自定义属性:

```javascript
modeling.updateProperties(shape, {
	name: '我是修改后的虚线节点',
	isInterrupting: false,
	customText: '我是自定义的text属性'
})
```

只不过, 它们会被放到`$attrs`中:

![bpmnModeler6.png](https://user-gold-cdn.xitu.io/2020/1/12/16f9558539e50e78?w=806&h=290&f=png&s=164051)


并且这种方式, 也是可以直接修改到`bpmn`文件中的:

![bpmnModeler7.png](https://user-gold-cdn.xitu.io/2020/1/12/16f9558856a612c9?w=1740&h=110&f=png&s=148534)


## 后语

这一章节主要是向大家介绍了一下`bpmn properties`的概念以及操作方式...其实在这之前, 我甚至不知道怎么给`xml`标签上添加属性, 也甚至不知道`updateProperties`怎么使用😂...

还好皮厚的我不耻下问, 在群里问了一些小伙伴...

哈哈哈, 手动艾特感谢 @网易-付超老哥 还有[@李岱老哥](https://juejin.im/user/57a1b33679bc4400549c05b7) , 在研究`bpmn.js`属性上面给我的帮助, 当然我也知道很多小伙伴希望我能快点更上一篇关于`properties-panel`的内容...

但今天真的有点累了... 容我先缓一缓, 咱明天再更行不😁.

(嗯...不行也得行, 我说了算...)

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注**霖呆呆(LinDaiDai)的公众号**, 选择 **其它** 菜单中的 **bpmn.js群** 即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

