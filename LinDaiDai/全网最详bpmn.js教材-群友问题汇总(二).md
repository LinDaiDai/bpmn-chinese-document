# 全网最详bpmn.js教材-群友问题汇总(二)

1. 如何获取所有的事件（超哥）
2. 把左侧工具栏上的四个功能分离出来(Ruby)
3. 画出来的图在activiti跑不起来，看了下xml才发现都是camunda(多人)
4. 设置初始化元素到正中心(WD)
5. 默认的bpmn.js，后端的`activity`解析不了（Ivan）
6. 自定义属性不能是default ？ 用default就报错(小笨蛋)
7. 如何给一些特殊的节点设置颜色（锦绣Erin）
8. bpmn底层是如何操作xml的, 源码在哪里（流星）
9. 条件顺序流的条件是怎么加进去的（Ivan）
10. 如何使得流程图不能被编辑，仅仅做展示用（Ivan和Mercury、锦绣）
11. 禁止滚动（kokoro、What If.）
12. 如何将自定义类型的元素通过基本类型渲染出来(好好干)
13. 开发环境正常，生产环境却会报错（锦绣Erin）
14. 自定义属性面板的监听添加(麦兜)
15. 如何获取流程图最外层的属性(根节点)





### 1. 如何获取所有的事件

(感谢该问题解决者超哥)

如果不了解事件的小伙伴请移步[《全网最详bpmn.js教材-事件篇》](https://juejin.im/post/5def47e16fb9a0160376e416)。

在官方文档中并没有很明确的地方介绍具体有哪些监听事件。

事实上，我们用的比较多的可能就是`shape.added(新增shape)`、`element.click(点击元素)`、`element.changed(元素发生改变)` 等等。

但如果想要知道在做某个操作时触发了什么事件该怎么办呢 🤔️？

如果你不想去啃源码的话，我们这里提供了一个办法，能勉强用用：

在`importXML`函数的回调中，获取到所有的可用事件，全部绑定到元素上，然后看你的操作触发了哪个。

来看看代码：

 ```javascript
createNewDiagram() {
	this.bpmnModeler.importXML(xmlStr, err => {
    if (!err) { // 渲染成功
      this.addEventBusListener()
    }
  })
}
addEventBusListener () { // 绑定监听事件
	const eventBus = this.bpmnModeler.get('eventBus');
	const eventTypes = Object.keys(eventBus._listeners); // 获取所有的可用事件
	console.log(eventTypes); // 打印出来有242种事件
	eventTypes.forEach((event) => {
		this.bpmnModeler.on(event, e => {
			console.log(event);
		})
	})
}
 ```

如图👇，打印出来的可用事件为`242`种，现在你可以在页面上去操作元素，然后查看你期望的事件了。

图片1



### 2. 把左侧工具栏上的四个功能分离出来

在[《全网最详bpmn.js教材-自定义palette篇》](https://juejin.im/post/5df197c4f265da33bd4976af)中我介绍的都是一些**如何使用palette创建元素**，但是如何使用`palette`中上面的那四个工具(官方上说它们属于`Activate`这个组)却没介绍。

先来简单介绍下这四个工具的作用：

##### (一) Activate the hand tool

工具栏第一个像手一样的那个工具，它的作用实际上就是能拖动整个画布，来进行位置调整(和单击长按画布效果一样)

##### (二) Active the lasso tool

点击之后，能选择多个元素进行集体拖拽：

![bpmn1](../resource/problem/bpmn1.gif)

##### (三) Active the create/remove space tool

点击之后，能调整一侧元素之间的间隔：

![bpmn2](../resource/problem/bpmn2.gif)

##### (四) Active the global connect tool

创建一根连接线：



![bpmn3](../resource/problem/bpmn3.gif)

把左侧工具栏上的四个功能分离出来

样式：用原先的class名称

功能：

![WechatIMG316](../resource/problem/WechatIMG316.png)

![WechatIMG319](../resource/problem/WechatIMG319.png)



### 3.

<img src="../resource/problem/0B01F220-CA9B-41EA-8AC9-84E113E3E390.png" alt="0B01F220-CA9B-41EA-8AC9-84E113E3E390" style="zoom:50%;" />



![WechatIMG312](../resource/problem/WechatIMG312.png)



![WechatIMG313](../resource/problem/WechatIMG313.png)



<img src="../resource/problem/AE96B155-9290-432A-84A1-957DC9D0DF8B.png" alt="AE96B155-9290-432A-84A1-957DC9D0DF8B" style="zoom:50%;" />



<img src="../resource/problem/46BDE64B-158E-4B32-ABC7-CACF51385D5F.png" alt="46BDE64B-158E-4B32-ABC7-CACF51385D5F" style="zoom:50%;" />



### 4.

设置初始化元素到正中心

```
bpmnViewer.get('canvas').zoom('fit-viewport', 'auto')
```



### 5.

默认的bpmn.js，后端的`activity`解析不了,主要是很多属性前面需要添加activity的namespace



<img src="../resource/problem/WeChate7d32848fc70b5c6a169bb0d6ed05ecf.png" alt="WeChate7d32848fc70b5c6a169bb0d6ed05ecf" style="zoom:50%;" />

<img src="../resource/problem/A256DA1C-E9B1-4EC8-90F7-6472B3D86509.png" alt="A256DA1C-E9B1-4EC8-90F7-6472B3D86509" style="zoom:50%;" />



<img src="../resource/problem/5A8C0AD2-4BD7-459F-8A05-F919B9ADB605.png" alt="5A8C0AD2-4BD7-459F-8A05-F919B9ADB605" style="zoom:50%;" />



![WechatIMG62](../resource/problem/WechatIMG62.png)



<img src="../resource/problem/WechatIMG64.png" alt="WechatIMG64" style="zoom:50%;" />



### 6.

自定义属性不能是default ？ 用default就报错

![57819D29-947C-4013-A1A9-FA66044ED11D](../resource/problem/57819D29-947C-4013-A1A9-FA66044ED11D.png)

### 7.

如何给一些特殊的节点设置颜色

![E5C86775-0DE7-4DFD-9BDD-98695C79C389](../resource/problem/E5C86775-0DE7-4DFD-9BDD-98695C79C389.png)



### 8.

bpmn底层是如何操作xml的, 源码在哪里

![WechatIMG206](../resource/problem/WechatIMG206.png)



<img src="../resource/problem/A8E7ECA0-EC4C-4DEB-BEAB-BB16C8FEA44F.png" alt="A8E7ECA0-EC4C-4DEB-BEAB-BB16C8FEA44F" style="zoom:50%;" />



### 9.

条件顺序流的条件是怎么加进去的

<img src="./../resource/problem/6BE53065-AA11-4446-B6B3-81B5CF6BB709.png" alt="6BE53065-AA11-4446-B6B3-81B5CF6BB709" style="zoom:50%;" />



![WechatIMG210](../resource/problem/WechatIMG210.png)



![WechatIMG214](../resource/problem/WechatIMG214.png)





### 10.

如何使得流程图不能被编辑，仅仅做展示用

1. 用 import BpmnViewer from 'bpmn-js' (Ivan)
2. 直接在上面蒙个遮罩层 (Mercury)
3. 锦绣提供的方法。（聊天记录2月21日）



### 11.

![WechatIMG855](../resource/problem/WechatIMG855.jpeg)



![WechatIMG70](../resource/problem/WechatIMG70.png)

### 12.

```javascript
drawShape(parentNode, element) {
    const types = {
        'wfe:CounterSignatureUserTask': 'bpmn:ScriptTask',
        'wfe:ConvergingParallelGateway': 'bpmn:ParallelGateway'
    }
    const type = types[element.type] || element.type
    const h = this.bpmnRenderer.handlers[type];

    return h(parentNode, element);
  }
```

使用了它底层的handlers，它预定义了基本的类型，我这里做了一个转换



### 13

![image-20200304111459558](../resource/problem/131.png)

$inject的问题

https://segmentfault.com/a/1190000020723731?utm_source=tag-newest



### 14

![WechatIMG2119](../resource/problem/333.png)



![WechatIMG2140](../resource/problem/4444.png)

![111](../resource/problem/111.png)

![222](../resource/problem/222.png)





### 15

```javascript
this.bpmnModeler.getDefinitions().rootElements[0]
```

