!> 有一些 css 属性我们很少接触到或者甚至都没听说过，但是善用它们有时可以给我们的开发带来意想不到的帮助和效率

## 文本

### text-transform

> 允许你设置要转换的字体
>
> 值包括：

-   none: 防止任何转型。
-   uppercase: 将所有文本转为大写。
-   lowercase: 将所有文本转为小写。
-   capitalize: 转换所有单词让其首字母大写。
-   full-width: 将所有字形转换成固定宽度的正方形，类似于等宽字体，允许对齐。拉丁字符以及亚洲语言字形（如中文，日文，韩文）

### text-decoration

> 设置/取消字体上的文本装饰 (你将主要使用此方法在设置链接时取消设置链接上的默认下划线 )
>
> 可用值为：

-   none: 取消已经存在的任何文本装饰
-   underline: 文本下划线
-   overline: 文本上划线
-   line-through: 穿过文本的线 strikethrough over the text

同时使用`text-decoration: underline overline;`

:::tip
text-decoration 是一个缩写形式，它是由 text-decoration-line,text-decoration-style，text-decoration-color 构成
:::

```
<div class="wave">wave</div>

<style>
.wave {
    text-decoration: line-through red wavy;
}
</style>
```

output: <span class="wave">wave</span>

<style>
.wave {
    text-decoration: line-through red wavy;
}
.paper {
	width: 100px;
  height: 100px;
  box-shadow: 3px 3px 5px #ccc,
    5px 5px 1px #fff,
  8px 8px 5px #ccc;
}
</style>

#### text-decoration-style

-   solid
-   double (双重下划线)
-   dotted
-   dashed
-   wavy（波浪线）

### 文本阴影

> text-shadow: 4px 4px 5px red;

### white-space

> 主要用于设置如何处理元素中的空白
>
> 可取值

#### normal

连续的空白符会被合并，换行符会被当作空白符来处理。填充 line 盒子时，必要的话会换行

#### nowrap

和 normal 一样，连续的空白符会被合并。但文本内的换行无效

#### pre

连续的空白符会被保留。在遇到换行符或者`<br>`元素时才会换行。

#### pre-wrap

连续的空白符会被保留。在遇到换行符或者`<br>`元素，或者需要为了填充 line 盒子时才会换行

#### pre-line

连续的空白符会被合并。在遇到换行符或者`<br>`元素，或者需要为了填充 line 盒子时会换行

#### inherit

## 区块

### border-image

> [查看 MDN 上的链接](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Styling_boxes/Borders)

### box-shadow

> 一般的 boxshadow 可能都比较熟悉，但是 boxshadow 可以叠加，使用逗号隔开，形成一种层叠的感觉

```css
.paper {
	width: 100px;
	height: 100px;
	box-shadow: 3px 3px 5px #ccc, 5px 5px 1px #fff, 8px 8px 5px #ccc;
}
```

<div class="paper"></div>

#### 内部阴影

与text-shadow不同，box-shadow有一个`inset`关键字可用——把它放在一个影子声明的开始，使它变成一个内部阴影，而不是一个外部阴影

#### 滤镜 filter

> [查看MDN](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Styling_boxes/Advanced_box_effects)

### vertical-align

:::tip
CSS 的属性 vertical-align 用来指定行内元素（inline）或表格单元格（table-cell）元素的垂直对齐方式
:::

* baseline
* top
* middle
* bottom
* sub
* text-top

### 修改scrollbar

```css
* {
  // 控制整体的滚动条样式
  &::-webkit-scrollbar {
    width: 10px;
    height: 4px;
  }
  // 滚动滑块的样式
  &::-webkit-scrollbar-thumb {
    border-radius: 5px;
    background: #dfe6e9;
  }
  // 滚动槽的样式
  &::-webkit-scrollbar-track{
   -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
   border-radius: 0;
   background: rgba(0,0,0,0.1);
  }
}
```

修改后的滚动条如下所示

![](http://ww1.sinaimg.cn/large/006tNc79ly1g4pa7140sej301t0cb0sk.jpg)