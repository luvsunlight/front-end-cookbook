## 1. 优化页面体积，提升网络加载

* 静态资源的压缩合并（js压缩，合并，css压缩，合并，雪碧图）
* 静态资源缓存（资源+md5）
* 使用CDN

## 2. 优化页面渲染

* css放js前面
* 懒加载
* 减少DOM查询，对DOM查询做缓存
* 减少DOM操作，多个操作尽量合并在一起（先加到Fragment上）
* 防抖和节流
* ssr
* 避免使用动态作用域（eval,with,setTimeout,setInterval）
* 使用位操作(& 1, >> 1)
* 使用位掩码
* 使用原生的方法

> 尾递归优化

<details>
<summary>查看解析</summary>
es6包含了一个性能领域的特殊要求，这与一个设计函数调用的特定优化形式相关，尾递归优化

> 简单来说，尾调用就是出现在另外一个函数结尾处的函数调用，这个调用结束后就没有其他事情要做了（除了可能要返回结果值）

```
function foo(x) {
    return x
}
function bar(y) {
    return foo(y + 1)
}
function baz() {
    return 1 + bar(40)
}
```

上例中的foo(y+1)是bar中的尾调用，因为foo完成后，bar也完成了，但是bar不是baz的尾调用，因为它还有一个+1的操作

正常情况下，调用一个新的函数需要额外的一块内存来管理调用栈，称为栈帧，所以前面的代码需要为三个函数各保留一个栈帧

但是如果支持TCO(尾递归优化)的引擎能够认识到这里可以进行TCO优化，那么在调用foo的时候，它就不需要创建一个新的栈帧，而是可以重用已经有的bar的栈帧，这样速度更快，也更节省内存

TCO乍一看好像也没有节省太多内存，但是它的特点决定了它及其适合用于递归的情景，无限次递归也只需要一个栈帧，引擎就不需要限制栈的深度了。于是我们在写递归的时候，就要尽量把它按照尾递归优化的情况去写

```
function fac(n) {
    function fact(n, res) {
        return n < 2 ? res : fact(n - 1, n * res)
    }
    return fact(n, 1)
}
```

</details>


> 位掩码的使用方式

<details>
<summary>查看解析</summary>
```
const OPTION_A = 1
const OPTION_B = 2
const OPTION_C = 4
const OPTION_D = 8
const OPTION_E = 16

let options = OPTION_A | OPTION_C | OPTION_D

Boolean(options & OPTION_A) // true 判断A是否在我们选取的OPTIONS里
```

</details>

> 原生的Math方法

<details>
<summary>查看解析</summary>

| 常量 | 值 |
| --- | --- |
| Math.E | E |
| Math.LN10 | log10 |
| Math.LN2 | log2 |
| Math.LOG2E | log2 e |
| Math.LOG10E | log10 e |
| Math.PI | pi |
| Math.SQRT1_2 | sqrt（1/2） |
| Math.SQRT2 | sqrt(2) |


| 方法 | 含义 |
| --- | --- |
| Math.abs |  |
| Math.exp |  |
| Math.log |  |
| Math.pow |  |
| Math.sqrt |  |
| Math.acos |  |
| Math.asin |  |
| Math.atan |  |
| Math.atan2 |  |
| Math.cos |  |
| Math.sin |  |
| Math.tan |  |

</details>



