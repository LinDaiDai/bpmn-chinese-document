## Properties-panel篇

大家在了解了前一篇`properties`的内容后, 应该对属性有了一个大概的认识吧.

这一章节让我们来说说`properties-panel` 😄...

其实在前面的[《全网最详bpmn.js教材-基础篇》](https://juejin.im/post/5def4377e51d4557f852baf9)中已经提到了怎样使用`properties-panel`, 不过那里只是简单的教了大家如何引用而没有细说, 现在就让我来详细为大家讲解一下它具体的使用方法。

通过这一章节的阅读你可以学习到:

- `Properties-panel`的基本使用
- 扩展使用`Properties-panel`

## Properties-panel的基本使用

`properties-panel`本质上是`bpmn.js`的一个扩展, 它实现了BPMN 2.0建模器，使你可以通过属性面板编辑与执行相关的属性。

官方的一个截图:


![](https://user-gold-cdn.xitu.io/2020/1/13/16f9f36651f7cec7?w=2812&h=1444&f=png&s=525520)

### 1. 安装`properties-panel`

在之前的文章中有很多内容没有介绍清楚, 在这一章中我会仔细的介绍.

首先是安装上.

如果你想要使用它的话, 得自己安装一下:

```javascript
$ npm install --save bpmn-js-properties-panel
```

同样的记得在项目中引入样式:

```javascript
import 'bpmn-js-properties-panel/dist/assets/bpmn-js-properties-panel.css' // 右边工具栏样式
```

使用上, 得在`html`代码中提供一个标签作为盛放它的容器:

```html
<div id="js-properties-panel" class="panel"></div>
```

之后, 在构建`BpmnModeler`的时候添加上它:

```javascript
 // 这里引入的是右侧属性栏这个框
import propertiesPanelModule from 'bpmn-js-properties-panel'
// 而这个引入的是右侧属性栏里的内容
import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda'

const bpmnModeler = new BpmnModeler({
	//添加控制板
  propertiesPanel: {
        parent: '#js-properties-panel'
  },
  additionalModules: [
  	propertiesPanelModule,
  	propertiesProviderModule
  ]
})
```
在之前的文章中我没有弄清楚`propertiesPanelModule`和`propertiesProviderModule`的作用, 导致将左侧工具栏和右侧属性的引用方式写错了, 现在已经在[<全网最详bpmn.js教材-基础篇>](https://juejin.im/post/5def4377e51d4557f852baf9)中更正了...抱歉...


### 2. 安装`camunda-bpmn-moddle`

还有一点, 如果你想使用[Camunda BPM](https://camunda.org/)来执行相关属性的话, 也得安装一个叫`camunda-bpmn-moddle`的扩展:

```javascript
$ npm install --save camunda-bpmn-moddle
```
将其添加到项目中:

```javascript
 // 右侧属性栏
import propertiesPanelModule from 'bpmn-js-properties-panel'
import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda'
 // 一个描述的json
import camundaModdleDescriptor from 'camunda-bpmn-moddle/resources/camunda'

const bpmnModeler = new BpmnModeler({
	//添加控制板
  propertiesPanel: {
  	parent: '#js-properties-panel'
  },
  additionalModules: [
  	propertiesPanelModule,
  	propertiesProviderModule
  ],
  moddleExtensions: {
    //如果要在属性面板中维护camunda：XXX属性，则需要此 
    camunda: camundaModdleDescriptor
  }
})
```

(`Camunda BPM`是一个用于工作流执行引擎和工作流自动化的解决方案, 在这里就不展开说了)

而`camunda-bpmn-moddle`的作用就是告诉使用者`camunda:XXX`扩展属性。

说了这个咱也听不懂啊, 来说点具体的吧, 比如你已经安装并已经在项目上使用了`properties-panel`之后, 打开页面, 随便选择一个节点(就拿开始节点来说吧), 会出现四个选项卡(tab)能让你修改属性, 如果你没有安装`camunda`并引用`camundaModdleDescriptor`的话, 使用后面三个功能, 控制台就会报错了:

![](https://user-gold-cdn.xitu.io/2020/1/12/16f9940bc30a54b1?w=1775&h=823&f=png&s=209555)

它会告诉你`unknown type <camunda:FormData>`...

因为其实你查看`camunda-bpmn-moddle/resources/camunda`的源码就会发现, 这其实就是一个`json`文件, 里面存放的就是对各个属性的描述. 我们在后面自定义`properties-panel`的时候也会需要编写这样的一个`json`文件, 待会你就知道了.


### 3. 实际使用效果

OK...让我们来实际使用看看它们有什么效果.

为了方便查看, 我给`bpmnModeler`绑定一个`commandStack.changed`事件, 在图形每次改变的时候将最新的`xml`打印出来.

(关于事件绑定的部分可以看[<全网最详bpmn.js教材-事件篇>](https://juejin.im/post/5def47e16fb9a0160376e416))

之后还是点击开始节点, 并修改一些属性. 结果发现你修改的属性竟然同步更新到了`xml`上面:

![](https://user-gold-cdn.xitu.io/2020/1/12/16f99573b5b0c3bf?w=1903&h=923&f=png&s=294784)

Good Body! 你是不是想到了什么?!

没错! 和上一篇文章的`updateProperties`方法是不是很像呢, 都是能够更新属性到`xml`上.



## 扩展使用`Properties-panel`

与`palette`, `contextPad`等自定义方式一样, `Properties-panel`也可以在默认的基础上进行修改, 它允许你加上一些自定义的属性.

不过官方把它叫做`Properties Panel Extension`, 好像更专业一些...不过无所谓了, 你知道是那个意思就行了.

官方这里也提供了一个例子: [properties-panel-extension](https://github.com/bpmn-io/bpmn-js-examples/tree/master/properties-panel-extension)

我其实也是跟着官方的这个例子来探索它是怎么使用的.

首先让我们来明确一点, 还记得我们在使用原版`properties-panel`的时候, 引入了两个东西吗?
```javascript
// 这里引入的是右侧属性栏这个框
import propertiesPanelModule from 'bpmn-js-properties-panel'
// 而这个引入的是右侧属性栏里的内容
import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda'

additionalModules: [
  propertiesPanelModule,
  propertiesProviderModule
]
```
我研究了一下, 如果你不引入第一个只引入第二个的话, 属性栏就出不来了.

而如果你只引入第一个不引入第二个的话, 就会报错...

我理解一下大概是这样意思:
- 第一个`propertiesPanelModule `表示的是属性栏这个框, 就是告诉别人这里要有个属性栏;
- 第二个`propertiesProviderModule`表示的是属性栏里的内容, 也就是点击不同的`element`该显示什么内容.

看到这, 你是不是有了点思路呢? 嘻嘻😁...

既然这样的话, 我们只需要重写`propertiesProviderModule`就可以了, 不要引入官方提供的(也就是从`bpmn-js-properties-panel/lib/provider/camunda`引入的), 而是自定义一个`propertiesProviderModule`来显示自己想要的内容.

### 1. 前期准备

`properties-panel`的内容可能有点多, 我就另外创建了一个项目来做案例分析.

项目还是用`vue`来编写, 不过其实你只要有点基础都能看得懂.

先让我们来看看要实现的效果:


![](https://user-gold-cdn.xitu.io/2020/1/13/16f9a95c94eb370b?w=1880&h=905&f=png&s=227801)

- 点击开始节点的时候, 右侧的属性栏中有`General`和权限两个选项卡(tab);
- 权限这个选项卡中有一个组, 名为 编辑权限;
- 编辑权限下会有一个属性, 名为 标题, 它是一个输入框;
- 修改该开始节点的信息, 能将属性关联到`xml`中

让我们在`components`文件夹下创建一个`properties-panel-extension`文件夹, 这里用来放我们要自定义的属性内容.

然后在`properties-panel-extension`下再新建一个`descriptors`和`provider`文件夹.

- descriptors是用来放一些描述的`json`文件
- provider放你要自定义的选项卡(tab)

由于`General`是它原本就有的一个选项卡, 所以我们可以不用管它, 现在我们想要自定义的是一个名叫`“权限”`的选项卡, 所以我在`provider`文件夹下又创建了一个`authority`文件夹, 里面用来放我们选项卡的内容...

之后一顿操作, 让目录变成了这样:

![](https://user-gold-cdn.xitu.io/2020/1/13/16f9f20815eb2c95?w=484&h=294&f=png&s=25764)


`AuthorityPropertiesProvider.js`这个文件就是来编写`权限`这个选项卡的, 它是我们需要编写的主要文件.

`parts`这个文件夹就是来放各个组下的子元素, 比如这里的`“标题”`, 我给它取名为`TitleProps`.


### 2. provider返回值介绍

如果你看到上面那么多的文件感觉眼花缭乱的话, 请不必慌张😂, 这是正常的反应...

所以为了后面更好的讲解, 我决定先来介绍一下`provider`的返回值与页面的结构是如何对应上的.


![](https://user-gold-cdn.xitu.io/2020/1/13/16f9f479fa6232e6?w=1148&h=1190&f=png&s=115787)

通过上面👆的图, 我们可以看出来:

1. 每个`provider`下都会有一个`tabs`数组(一个`tab`就是一个选项卡)
2. 每个`tab`下都会有一个`groups`数组(一个`group`就是一个组)
3. 每个`group`下都会有一堆`props`, 它们可能是输入框, 也可能是下拉框

OK...现在是不是好理解多了😄...

所以我们只需要在`AuthorityPropertiesProvider.js`中返回一个这样的结构就可以了:

```javascript
/*-选项卡
  |
  -组
   |
   -属性*/
return [
    { // 选项卡
        id: 'general',
        groups: [] // 组
    },
    { // 选项卡
        id: 'authority',
        groups: [
            { // 组
                id: 'edit-authority', // 组id
                entries: [
                    { // 单个props
                        id: 'title',
                        description : '权限的标题',
                        label : '标题',
                        modelProperty : 'title'
                    }
                ]
            }
        ]
    }
]
```

### 3. 编写`AuthorityPropertiesProvider.js`代码

编写的顺序我打算从上往下一层一层的讲.

所以先来看看`AuthorityPropertiesProvider.js`总体是要返回什么.

```javascript
// AuthorityPropertiesProvider.js

import inherits from 'inherits';
// 引入自带的PropertiesActivator,  因为我们要用到它来处理eventBus
import PropertiesActivator from 'bpmn-js-properties-panel/lib/PropertiesActivator';

export default function AuthorityPropertiesProvider(
    eventBus, bpmnFactory, canvas, // 这里是要用到什么就引入什么
    elementRegistry, translate
) {
    PropertiesActivator.call(this, eventBus);
    
    this.getTabs = function (element) {
        var generalTab = {};
        var authorityTab = {};
        return [
            generalTab,
            authorityTab
        ];
    }
}

inherits(AuthorityPropertiesProvider, PropertiesActivator);
```

这样看, 结构是不是也很清晰呢? 😊

我们其实就是要重写里面的`getTabs`方法, 返回我们需要的`tab`.

每个`tab`都有固定的属性:

```javascript
var authorityTab = {
        id: 'authority',
        label: '权限',
        groups: createAuthorityTabGroups(element)
    };
```
你必须得准守以上命名规则来写哈😣...


#### 编写`createAuthorityTabGroups`函数代码

在确定了`tab`之后, 我们需要告诉它里面有哪些组, 这时候就可以创建一个`createAuthorityTabGroups`函数来返回想要的组.

```javascript
// AuthorityPropertiesProvider.js

import TitleProps from './parts/TitleProps';

function createAuthorityTabGroups(element) {
    var editAuthorityGroup = {
        id: 'edit-authority',
        label: '编辑权限',
        entries: [] // 属性集合
    }
    // 每个属性都有自己的props方法
    TitleProps(editAuthorityGroup, element);
    // OtherProps1(editAuthorityGroup, element);
    // OtherProps2(editAuthorityGroup, element);
    
    return [
        editAuthorityGroup
    ];
}
```

比如上面👆我就返回了一个`编辑权限`的组.

而各个属性是放到组的`entries`字段下的...

咿呀, 这里怎么没看到给`entries`数组添加属性呢?

但是下面好像有一个`TitleProps`呀, 这个是干嘛的🤔?

看着有点像用来添加属性的...

#### 编写`TitleProps.js`代码

是的, 由于属性可能会被多处用到, 所以我将它单独提了出来, 放到了`parts`这个文件夹下, 后面就可以往里面不停的加属性了.

这个属性的方法有点特别, 它接收两个参数:

- 一个组
- 当前`element`

因为同一个属性可能存在于不同的组里, 所以可以传入一个组.

另外可能要通过元素的类型来做各种判断, 所以可以传入当前元素.

```javascript
// /parts/TitleProps.js
import entryFactory from 'bpmn-js-properties-panel/lib/factory/EntryFactory';

import { is } from 'bpmn-js/lib/util/ModelUtil';

export default function(group, element) {
  if (is(element, 'bpmn:StartEvent')) { // 可以在这里做类型判断
    group.entries.push(entryFactory.textField({
      id : 'title',
      description : '权限的标题',
      label : '标题',
      modelProperty : 'title'
    }));
  }
}
```
啊😺, 原来`entries`是在每一个`Props`里添加属性的啊🙈...

在`push`方法里, 你得告诉它是要添加一个什么类型的`Props`.

主要就是通过`entryFactory`, 例如这里就是返回一个`text`类型的输入框.

有时候你想要的不仅仅是输入框怎么办🙈?

没关系, `entryFactory`本身为你提供了很多类型.

`Ctrl + 左键`查看`entryFactory`的源码, 你可以发现有很多类型:


![](https://user-gold-cdn.xitu.io/2020/1/13/16f9f7af40de3956?w=1640&h=1234&f=png&s=351389)

OK...

至此, 我们的自定义`authorityTab`权限选项卡就写完了 😊!

你如果想添加其它的选项卡用上面👆的方式就可以了...

#### 编写`generalTab`代码

上面👆的`权限`选项卡是我们自定义的一些内容, 如果你想要使用官方提供的一些`tab`和属性怎么办呢?

`generalTab`就为你演示了该如何做...

首先同样的, `generalTab`需要长成这样:

```javascript
var generalTab = {
    id: 'general',
    label: 'General',
    groups: createGeneralTabGroups(element, bpmnFactory, canvas, elementRegistry, translate)
};
```

我们看到`createGeneralTabGroups`好像传递了很多参数进去, 那是因为我们要在里面用到它们, 而这些参数在构造`AuthorityPropertiesProvider`函数的时候就引入进来的...

来看看`createGeneralTabGroups`是如何编写的:

```javascript
// AuthorityPropertiesProvider.js

import idProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/IdProps';
import nameProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/NameProps';
import processProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/ProcessProps';
import linkProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/LinkProps';
import eventProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/EventProps';
import documentationProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/DocumentationProps';

function createGeneralTabGroups(element, bpmnFactory, canvas, elementRegistry, translate) {

    var generalGroup = {
        id: 'general',
        label: 'General',
        entries: []
    };
    idProps(generalGroup, element, translate);
    nameProps(generalGroup, element, bpmnFactory, canvas, translate);
    processProps(generalGroup, element, translate);

    var detailsGroup = {
        id: 'details',
        label: 'Details',
        entries: []
    };
    linkProps(detailsGroup, element, translate);
    eventProps(detailsGroup, element, bpmnFactory, elementRegistry, translate);

    var documentationGroup = {
        id: 'documentation',
        label: 'Documentation',
        entries: []
    };

    documentationProps(documentationGroup, element, bpmnFactory, translate);

    return [
        generalGroup,
        detailsGroup,
        documentationGroup
    ];
}
```

在`general`中, 导出了三个组, 而每个组中的`Props`都是`bpmn-js-properties-panel/lib/provider/bpmn/parts`这个文件夹中拿的...

同样的, 你查找它的源码, 也能发现很多其它的`Props`, 你需要什么, 直接取来用就可以了[狗头].

#### `AuthorityPropertiesProvider.js`完整代码

额, 要不还是贴下完整的代码?

其实也不多, `91`行:

```javascript
import inherits from 'inherits';

import PropertiesActivator from 'bpmn-js-properties-panel/lib/PropertiesActivator';


import idProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/IdProps';
import nameProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/NameProps';
import processProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/ProcessProps';
import linkProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/LinkProps';
import eventProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/EventProps';
import documentationProps from 'bpmn-js-properties-panel/lib/provider/bpmn/parts/DocumentationProps';

import TitleProps from './parts/TitleProps';

function createGeneralTabGroups(element, bpmnFactory, canvas, elementRegistry, translate) {

    var generalGroup = {
        id: 'general',
        label: 'General',
        entries: []
    };
    idProps(generalGroup, element, translate);
    nameProps(generalGroup, element, bpmnFactory, canvas, translate);
    processProps(generalGroup, element, translate);

    var detailsGroup = {
        id: 'details',
        label: 'Details',
        entries: []
    };
    linkProps(detailsGroup, element, translate);
    eventProps(detailsGroup, element, bpmnFactory, elementRegistry, translate);

    var documentationGroup = {
        id: 'documentation',
        label: 'Documentation',
        entries: []
    };

    documentationProps(documentationGroup, element, bpmnFactory, translate);

    return [
        generalGroup,
        detailsGroup,
        documentationGroup
    ];
}

function createAuthorityTabGroups(element) {
    var editAuthorityGroup = {
        id: 'edit-authority',
        label: '编辑权限',
        entries: []
    }

    // 每个属性都有自己的props方法
    TitleProps(editAuthorityGroup, element);
    // OtherProps1(editAuthorityGroup, element);
    // OtherProps2(editAuthorityGroup, element);

    return [
        editAuthorityGroup
    ];
}

export default function AuthorityPropertiesProvider(
    eventBus, bpmnFactory, canvas, // 这里是要用到什么就引入什么
    elementRegistry, translate
) {
    PropertiesActivator.call(this, eventBus);

    this.getTabs = function(element) {
        var generalTab = {
            id: 'general',
            label: 'General',
            groups: createGeneralTabGroups(element, bpmnFactory, canvas, elementRegistry, translate)
        };

        var authorityTab = {
            id: 'authority',
            label: '权限',
            groups: createAuthorityTabGroups(element)
        };
        return [
            generalTab,
            authorityTab
        ];
    }
}

inherits(AuthorityPropertiesProvider, PropertiesActivator);

```

经过我们的拆分, 感觉异常简单有木有 😊 !

(没错, 霖呆呆就是这么一个简单善良如白纸一般的男子😳...)

### 4. 编写`authority.json`代码

OK...其实到了这里就接近尾声了, 但是其实还有非常关键的一步要做...

刚刚我们自定义了一个叫做`权限`的选项卡, 还有一个叫`title`的属性, 并且还指定了只有`StartEvent`中出现, 那么此时我们还得在一个叫`authority.json`的文件中做一些说明.

(之所以取名为`authority.json`, 是因为我添加的选项卡叫`权限`, 这个命名随便你自己)

它长成这样:

```json
{
    "name": "Authority",
    "prefix": "authority",
    "uri": "http://authority",
    "xml": {
      "tagAlias": "lowerCase"
    },
    "associations": [],
    "types": [
      {
        "name": "LinDaiDaiStartEvent",
        "extends": [
          "bpmn:StartEvent"
        ],
        "properties": [
          {
            "name": "title",
            "isAttr": true,
            "type": "String"
          }
        ]
      }
    ]
  }
```

在这个描述文件中, 我们定义了一个新类型`LinDaiDaiStartEvent`, 该类型扩展了该类型`bpmn:StartEvent`并向其添加`“title”`属性作为属性。

**注**️: 有必要在描述符中定义要扩展的元素。如果希望该属性对所有`bpm`n元素均有效，则可以扩展`bpmn:BaseElement`️

例如🌰这样:

```javascript
...
{
  "name": "LinDaiDaiStartEvent",
  "extends": [
    "bpmn:BaseElement"
  ],
  ...
}
```


### 5. 导出并使用`AuthorityPropertiesProvider`

经过一轮翻云覆雨(C + V)的操作, 终于将大头给写完了...

下面让我们来看看怎么用它...

在`/provider/authority`文件夹下创建一个`index.js`用于导出:

```javascript
import AuthorityPropertiesProvider from './AuthorityPropertiesProvider';

export default {
  __init__: [ 'propertiesProvider' ],
  propertiesProvider: [ 'type', AuthorityPropertiesProvider ]
};
```
看着很眼熟啊, 哈哈😄... 和`contextPad`什么的好像...

用于演示, 我在项目中创建了一个`properties-panel-extension.vue`, 并在其中引用上`bpmn.js`和我们的刚刚编写好的`authority`.

```html
<template>
  <div class="containers" ref="content">
    <div class="canvas" ref="canvas"></div>
    <div id="js-properties-panel" class="panel"></div>
  </div>
</template>
<script>
// 原有的 properties-panel 这个框
import propertiesPanelModule from 'bpmn-js-properties-panel'
// 自定义的 properties-panel内容
import propertiesProviderModule from './properties-panel-extension/provider/authority';
// 引入描述文件
import authorityModdleDescriptor from './properties-panel-extension/descriptors/authority'

...
additionalModules: [
  // 右边的工具栏(固定引入)
  propertiesPanelModule,
  // 自定义右边工作栏的内容
  propertiesProviderModule
],
moddleExtensions: {
  // camunda: camundaModdleDescriptor,
  authority: authorityModdleDescriptor
}
...
</script>
```

看到这里, 相信你对`properties-panel`又有了一个新的认识...

恭喜你🎉🎉🎉

霖呆呆很是欣慰...

## 后语

上面👆教材案例的代码地址: [LinDaiDai/bpmn-vue-properties-panel](https://github.com/LinDaiDai/bpmn-vue-properties-panel)

还有几天过年了🧨...霖呆呆有个小小的愿望, 就是在年前能破`200`的粉丝...

卑微博主在线恳求关注...哈哈哈😂

(看着好心酸)

系列全部目录请查看此处:  [《全网最详bpmn.js教材目录》](https://github.com/LinDaiDai/bpmn-chinese-document/blob/master/directory.md)

最后, 如果你也对`bpmn.js` 感兴趣可以进我们的bpmn.js交流群👇👇👇, 共同学习, 共同进步.

关注霖呆呆的公众号, 选择“其它”菜单中的“bpmn.js群”即可😊.

![LinDaiDai公众号二维码.jpg](/Users/lindaidai/codes/bpmn/bpmn-chinese-document/resource/LinDaiDai公众号二维码.jpg)

