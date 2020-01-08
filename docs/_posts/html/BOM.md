## 1. 定义

ECMAScript是js的核心，但如果在Web中使用js，那么BOM（浏览器对象模型）才是真正的核心。从根本上，BOM只处理浏览器窗口和框架，但是人们习惯上也把所有针对浏览器的js拓展算作BOM的一部分，下面就是这样一些拓展

* 弹出新浏览器窗口的功能
* 移动，缩放和关闭浏览器窗口
* navigator对象
* location对象
* screen对象
* cookie的支持
* XMLHttpRequet和ActiveXObject这样的自定义对象

由于在h5之前没有BOM标准可以遵循，因此每个浏览器都有自己的实现,浏览器之间共有的对象成为了事实的标准，这些对象在浏览器中得以存在，很大程度是因为它们提供了和浏览器的互操作性。在es5之后，W3C为了把浏览器中js最基本的部分标准化，已经将BOM的主要方面纳入了H5的规范中

## 2. window对象

BOM的核心对象是window 

* 在所有的全局作用域中声明的变量，函数都会变成window对象的属性和方法
* 如果页面中包含框架，则每个框架都拥有自己的window对象，并且保存在iframes集合里。
* 在BOM中有两个特殊的对象，top对象始终指向最高层的框架，比如有三个iframe页面，我们可以通过`top.frames[0]`来访问其中一个框架。parent对象始终指向当前框架的父框架
* 在BOM中，我们可以通过alert，confirm，prompt三个方法来弹出不同类型的对话框
* location对象是最有用的BOM对象之一，要注意的是window.location和document.location引用的是同一个对象
    * 查询字符串参数。location.search只会返回当前地址中从问号到url末尾的全部内容，我们可以自己写一个解析函数来处理(见下)
    * 位置操作。我们可以通过`location.assign`,`window.location = `,`location.href=`等方法来改变浏览器的位置
    * navigator对象，包含了用户的浏览器和系统等硬件信息
    * history对象，保留着用户的上网记录，也可以用来操作记录


```
function getQueryStringArgs() {
    let qs = (location.search.length > 0 ? location.search.ubstring(1) : "")
    let args = {}
    let items = qs.length ? qs.split('&') : []
    let item = null, name = null, value = null
    for(let i=0;i<len;i++) {
        item = items[i].split("=")
        name = decodeURIComponent(item[0])
        value = decodeURIComponent(item[1])
        if(name.length) {
            args[name] = value
        }
    }
}
```