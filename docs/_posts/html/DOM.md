## 1. 定义

> DOM（文档对象模型）是针对XML但经过拓展用于HTML的应用程序API

DOM将整个页面映射为一个多层次节点结构，HTML或者XML页面中的每个组成部分都是某种类型的节点，这些节点又包含着不同类型的数据

:::warning
请注意，DOM并不是针对js的，很多别的语言也都实现了DOM，不过DOM确实已经成为了js语言的重要组成部分。（同时css也不是html体系独占的）
:::

## 2. DOM级别

DOM1级于1998年10月成为W3C（World Wide Web Consorttium万维网联盟）的标准，DOM1有两个模块：

* DOM核心。规定了如何映射基于XML的文档结构
* DOM HTML。在上者的基础上拓展，添加了针对HTML的对象和方法

DOM1是映射文档的机构，DOM2则引入了如下模块

* DOM2级核心，为节点添加了更多的方法和属性
* DOM2级视图，为文档定义了基于样式信息的不同视图
* DOM2级事件，说明了如何使用事件和DOM文档交互
* DOM2级样式。定义了基于css为元素应用样式的接口
* DOM2级HTML。在1级HTML的基础上构建，添加了更多属性，方法和新街口。定义了遍历和操作文档树的接口

DOM3的标准则进一步拓展，引入了如下模块

* 统一方式加载和保存文档的方法
* 验证文档的方法
* 对DOM核心进行了拓展，开始支持XML1.0规范

!> DOM0级是不存在的，或者说它就会一个参考点而已。

## 3. 在html中使用js

### 3.1 script标签

* 具有6个属性，async，charset，defer，language（废弃），src，type
* 不要在script标签中直接出现`</script>`标签，哪怕是在字符串中，如果需要可以使用转义`<\/script>`
* script标签和img一样，可以加载来自外域的文件，这样做有好处（跨域+JSONP），但是也有坏处，即不怀好意的程序员可能会修改外域的链接

### 3.2 标签的位置

传统做法里式放在head标签中，但是这么做可能会导致渲染阻塞。现代web应用程序一般会将其放在body元素中，以保证渲染页面元素再加载js脚本

### 3.3 defer和asnyc

defer延迟，该属性表示脚本在执行时不会影响，全部被延迟到整个页面解析完毕之后再运行（立即下载，但延迟执行）。h5规范要求延迟的脚本需要按顺序执行，并且在window.onload之前执行完毕，但是事实并非总是如此，所以最好`只包含一个延迟文本`。并且defer只适用于外部脚本，会自动忽略掉具有defer属性的内嵌脚本。

asnyc异步，同样也只适用于外部脚本。但是它没有整个页面解析完后才执行的规矩，它是立马异步执行。不过异步的代码之前不保证相互顺序。

总结，不管是延迟还是异步，都不能绝对保证代码的执行顺序，所以`最好代码里不要包含DOM操作`，更不要存在先后依赖。两者的区别在于defer是等待页面解析完再执行脚本，而asnyc是立马异步执行脚本

![](https://image-static.segmentfault.com/28/4a/284aec5bb7f16b3ef4e7482110c5ddbb_articlex)

## 3. Node类型

DOM1中规定了一个Node接口，每个节点都有一个nodeType属性，用来表示节点的类型，节点类型由在Node类型中定义的12个数值常数来决定

```
Node.ELEMENT_NODE
Node.ATTRIBUTE_NODE
Node.TEXT_NODE
Node.CDATA_SECTION_NODE
Node.ENTITY_REFERENCE_NODE
Node.ENTITY_NODE
Node.PROCESING_INSTRUCTION_NODE
Node.COMMENT_NODE
Node.DOCUMENT_NODE
Node.DOCUMENT_TYPE_NODE
Node.DOCUEMNT_FRAGMENT_NODE
Node.NOTATION_NODE
```

## 4. 事件

事件机制有两种，一个是IE首先提出来的冒泡机制，事件先从小元素开始逐级扩散，另一个是网景浏览器提出的事件捕捉机制，从上往下逐渐缩小包围

`DOM2级事件`规定的事件流包含了三个阶段，分别是

* 事件捕获阶段啊
* 处于目标阶段
* 事件冒泡阶段

事件处理被视为冒泡的一个部分，在捕获阶段什么也不会发生

### 4.1 事件类型

* UI事件
    * load，页面完全加载好后在window上触发
    * unload 跳转页面
    * abort，当用户停止下载，如果嵌入的内容没有加载完，则在元素上触发
    * error 任何没有捕获到的error都会触发window对象的
    * select
    * resize
    * scroll
* 焦点事件
    * blur
    * focus
* 鼠标与滚轮事件
    * click
    * dblclick
    * mousedown
    * mouseenter
    * mouseleave
    * mousemove
    * mouseout
    * mouseover
    * mouseup
    * mousewheel
    * 修饰键（event.shiftKey etc.）
* 键盘与文本事件
    * keydown
    * keypress
    * keyup
* 复合事件（用于处理特殊字符的输入）
* 特殊事件
    * DOMSubtreeModified
    * DOMNodeInserted
    * DOMNodeRemoved
    * DOMNodeInsertedIntoDocument
    * DOMNodeInsertedIntoDocument
    * DOMNodeRemovedFromDocument
    * DOMAtrribute
    * DOMCharacterDataModified
* H5事件
    * contextmenu事件
    * beforeunload（这个很常用，用于用户想要前往其他网站时的拦截）
    * DOMContentLoaded。DOM加载好即可触发，不需要等待css，图片等资源
    * readystatechange事件
    * pageshow和pagehide事件
    * hashchange事件
* 设备事件（都是一些移动端开发需要面对的问题）