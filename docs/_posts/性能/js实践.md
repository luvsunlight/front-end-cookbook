## 1. 避免双重求值

js中允许动态作用域的出现，可以通过下面四种方式

* eval
* Function
* setTimeout
* setInterval

但是这么做会导致双重求值，导致效率变慢

## 2. 使用Object.Array直接量

## 3. 避免重复工作

* 延迟加载：即设置函数被调用时才被加载
* 条件预加载

## 4. 使用速度快的部分

### 4.1 使用位操作

适合将一些基础操作封装到底层

* >>1 (除2)
* & 1 (判断奇偶)

### 4.2 位掩码

适合用来判断是否通知存在多个布尔选项的情形

```
const OPTION_A = 1
const OPTION_B = 2
const OPTION_C = 4
const OPTION_D = 8
const OPTION_E = 16

let options = OPTION_A | OPTION_C | OPTION_D

Boolean(options & OPTION_A) // true 判断A是否在我们选取的OPTIONS里
```

### 4.3 原生方法

无论我们的js代码如何优化，都永远不会比js引擎提供的原生方法更快，这里是一些常见的数字常量

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

## 5. 合并多个js文件

因为http请求本身还需要时间，所以一般我们都会尽量将请求合并

## 6. js预处理

## 7. js压缩

## 8. 使用缓存

## 9. 使用内容分发网络

## 10. 尾递归优化

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