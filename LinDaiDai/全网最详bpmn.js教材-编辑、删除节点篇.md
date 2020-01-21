## 前言
> Q: bpmn.js是什么? 🤔️

[bpmn.js](https://bpmn.io/)是一个BPMN2.0渲染工具包和web建模器, 使得画流程图的功能在前端来完成.

> Q: 我为什么要写该系列的教材? 🤔️

因为公司业务的需要因而要在项目中使用到`bpmn.js`,但是由于`bpmn.js`的开发者是国外友人, 因此国内对这方面的教材很少, 也没有详细的文档. 所以很多使用方式很多坑都得自己去找.在将其琢磨完之后, 决定写一系列关于它的教材来帮助更多`bpmn.js`的使用者或者是期于找到一种好的绘制流程图的开发者. 同时也是自己对其的一种巩固.

由于是系列的文章, 所以更新的可能会比较频繁, **您要是无意间刷到了且不是您所需要的还请谅解**😊.

不求赞👍不求心❤️. 只希望能对你有一点小小的帮助.

## 编辑、删除节点篇

虽然前面已经说了很多关于如何创建, 渲染元素的知识, 但是在实际使用上肯定不仅仅只局限于创建`Task`、 `Event`这些节点上.

你可能还需要创建: **线(`bpmn:SequenceFlow`)、网关(`ExclusiveGateway`)、活动(`Activities`)** 等等其他类型的节点.

甚至你想要在`contextPad`中定义一个删除、编辑节点的功能.

那么这一章节我们主要就是来讲解这些.

通过阅读你可以学习到:

- [`contextPad`上的删除功能](`contextPad`上的删除功能)
- [`contextPad`上的编辑功能](`contextPad`上的编辑功能)

## `contextPad`上的删除功能

让我们接着上个章节的案例进行讲解哈, 项目还是之前的项目[LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom)

想要实现的功能是: 在`contextPad`中加上一个删除功能(这里加上一个小垃圾桶):

![bpmnCustom20.png](https://user-gold-cdn.xitu.io/2019/12/20/16f23b8dade787b3?w=890&h=426&f=png&s=84123)

并且点击它的时候可以删除当前的节点...

让我们打开`CustomContextPad.js`文件或者`CustomContextPadProvider.js`文件, 然后在`getContextPadEntries`方法中加上以下代码:

```javascript
// CustomContextPad.js
getContextPadEntries(element) {
    const { modeling } = this // modeling需要利用CustomContextPad.$inject注册进来
    function removeElement(e) { // 点击的时候实现删除功能
        modeling.removeElements([element])
    }
    function deleteElement() { // 创建垃圾桶
        return {
            group: 'edit',
            className: 'icon-custom icon-custom-delete',
            title: translate('删除'),
            action: {
                click: removeElement
            }
        }
    }
    return {
        'append.lindaidai-task': {...},
        'delete': deleteElement() // 返回值加上删除的功能
    }
}
```
可以看到要点就是:

- 将`modeling`引进来, 因为要使用到它的`removeElements`方法;
- 定义绘制垃圾桶的功能
- 编写`className`来实现修改默认样式的功能

OK👌, 接下来别忘了在我们的`app.css`中加上垃圾桶的样式:
```css
/* app.css*/
.icon-custom-delete {
    background-image: url('https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/delete.png');
}

.djs-context-pad .icon-custom-delete.entry:hover {
    background: url('https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/delete.png') center no-repeat!important;
    background-size: cover!important;
}
```

(自定义`modeler`中的`CustomContextPadProvider.js`也是这么写的)

这样删除功能就实现了.

## `contextPad`上的编辑功能

其实这里说的编辑功能, 是指在`contextPad`上定一个编辑的图标, 然后点击的时候, 可以出现一个弹窗, 或者右边出现一个自定义的`properties-panel`, 然后这里面可以显示出节点的一些信息.

这么做的原因是: 

- 你期望的可能不是点击节点的时候右边出现`properties-panel`, 而是将`properties-panel`作为一个抽屉隐藏在右侧, 点击`contextPad`中的某个图标才从右侧出来.
- 点击`contextPad`中的某个图标获取到当前节点的节点信息然后做其他自定义的操作.

### 通过点击小图标获取节点信息

![bpmnCustom21.png](https://user-gold-cdn.xitu.io/2019/12/20/16f23d7d59b56775?w=2268&h=1380&f=png&s=1060268)

如上图, 先实现这个功能: 点击编辑图标, 将节点信息打印出来.

其实也很简单, 经过了`lindaidi-task`和`delete`之后, 我相信你也掌握了一些规律了.

反正要创建什么图标就往`getContextPadEntries`的返回值加就可以了:

```javascript
// CustomContextPad.js
getContextPadEntries(element) {
    function clickElement(e) {
        console.log(element)
    }
    function editElement() { // 创建编辑图标
        return {
            group: 'edit',
            className: 'icon-custom icon-custom-edit',
            title: translate('编辑'),
            action: {
                click: clickElement
            }
        }
    }
    return {
        'append.lindaidai-task': {...},
        'edit': editElement(), // 返回值加上编辑功能
        'delete': deleteElement() // 返回值加上删除的功能
    }
}
```
然后记得在`app.css`中加上`.icon-custom-edit`的样式, 这里就不贴代码了.

### 将节点的信息传递出去

其实我们会发现, 通过点击小图标获取到节点信息很容易就实现了, 但是如何将在`CustomContextPad.js`中的信息传递出去呢, 也就是我们在页面上该怎么拿到这个信息呢?

比如我想实现: **点击上面👆所说的编辑小图标, 然后出现这么一个弹窗, 显示出节点的相关信息**:

![bpmnCustom22.png](https://user-gold-cdn.xitu.io/2019/12/21/16f2630fdaeaa223?w=1814&h=1550&f=png&s=260254)

(由于没有引入任何的`UI`组件, 所以随手写了一些样式)

哈哈😄, 方法其实也有很多种:

- 前端本地存储`localStorage`
-  `vue`的`vuex`
-  `react`的`redux`

以上技术都可以实现...

由于项目是用`vue`开发的, 所以这里我就演示一下利用`vuex`来进行交互😄.

首先在我们的项目里安装上`vuex`:
```
$ npm i vuex --save-D
```
然后在`src`目录下创建好一个`store`文件夹用来存放它, 并记得在`main.js`用进行引用:
```javascript
// main.js
import store from './store'
...
new Vue({
    ...
    store,
    render: h => h(App),
}).$mount('#app')
```
让我们在`store`中创建一个叫做`bpmn`模块, 专门用来定义`bpmn`相关的存储. 然后在其中定义:
1. `nodeInfo<Object>`:  用于存储当前点击的节点的信息
2. `nodeVisible<Boolean>`: 用于判断弹窗显示隐藏的变量

```javascript
// store/modules/bpmn.js
const bpmn = {
    state: {
        nodeVisible: false,
        nodeInfo: {}
    },
    mutations: {
        TOGGLENODEVISIBLE: (state, visible) => {
            state.nodeVisible = visible
        },
        SETNODEINFO: (state, info) => {
            state.nodeInfo = info
        }
    },
    actions: {}
}

export default bpmn
```
定义好这些之后, 我们就可以在`CustomContextPadProvider.js`里的`clickElement`做文章了:
```javascript
// CustomContextPadProvider.js
import store from '../../../store' // 引入store

function clickElement(e) {
    console.log(element)
    store.commit('SETNODEINFO', element) // 存储节点信息
    store.commit('TOGGLENODEVISIBLE', true)
}
```

**由于`CustomContextPadProvider.js`和`CustomContextPad.js`的做法都是一样的, 这里我就以`CustomContextPadProvider.js`为案例进行讲解.**

通过以上的步骤, 已经可以将这两个值存储到`store`中了, 接下来只是看看页面上该如何调用它们.

让我们打开`custom-modeler.vue`文件, 给里面加个小弹窗:
```html
<template>
    <div class="modal" v-if="bpmnNodeVisible" @click="close">
      <div class="modal-content">
        <div class="modal-ctx">
          <div class="modal-item">
            节点id: {{ bpmnNodeInfo.id }}
          </div>
          <div class="modal-item">
            节点type: {{ bpmnNodeInfo.type }}
          </div>
        </div>
      </div>
    </div>
</template>
```
弹窗样式随便写了点, 在项目代码里可以看到, 这里就不贴了.

然后编辑相关的`js`代码:
```html
<script>
import { mapState, mapMutations } from 'vuex'

export default {
    ... // 这个省略号是省略代码
    methods: { // 方法
        ...mapMutations(['TOGGLENODEVISIBLE']), // 这个省略号是解构
        close () {
            this.TOGGLENODEVISIBLE(false)
        }
    },
    computed: { // 计算属性
        ...mapState({ // 解构
            bpmnNodeInfo: state => state.bpmn.nodeInfo,
            bpmnNodeVisible: state => state.bpmn.nodeVisible
        })
    }
}
</script>
```

完成了上面的步骤之后, 我们就实现了点击`contextPad`中的编辑图标, 出现显示节点相关信息的小弹窗, 点击阴影出关闭小弹窗的功能了, 当然了你也可以在关闭的时候, 清空掉`store`中的节点信息`nodeInfo`, 这里就不做这些操作了.

最后让我们来梳理一下, 前面的关键步骤:
- 引用`vuex`来实现跨组件传递数据;
- 在点击编辑小图标的时候将节点信息存储到`store`中;
- 页面要使用的时候, 利用`vue`中计算属性能够监听`state`的改变的原理来更新你的`UI`(也就是出现弹窗)

(我开始是想用最简单的`localStorage`来实现的, 后来发现`computed`不能够监听到它的改变, 导致`localStorage`中的`nodeVisible`虽然已经变化了, 但是`bpmnNodeVisible`还是没有. 因此后来转用了`vuex`)

## 后语

其实这一章节主要是给大家传递一种思路, 如何将`contextPad`与你的页面结合起来, 讲解中只是说了一种最简单的出现小弹窗的情况, 可能在实际开发中你会有更多复杂的需求, 复杂的交互. 

不过也很高兴能给你提供一个这样的解决方案, 也可以说给你一点灵感吧😊...

因为自己在研究`bpmn.js`的时候, 也是没有任何人指导, 全靠自己查看官方案例还有绞尽脑汁的想, 所以我才明白这玩意的麻烦... 哈哈哈😂, 扯多了, 这一章节就到这里吧.

上面👆案例用的都是同一个项目🦐

项目案例Git地址: [LinDaiDai/bpmn-vue-custom](https://github.com/LinDaiDai/bpmn-vue-custom) 喜欢的小伙伴请给个`Star`🌟呀, 谢谢😊

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注**霖呆呆(LinDaiDai)的公众号**, 选择 **其它** 菜单中的 **bpmn.js群** 即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

