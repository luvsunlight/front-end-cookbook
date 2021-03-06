> 我做了一个css布局的展示demo [github链接](https://github.com/luvsunlight/css-layout-demo)

## 正常流

> 正常流就是：依次排列，拍不下了换行
 
## 1. display

### 块级元素（block）

* 内容独占一行（占比+分行）
* 可以设置宽高

### 行内元素（inline）

* 并不单独分行
* 具有包裹性
* 无法单独设置宽高
* 具有`断裂性`，如果段落需要分行的话，它就会失去框的特性，断裂掉
* 设置行内元素的`margin`，`border`，`padding`对周围的block没有影响

### 行内块状框（inline-block）

* 并不单独分行
* 具有包裹性
* 可以单独设置宽高
* 不具有断裂性

> inline-block元素之间存在间隙

原因是因为我们的代码中加入的换行被html当做了空格文本

```
<div class="outer">
    <div class="inner"></div>
    <div class="inner"></div>
    <div class="inner"></div>
</div>
.inner {
    width:33.33%;
    height:300px;
    display:inline-block;
    outline:solid 1px blue;
}
```

我们可以手动去掉换行

```

<div class="outer"><div class="inner"></div><div class="inner"></div><div class="inner"></div></div>
```

但是这么做，影响了源代码的可读性，一个变通的方案是，改变outer中字号为0

```
.inner {
    width:33.33%;
    height:300px;
    display:inline-block;
    outline:solid 1px blue;
    font-size:30px;
}
.outer {
    font-size:0;
}
```

### flex


## 2. float 浮动

float之后的元素会`脱离文档流`，脱离文档流最根本的特质就是它`会从正常的文档流中移除`，这表现为找不到父元素，比如如果你想设置width:80%，那么这个80%绝对不是相对于它在词法作用域中定义的父级元素。

浮动元素的定位机制是`首先在容器内尽可能地左移/右移，直至遇到了边框或者另一个浮动的元素`

在脱离文档流这个前提下，float和absolute有一点不一样，absolute的元素会覆盖行内元素，但是float的话行内元素是会以环绕的形式保留在其周围的

* 浮动元素会脱离正常文档流，并吸附至父容器的左侧（右侧），在正常文档流布局中位于该浮动元素下的内容，此时会围绕着浮动元素，填满其右侧（左侧）的空间
* 仍然遵守盒子模型诸如外边距和边界
* 脱离文档流，具有包裹性

### 清除浮动

> 有时候，我们会发现在浮动元素附近的不具有浮动特性的元素也会围绕着浮动元素而进行布局，如果我们想要消除这种特性就需要使用`clear`属性（left | both | right）,这个属性表示我们元素可以移动到左浮动元素/右浮动元素的下方

```css
clear: both;
```

#### 闭合浮动/解决浮动坍陷

我们发现float的元素会脱离文档流，这会直接导致父元素的`坍塌`，对此有一种新技术叫`清除浮动`的方法，在父元素添加名为`clearfix`的类，然后添加如下样式

```css
.clearfix {
  *zoom: 1;
}
.clearfix::after {
  content: '';
  display: table;
  clear: both;
}
```

原理很简单，就是添加在最后一个float元素加入一个after伪元素，用这个元素的clear:both属性来撑起整个容器，因为这个元素肯定是会排在float元素下的，而且它也在正常的文档流中，所以也会被父元素所包裹起来，这样就达到了父元素包裹float元素的目的，但是我们要知道父元素本质上是没有包裹它们的，因为它们已经脱离了文档流，父元素包裹的是那一个clear的伪元素罢了

## 3. position 定位

### static

> 静态定位是每个元素获取的默认值——它只是意味着“将元素放入它在文档布局流中的正常位置 ——这里没有什么特别的

### 相对定位 relative

### 绝对定位 absolute

> 绝对定位的上下文（锚）取决于绝对定位元素的父元素的position属性。如果所有的父元素都没有显式地定义position属性，那么所有的父元素默认情况下position属性都是static。结果，绝对定位元素会被包含在初始块容器中。这个初始块容器有着和浏览器视口一样的尺寸，并且<html>元素也被包含在这个容器里面。简单来说，绝对定位元素会被放在<html>元素的外面，并且根据浏览器视口来定位

注意，当absolute定位时如果没有指定left，right，top，bottom等值，它的位置默认是它在正常定位下的位置

### 固定定位 fixed

> 还有一种类型的定位覆盖——fixed。 这与绝对定位的工作方式完全相同，只有一个主要区别：绝对定位固定元素是相对于 <html> 元素或其最近的定位祖先，而固定定位固定元素则是相对于浏览器视口本身。 这意味着您可以创建固定的有用的UI项目，如持久导航菜单。
> 
> 使用fixed定位之后，元素本身将具有包裹性，即不再默认占据一整行了

### 黏性定位 sticky

粘性定位可以被认为是相对定位和固定定位的混合。元素在跨越特定阈值前为相对定位，之后为固定定位。例如：

```css
#one { position: sticky; top: 10px; }
```

在 viewport 视口滚动到元素 top 距离小于 10px 之前，元素为相对定位。之后，元素将固定在与顶部距离 10px 的位置，直到 viewport 视口回滚到阈值以下

* 父级元素不能有任何overflow:visible以外的overflow设置，否则没有粘滞效果。因为改变了滚动容器（即使没有出现滚动条）。因此，如果你的position:sticky无效，看看是不是某一个祖先元素设置了overflow:hidden，移除之即可。

* 父级元素也不能设置固定的height高度值，否则也没有粘滞效果。
* 同一个父容器中的sticky元素，如果定位值相等，则会重叠；如果属于不同父元素，则会鸠占鹊巢，挤开原来的元素，形成依次占位的效果。
* sticky定位，不仅可以设置top，基于滚动容器上边缘定位；还可以设置bottom，也就是相对底部粘滞。如果是水平滚动，也可以设置left和right值。


> 定位用的bottom，效果和top正好是对立的。设置top粘滞的元素随着往下滚动，是先滚动后固定；而设置bottom粘滞的元素则是先固定，后滚动

## 4. margin 外边距

### 外边距折叠

> 外边距有一个神奇的特性，叫外边距折叠，如果两个相邻元素都在其上设置外边距，并且两个外边距接触，则两个外边距中的较大者保留，较小的一个消失

看起来这个特性很难理解，但是我们可以把margin理解成`一个元素规定了自身周围至少需要的空间`，这样，我们就很容易理解margin需要折叠了

![](http://ww3.sinaimg.cn/large/006tNc79ly1g4p6ywflhej30h80audg5.jpg)

### 负边距

> 外边距margin可以为负值，比如`margin-left：-20px`就表示与元素在原有位置向左偏移20px。你可能会说这个功能和relative功能重叠，并不，有时候我们需要首先`absolute`将元素整体定位，这个时候再来一个负边距值就可以很好地进行二次调位，当然在`absolute`里嵌一个relative也可以，但是会让整体结构变复杂

## 5. 脱离文档流

很多途径可以脱离正常的文档流

* absolute
* fixed
* sticky
* float

> 一个共性是脱离文档流的元素都有具有**包裹性**