## 1. 强类型与弱类型

### 1.1 定义

> 强类型：语言层面限制函数的形参类型必须和实参类型相同

```java
class Main {
 static void foo(int num) {
    System.out.printLn(num)
 }
 public void test() {
    Main.foo('100') // Error
    Main.foo(100) // ok
 }
}
```

> 弱类型：语言层面不会限制实参的类型

但其实强弱类型没有一个官方的定义，上述表述仅为专家的意见。在理解时，可以将强类型理解为**不允许任意的数据隐式类型转换**，弱类型相反。

以语言距离，JS是一门弱类型的语言，因为其允许`100 - ‘10’`这样的操作，而Python是一门强类型的语言，上述代码在语言层面就会抛出错误而不是在运行时

## 2. 静态类型和动态类型

静态类型和动态类型较好理解，即在运行时环境下，变量的类型是否允许随意更改，JS就是一种非常典型的动态类型语言

## 3. 总结

JS类型可以参考[类型](/_posts/JS/类型)

从类型安全的角度来说，语言可以分为强类型和弱类型，区分标准是**是否允许隐式类型转换，换句话说我们是否需要提前声明变量的类型**，从类型检查的角度来说，语言可以分为静态和动态类型，区分标准是**是否允许变量类型随意更改**

JS自身是一种弱类型，动态的语言，好处是自由多变，坏处也有很多，比如臭名昭著的`==`以及各种稀奇古怪的类型问题

强类型有很多优点，比如

* 错误更早暴露（JS里有一些类型错误只能在运行时暴露，而运行时的触发环境可能很难测试）
* 代码更智能，编码更准确（在编码时，因为JS不显式指明变量类型，所以比如我们在写一个DOM元素时没法通过智能提示获取其方法和属性名）
* 重构更牢靠（更改变量名之后，强类型语言的检查器会报错，而弱类型语言智能在运行时报错）
* 减少不必要的类型判断

## 4. Flow静态类型检查方案

### 4.1 Flow基本使用

Flow是一个JS的静态类型检查器

首先我们可以通过`npm i -g flow-bin`来在全局环境下安装flow的插件

然后`flow init`一个flow配置文件

之后用`flow`命令就可以自动检查当前路径下的JS文件（文件开头要加上`//@flow`才能让flow去检查该文件）

```js
// @flow

function sum(a: number, b: number) {
    return a + b
}

sum(1, 2)

sum(100, 2)
```

可以看到Flow在JS的基础上加入了类型检查，而这也正是强类型语言的特征：在编写时就需要指明变量的类型。但是这种语法在JS下是不支持的，也就是该JS文件直接运行会报错，最好是我们先用Flow检查类型错误，没有问题的话生成一份去除类型声明的代码就好了，flow给我们提供了这样一个插件

```
npm i -D flow-remove-types
```

它可以生成一份去除掉注解的JS文件

但是这样比较麻烦，因为它和babel一样，是需要手动输入命令才能做的转换，更自动一点的方式是用babel插件`@babel/preset-flow`来在babel里加入flow的检查和代码转化，如果有webpack也可以和webpack集成

此外，还可以安装一个Flow的官方插件`Flow-language-Support`


### 4.2 Flow语法

Flow强类型的特性决定了它最基本的使用方式是指定变量的类型，这可以表现为普通变量，也可以表现为形参

```js
// 形参
function foo(num: number) {
  ...  
}

// 普通变量
let a: number = 1

// 指定函数返回值
function foo(): number {
}
```

### 4.2.1 Flow中的数组类型

Flow中表示数组方式如下

```js
const arr: Array<number> = []

// or

const arr: number[] = []

// 元组

const arr: [number, string] = [1, '1']
```

### 4.2.2 Flow中的对象类型

```js
// @flow

const obj1: { foo: string, bar: number } = { foo: "h", bar: 2 }

// ?:表示该
const obj2: { foo?: string, bar: number } = { bar: 2 }

// 
const obj3: { [string]: string } = {}

obj3.key1 = "1"
obj3.key2 = "2"

// 函数的参数
function foo(callback: (string, number) => void) {
    ...
}
foo(function(str, n))
```

### 4.2.3 Flow中的特殊类型

```js
const a: "foo" | "bar" = "foo"

const type: "success" | "warning" | "error" = "success"

const b: string | number = "123"

type StringOrNumber = string | number

const c: StringOrNumber = "234"

// Equals const gender: number | void | null = null
const gender: ?number = null
```

### 4.2.4 Mixed类型和Any类型

这两种类型都是表示任意类型，但是区别在于Mixed还是一种强类型，这意味着在没有确定变量类型之前，是不可以将其认为是某一种特定的类型进行调用的，而Any则是弱类型，它可以随意调用，不理解的话可以看下述代码

```js
function foo(value: mixed) {
    value.substr(1) // Error
    if（typeof value === "string"）{
        value.substr(1) // ok
    }
}

function bar(value: any) {
    value.substr(1)
}

```

## 5. TypeScript语言规范与基本应用

TS是一门JS的超集语言，它包含JS语言，类型系统，和最新的ES规范，相比于Flow（一个小工具），它功能更强大，生态也更健全，更完善

缺点

* 多了很多的概念
* 项目初期，增加了一些开发成本

### 5.1 TS使用指南

TS中，不同文件下有相同的变量名是会报错的，因为认为其处于同一个环境下（为什么要这么设计？），这个时候我们可以用iife来创造一个作用域，或者我们用`export`来将我们的文件声明为一个模块文件

#### 5.1.1 枚举类型

```js
enum Status {
    draft = 0,
    unpublished = 1,
    published = 2
}

const post = {
    status: Status.draft
}
```

#### 5.1.2 隐式类型转换

在TS中，如果一个变量在声明时就赋值了，那么即使没有显示指明其类型，其也会自动带上类型，比如

``` js
var a = 123
a = '123' // Error
```

这个时候只能通过声明但并不赋值的写法来兼容老代码

#### 5.1.3 类型断言

TS内置的类型有时候会产生一些问题，因为它考虑的情况很细，所以会让我们在写一些很简单的逻辑时产生意想不到的报错，比如

```js
const num = [1, 2, 3]
var val = num.find(v => v > 2)
val * val //Error 

// You can
var res = num.find(v => v > 2) as number

// or 
var res = <number>num.find(v => v > 2)
```

#### 5.1.4 接口

即用来声明一些很复杂的对象类型的声明

```js
interface Obj { hello: string }
function foo(val: Obj): void {
    console.log(val.hello)
}

foo({ hello: '1' })
```

接口中有一些特殊的标签

```js
interface Obj {
    hello: string
    warning?: string // 可选类型
    readonly world: string // 只读类型
    [prop: string]: string // 动态类型
}
function foo(val: Obj): void {
    console.log(val.hello)
    val.world = 2
}

const helloL: Obj = { hello: '1', world: '2' }

foo(helloL)
```

#### 5.1.5 类

TS中类有一些和JAVA中很像的机制，比如private，public，protected，比较有意思的是可以实现下述操作

```js
class Person {
    private name: string
    private constructor(name: string) {
        this.name = name
    }
    static create(name: string): Person {
        return new Person(name)
    }
}

var p = Person.create('hello')
```

