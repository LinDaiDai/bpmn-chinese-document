## 前言
> Q: bpmn.js是什么? 🤔️

[bpmn.js](https://bpmn.io/)是一个BPMN2.0渲染工具包和web建模器, 使得画流程图的功能在前端来完成.

> Q: 我为什么要写该系列的教材? 🤔️

因为公司业务的需要因而要在项目中使用到`bpmn.js`,但是由于`bpmn.js`的开发者是国外友人, 因此国内对这方面的教材很少, 也没有详细的文档. 所以很多使用方式很多坑都得自己去找.在将其琢磨完之后, 决定写一系列关于它的教材来帮助更多`bpmn.js`的使用者或者是期于找到一种好的绘制流程图的开发者. 同时也是自己对其的一种巩固.

由于是系列的文章, 所以更新的可能会比较频繁, **您要是无意间刷到了且不是您所需要的还请谅解**😊.

不求赞👍不求心❤️. 只希望能对你有一点小小的帮助.

## 自定义ContextPad篇

经过前面几章的学习, 相信大家都已经掌握了自定义`palette`和`renderer`, 这一章节主要讲解的是自定义`contextPad`.

先让我们来回顾一下, `contextPad`是什么?


![](https://user-gold-cdn.xitu.io/2019/12/19/16f1e913370c96fe?w=2028&h=1560&f=png&s=860707)

如图, 可以看到除了在左侧的工具栏处能添加节点之外, 点击节点的时候也会出现一个小弹窗, 这里面也可以添加节点. 这个小弹窗就是 **`contextPad`**.

那么, 通过阅读你可以学习到:

- [在默认的ContextPad基础上修改](#在默认的ContextPad基础上修改)
- [完全自定义ContextPad](#完全自定义ContextPad)


## 在默认的`ContextPad`基础上修改

### 前期准备

让我们接着在[LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom)案例项目上进行开发.

在`components`文件夹下新建一个`custom-context-pad.vue`文件, 同时配置路由“自定义contextPad”.

在`components/custom`文件夹下新建一个`CustomContextPad.vue`文件, 用来自定义`contextPad`.

### 编写`CustomContextPad.vue`代码

其实自定义`contextPad`和`palette`很像, 只不过是使用`contextPad.registerProvider(this)`来指定它是一个`contextPad`, 而自定义`palette`是用`platette.registerProvider(this)`. 

代码如下:

```javascript
// CustomContextPad.js
export default class CustomContextPad {
    constructor(config, contextPad, create, elementFactory, injector, translate) {
        this.create = create;
        this.elementFactory = elementFactory;
        this.translate = translate;

        if (config.autoPlace !== false) {
            this.autoPlace = injector.get('autoPlace', false);
        }

        contextPad.registerProvider(this); // 定义这是一个contextPad
    }

    getContextPadEntries(element) {}
}

CustomContextPad.$inject = [
    'config',
    'contextPad',
    'create',
    'elementFactory',
    'injector',
    'translate'
];
```
相信大家都已经看出来了, 重点还是在于`getContextPadEntries`这个方法, 接下来让我们来构建这个方法.

### 编写`getContextPadEntries`代码

其实这个方法, 需要返回的也是一个对象, 也就是你要在`contextPad`这个容器里显示哪些自定义的元素, 比如我这里需要给容器里添加一个`lindaidai-task`的元素, 那么我们可以在返回的对象中添加上`append.lindaidai-task`这个属性.

而属性值就是这个元素的一系列配置, 和`palette`中一样, 包括:

- group: 属于哪个分组, 比如`tools、event、gateway、activity`等等,用于分类
- className: 样式类名, 我们可以通过它给元素修改样式
- title: 鼠标移动到元素上面给出的提示信息
- action: 用户操作时会触发的事件


大概是这样:
```javascript
// CustomContextPad.js
getContextPadEntries(element) {
    return {
        'append.lindaidai-task': {
            group: 'model',
            className: 'icon-custom lindaidai-task',
            title: translate('创建一个类型为lindaidai-task的任务节点'),
            action: {
                click: appendTask,
                dragstart: appendTaskStart
            }
        }
    };
}
```

接下来就是构建`appendTask`和`appendTaskStart`
```javascript
// CustomContextPad.js
getContextPadEntries(element) {
        const {
            autoPlace,
            create,
            elementFactory,
            translate
        } = this;

        function appendTask(event, element) {
            if (autoPlace) {
                const shape = elementFactory.createShape({ type: 'bpmn:Task' });
                autoPlace.append(element, shape);
            } else {
                appendTaskStart(event, element);
            }
        }

        function appendTaskStart(event) {
            const shape = elementFactory.createShape({ type: 'bpmn:Task' });
            create.start(event, shape, element);
        }

        return {
            'append.lindaidai-task': {
                group: 'model',
                className: 'icon-custom lindaidai-task',
                title: translate('创建一个类型为lindaidai-task的任务节点'),
                action: {
                    click: appendTask,
                    dragstart: appendTaskStart
                }
            }
        };
    }
}
```

这里和`palette`中有一点不同, 就是多了一层`autoPlace`的判断, 其实我也没太搞明白这个`autoPlace`的作用是什么, 自动放置? 而且官方给的例子🌰就是这么写的, 有知道的小伙伴还请评论区留言哦, 谢谢~

### 修改`contextPad`的相关样式

此时我们看看效果吧😄...

![bpmnCustom16.png](https://user-gold-cdn.xitu.io/2019/12/19/16f1edb5fbf166e2?w=680&h=398&f=png&s=80316)

咿~ 好像可以耶, 但是, 这个小积木也太小了一点吧😅, 而且鼠标移上去之后, 黄色的背景色直接就覆盖它了...

![bpmnCustom17.png](https://user-gold-cdn.xitu.io/2019/12/19/16f1edceed269d46?w=700&h=440&f=png&s=81109)

哇, 这不能忍啊...

得想法子解决它, 还我漂亮的小积木❤️...

接着让我们打开控制台审查元素, 可以发现这个背景色是一个叫`.djs-context-pad .entry`的类提供的样式, 也许, 我们可以全局修改这个样式, 让我们试试看:
```css
/* app.css */

/* 自定义 contextPad 的样式 */
.djs-context-pad .lindaidai-task.entry:hover {
    background: url('https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/rules.png') center no-repeat!important;
    background-size: cover!important;
}

.djs-context-pad .entry:hover { /* 重新修改了 hover 之后的样式 */
    border: 1px solid #1890ff;
}

.djs-context-pad .entry {
    box-sizing: border-box;
    background-size: 94%;
    transition: all 0.3s;
}
```

打开页面看看效果哈.

![bpmnCustom18.png](https://user-gold-cdn.xitu.io/2019/12/19/16f1ee2a5bae159d?w=1020&h=434&f=png&s=112037)

不错, 解决了, 哈哈😄.

**(直接修改样式虽然不是最好的解决办法, 但这是目前我能想到的办法, 而且它确实也能够解决问题)**

## 完全自定义`ContextPad`

### 前期准备

同样的, 如果你已经学会了**在默认的`ContextPad`基础上修改**, 那么完全自定义`ContextPad`也就差不多了😁.

不过完全自定义`ContextPad`不是叫`contextPad`, 而是`contextPadProvider`, 好像要更厉害一点🤭...

OK👌, 让我们在`customModeler/custom`文件夹下新建一个`CustomContextPadProvider.js`文件.

### 编写`CustomContextPadProvider.js`代码

先让我们来看下主要的结构:

```javascript
// CustomContextPadProvider.js
export default function ContextPadProvider(contextPad, config, injector, translate, bpmnFactory, elementFactory, create, modeling, connect) {
  this.create = create
  this.elementFactory = elementFactory
  this.translate = translate
  this.bpmnFactory = bpmnFactory
  this.modeling = modeling
  this.connect = connect
  config = config || {}
  if (config.autoPlace !== false) {
      this._autoPlace = injector.get('autoPlace', false)
  }
  contextPad.registerProvider(this)
}

ContextPadProvider.$inject = [
  'contextPad',
  'config',
  'injector',
  'translate',
  'bpmnFactory',
  'elementFactory',
  'create',
  'modeling',
  'connect'
]

ContextPadProvider.prototype.getContextPadEntries = function(element) {}
```

别看上面代码很长的样子, 其实没啥东西:

- 定义一个`ContextPadProvider`类, 然后引入一些我们后面要用到的方法或者属性
- 通过`$inject`注入进来
- 重写原型链上的`getContextPadEntries`方法

### 编写`getContextPadEntries`代码

你应该也发现了, 重点还是重写`getContextPadEntries`这个方法.

额, 这里我先以一个简单的为例, 先只是创建一个`lindaidai-task`. 因此可以直接把[在默认的ContextPad基础上修改](#在默认的ContextPad基础上修改)案例中的`getContextPadEntries`中的代码复制过来:

```javascript
// CustomContextPad.js
getContextPadEntries(element) {
        const {
            autoPlace,
            create,
            elementFactory,
            translate
        } = this;

        function appendTask(event, element) {
            if (autoPlace) {
                const shape = elementFactory.createShape({ type: 'bpmn:Task' });
                autoPlace.append(element, shape);
            } else {
                appendTaskStart(event, element);
            }
        }

        function appendTaskStart(event) {
            const shape = elementFactory.createShape({ type: 'bpmn:Task' });
            create.start(event, shape, element);
        }

        return {
            'append.lindaidai-task': {
                group: 'model',
                className: 'icon-custom lindaidai-task',
                title: translate('创建一个类型为lindaidai-task的任务节点'),
                action: {
                    click: appendTask,
                    dragstart: appendTaskStart
                }
            }
        };
    }
}
```
此时让我们先看看效果哈:

![bpmnCustom19](https://user-gold-cdn.xitu.io/2019/12/20/16f238a5c6f86598?w=946&h=460&f=png&s=32554)

效果好像是实现了, 而且点击和拖拽它也能实现创建`lindaidai-task`的效果...

但总感觉好像少了什么, 因为光创建`task`类型但元素是不够的呀, 可不可以创建线或者实现编辑, 删除元素的功能呢? 当然可以啦, 哈哈😄.

不过这一章节先说这么多, 如何创建线和实现编辑, 删除我会把它放到一下章来细讲哈😊, 不用着急.

## 后语

上面👆案例用的都是同一个项目🦐

项目案例Git地址: [LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom) 喜欢的小伙伴请给个`Star`🌟呀, 谢谢😊

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注**霖呆呆(LinDaiDai)的公众号**, 选择 **其它** 菜单中的 **bpmn.js群** 即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

