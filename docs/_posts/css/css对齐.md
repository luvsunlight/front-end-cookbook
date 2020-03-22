## 水平居中

### 1. 行内元素

> 父元素设置`text-align:center`

### 2. 块级元素 + 宽度未定

* 元素设置`margin: 0 auto`
* flex
* absolute + left50% + transform: translateX(-50%)

### 3. 块级元素 + 宽度已定

* 上述方法
* margin-left = (parentWidth - childWidth) / 2


## 竖直居中

### 1. 行内元素

* 设置父元素的line-height等于高度即可
* 父元素设置`display: table-cell; vertical-align:middle;`

### 2. 块级元素 + 高度未定

* 父元素设置`display: table-cell; vertical-align:middle;`
* flex
* absolute + top 0 + bottom 0 + margin auto
* absolute + top 50% + transform:translateY(-50%)

### 3. 子元素是块级元素，子元素高度已定

* 上述方法
* margin-top = (parentHeight - childHeight) / 2

## vertical-align

查看[这篇文章](https://www.cnblogs.com/starof/p/4512284.html)