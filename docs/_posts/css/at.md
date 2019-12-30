## 1. 定义

css的顶层样式表由两种规则组成的规则列表构成

* @rule
* 普通规则

@rule以@关键字和后续的区块构成，如果没有区块，则以分号结尾，这些规则在平时开发中使用的机会远远小于普通的规则，所以它的大部分内容，你可能会感觉很陌生

* @charset
* @import
* @media
* @page
* @counter-style
* @keyframes
* @fontface
* @supports
* @namespace

## 2. 内容

### 2.1 @charset

用于提示css文件实用的字符编码方式，它如果被使用，必须出现在最前面

```css
@charset "utf-8"
```

### 2.2 @import 

用于引入另外一个css文件，除了@charset规则不会被引入，@import可以引入另一个文件的全部内容

```css
@import "mystyle.css"
```

### 2.3 @media

它能够对设备的类型进行一些判断

```css
@media print {
    body { font-size: 10px }
}
```

### 2.4 @page

用于分页媒体访问网页时的表现设置，页面是一种特殊的盒模型结构，除了页面本身，还可以设置它周围的盒

```css
@page {
  size: 8.5in 11in;
  margin: 10%;

  @top-left {
    content: "Hamlet";
  }
  @top-right {
    content: "Page " counter(page);
  }
}
```

### 2.5 @counter-style

产生一种数据，用于定义列表项的表现

```css
@counter-style triangle {
  system: cyclic;
  symbols: ‣;
  suffix: " ";
}
```

### 2.6 @key-frames

产生一种数据，用于定义动画关键帧

```css
@keyframes diagonal-slide {

  from {
    left: 0;
    top: 0;
  }

  to {
    left: 100px;
    top: 100px;
  }

}
```

### 2.7 @fontface

用于定义一种字体，iconFont就是利用这个特性实现的

```css
@font-face {
  font-family: Gentium;
  src: url(http://example.com/fonts/Gentium.woff);
}

p { font-family: Gentium, serif; }
```

### 2.8 @support

检查环境的特性，类似于media

### 2.9 namespace

用于跟XML命名空间配合的一个规则，表示内部的css选择器全都带上特定的命名空间

### 2.10 @viewport

用于设置视口的一些特性，不过目前兼容性不好，大部分情况下被元数据替代