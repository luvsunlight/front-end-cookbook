## 水平居中

### 1. 行内元素水平居中

> 父元素设置`text-align:center`

### 2. 块级元素水平居中

> 元素设置`margin: 0 auto`

### 3. flex	

```css
.parent {
	display: flex;
	justify-content: center/space-between/space-around;
}
```

## 竖直居中

### 1. 子元素是行内元素

* 设置父元素的line-height等于高度即可
* 父元素设置`display: table-cell; vertical-align:middle;`

### 2. 子元素是块级元素，但子元素高度未定

* 父元素设置`display: table-cell; vertical-align:middle;`
* flex。父元素设置

```css
display: flex;
align-items: center;
```

### 3. 子元素是块级元素，子元素高度已定

* 父元素设置`display: table-cell; vertical-align:middle;`
* 子元素设置`margin-top: (父元素高度-子元素高度)/2`
* 绝对定位。父元素设置`relative`，子元素设置

```css
position:absolute;
top: calc(50% - (父元素高度-子元素高度)/2);
```
s
* 绝对定位+`transform`。父元素设置`relative`

```css
position:absolute;
top: 50%;
transform:translate(0, -50%);
```

* 绝对定位+`margin：0`。父元素设置`relative`

```css
position:absolute;
top: 0;
bottom: 0;
margin: auto;
```

* flex。父元素设置

```css
display: flex;
align-items: center;
```

* flex(变种)。父元素设置

```css
display: flex;
flex-direction: column;
justify-content: center;
```