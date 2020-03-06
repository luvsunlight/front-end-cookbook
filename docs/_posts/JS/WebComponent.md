## 关于WebComponents

> Web components即web原生的模块化开发。它由三项技术组成，可以一起使用来创建封装功能的定制元素，可以在喜欢的任何地方使用，不必担心代码冲突，它是组件化的实现标准，它的实现主要依赖以下三个步骤

* CustomElements。允许我们在原生DOM元素的基础上进行拓展，包括自定义属性，自定义内容，自定义元素行为，具体的实现方法是用一个类来描述自定义元素，然后再用CustomElements来define这个类，即注册
* ShadowDOM，允许我们以影子树的形式填充自定义元素的内容，影子DOM可以自定义，最后附在原DOM的一个host节点上。流程是首先用attcahShadow获取句柄，然后再添加元素。ShadowDOM的好处是
* 如果说在ShadowDOM中我们还需要自己用DOMAPI一点点构建ShadowDOM内部的DOM结构的话，template的出现就是解放JS，用原生的写DOM的形式来构建模板，然后在ShadowDOM中引用它

## Custom Elements

:::tip
注意：Firefox、Chrome和Opera默认就支持 custom elements。Safair目前只支持 autonomous custom elements（自主自定义标签），而 Edge也正在积极实现中。
:::

### 生成

我们使用`CustomElement.define(name<String>, class<Class>, option<Object>)`方法来注册一个`custon-element`

参数说明

* name => 表示所创建的元素名称符合DOMString标准的字符串，注意，它的名称不能是单个单词，而且必须要有短横线
* class => 用来定义元素行为的类
* option => 可选参数，一个包含extends属性的配置对象，它指定了所创建元素继承至哪个内置元素，可以集成任何内置元素

```
class WordCount extends HTMLParagraphElement {
	constructor() {
		// 首先必须调用super（）
		super();
		
		// 元素的功能代码写在这里
		
		...
	}
}
customElement.define(‘word-count’, WordCount, {extends: ‘p’})
```

一共有两种`custom elements`

* **Autonomous custom elements** 是独立的元素。它不继承其他内建的HTML元素。你可以直接把它们写成HTML标签的形式，来在页面上使用。例如 <popup-info>，或者是document.createElement("popup-info")这样。
* **Customized built-in elements** 继承自基本的HTML元素。在创建时，你必须指定所需扩展的元素（正如上面例子所示），使用时，需要先写出基本的元素标签，并通过 is 属性指定custom element的名称。例如`<p is=“word-count”>`, 或者 document.createElement("p", { is: "word-count" })

### 实例

```js
class RedText extends HTMLElement {
	constructor() {
		super()
		let shadow = this.attachShadow({ mode: “open” })
		let wrapper = document.createElement(“div”)
		wrapper.textContent = “div” + this.textContent
		shadow.appendChild(wrapper)
	}
}
customElements.define(“red-text”, RedText)

<red-text>hello<red-text> => divhello

class ExpandingList extends HTMLULElement {
	constructor() {
		super()
		let shadow = this.attachShadow({ mode: “open” })
		let wrapper = document.createElement(“div”)
		wrapper.textContent = “div” + this.textContent
		shadow.appendChild(wrapper)
	}
}
customElements.define(“expanding-list”, ExpandingList, { extents: ”ul” })

<ul is=“expanding-list”>

```

### 使用生命周期回调函数

在custom element的构造函数里，可以指定多个不同的回调函数

* connectedCallback：当 custom element首次被插入文档DOM时，被调用。
* disconnectedCallback：当 custom element从文档DOM中删除时，被调用。
* adoptedCallback：当 custom element被移动到新的文档时，被调用。
* attributeChangedCallback: 当 custom element增加、删除、修改自身属性时，被调用。

## Shadow DOM

> WebComponents的一个重要特性是封装，可以将html标签结构，css样式和行为隐藏起来，并从页面上中分离开，这样不同的功能不会混在一起，代码看起来也会更加干净整洁。其中shadowDOM接口是关键所在，它可以将一个隐藏的，独立的DOM添加到一个元素上

### 原理

> shadowDOM允许将隐藏的DOM树添加至常规的DOM树中，它以shadow root为起点

![](https://mdn.mozillademos.org/files/15788/shadow-dom.png)

需要了解的技术点

* shadow host 一个常规DOM节点，shadowDOM会被添加到这个节点上
* shadow tree shadowDOM内部的DOM tree
* shadow boundary shadowDOM结束的地方，也是常规DOM开始的地方
* shadow root shadow tree的节点

你可以使用同样的方式来操作Shadow DOM，就和操作常规DOM一样——例如添加子节点、设置属性，以及为节点添加自己的样式（例如通过 element.style.foo属性），或者为整个 Shadow DOM添加样式（例如在`<style>` 元素内添加样式）。不同的是，Shadow DOM内部的元素始终不会影响到它外部的元素，这为封装提供了便利。

注意，不管从哪个方面来看，Shadow DOM 都不是一个新事物——在过去的很长一段时间里，浏览器用它来封装一个元素的内部结构。以一个有着默认播放控制按钮的`<video>`元素为例。你所能看到的只是一个 `<video>`标签，实际上，在它的Shadow DOM中，包含来一系列的按钮和其他控制器。Shadow DOM标准允许你为你自己的元素（custom element）维护一组 Shadow DOM。

### 特点

* 外部可以传参进来(在CustomElements中设置attr，然后在构造函数内部使用this.getAttribute)
* 通过设置`{mode: ‘open’}`可以控制外部能不能获取该shadowRoot
* 具有封闭性，外部除了特定的接口（比如设置attr）无法影响内部的元素样式或行为，shadowDOM内部的元素乃至样式也不会污染到全局变量

## Template & slot

### 用法

template 本身就可以用，但是和`webComponents`一起使用更好

```
<template id="my-paragraph">
	<style>
    p {
      color: white;
      background-color: #666;
      padding: 5px;
    }
  </style>
  <p>My paragraph</p>
</template>
```

不会出现在你的页面中

```js
let template = document.getElementById('my-paragraph');
let templateContent = template.content;
document.body.appendChild(templateContent);

// or use custom element

customElements.define('my-paragraph',
  class extends HTMLElement {
    constructor() {
      super();
      let template = document.getElementById('my-paragraph');
      let templateContent = template.content;

      const shadowRoot = this.attachShadow({mode: 'open'})
        .appendChild(templateContent.cloneNode(true));
  }
})
```

### Template的特点

* 可以单独使用，也可以和WebComponent配合使用
* 不会出现在DOM中，可以用JS获取，其Content属性即为标准的DOM格式

### 使用`slot`

```
<my-paragraph>
  <span slot="my-text">Let's have some different text!</span>
</my-paragraph>

<my-paragraph>
  <ul slot="my-text">
    <li>Let's have some different text!</li>
    <li>In a list!</li>
  </ul>
</my-paragraph>
```

## WebComponent的优缺点

* 现行的框架比如Vue，React本质只是仿照WebComponent的实现，比如template，slot，它并没有用到这些已经有的实现
* 好处自然是显而易见的，它是标准，未来肯定是会被协商然后大面积实现的
* 为什么现在没有大面积铺开
    * 规范还没确定，浏览器支持不一
    * 还不够灵活，VDOM灵活性和性能应该要比ShadowDOM强很多