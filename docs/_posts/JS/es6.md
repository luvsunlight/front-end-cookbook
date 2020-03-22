> ES6(es2015)的新语法介绍
>
> 这里只挑最重要，最常用的语法介绍，具体的可以查看[阮一峰的这篇博客](http://es6.ruanyifeng.com/)

## es6 的部署进度

可以在[这个网站](kangax.github.io/compat-table/es6/)上查看 es6 对各个浏览器的支持进度

## let 和 const

let 主要是用于解决`块级作用域`问题的，一个经常场景如下

```js
<button>一</button>
<button>二</button>
<button>三</button>
<button>四</button>

<div id="output"></div>

<script>
  var buttons = document.querySelectorAll('button')
  var output = document.querySelector('#output')

  for (var i = 0; i < buttons.length; i++) {
    buttons[i].addEventListener('click', function() {
      output.innerText = buttons[i].innerText
    })
  }
</script>
```

乍看没有问题，但是直接点击按钮则会发现报错，如果调试的话会发现此时的 i 已经变成了 11

在 es6 种，我们只需要一个小小的变动

```js
// ...
for (/* var */ let i = 0; i < buttons.length; i++) {
	// ...
}
// ...
```

上面这段代码编译之后会变成这样

```js
// ...
var _loop = function(i) {
	buttons[i].addEventListener("click", function() {
		output.innerText = buttons[i].innerText
	})
}

for (var i = 0; i < buttons.length; i++) {
	_loop(i)
}
// ...
```

const 则很好理解，它就是用于定义常量即不可改变的量

## 箭头函数

> 重点注意它的上下文绑定

```js
let obj = {
	hello: "world",
	foo() {
		let bar = () => {
			return this.hello
		}
		return bar
	},
	barz: () => this.hello
}

window.hello = "ES6"
window.bar = obj.foo()
window.bar() //=> 'world'
obj.barz // 'undefined'
```

为什么 this 的上下文绑定会跳一个层级，我们来看这样一个例子

```js
let DataCenter = {
  baseUrl: 'http://example.com/api/data',
  search(query) {
    return fetch(`${this.baseUrl}/search?query=${query}`)
      .then(res => res.json())
      .then(rows => {
        return fetch(`${this.baseUrl}/fetch?ids=${rows.join(',')}`)
        // 此处的 this 是 DataCenter，而不是 fetch 中的某个实例
      })
      .then(res => res.json())
  }
}

DataCenter.search('iwillwen')
  .then(rows => console.log(rows))s
```

在单行匿名函数中，如果 this 指向的事该函数的上下文，则不符合直观的语义表达

:::warning
要注意的是，箭头函数对上下文的绑定是强制性的，无法通过 apply 或者 call 改变上下文
:::

## 模板字符串

```js
let str = "hello"
let str2 = `${str}world` // helloworld
```

## 对象字面量扩展语法

### 方法属性省略 function

```js
let obj = {
	// before
	foo: function() {
		return "foo"
	},

	// after
	bar() {
		return "bar"
	}
}
```

### 支持 **proto** 注入

> 主要用于拓展一个类

```js
class Foo {
	constructor() {
		this.pingMsg = "pong"
	}

	ping() {
		console.log(this.pingMsg)
	}
}

let o = {
	__proto__: new Foo(),

	constructor() {
		this.pingMsg = "alive"
	},

	msg: "bang",
	yell() {
		console.log(this.msg)
	}
}

o.yell() //=> bang
o.ping() //=> alive
```

### 可以动态计算的属性名称

```js
let arr = [1, 2, 3]
let outArr = arr.map(n => {
	return {
		[n]: n,
		[`${n}^2`]: Math.pow(n, 2)
	}
})
console.dir(outArr)[ //=>
	({ "1": 1, "1^2": 1 }, { "2": 2, "2^2": 4 }, { "3": 3, "3^2": 9 })
]
```

### 表达式解构

```js
let result = { users, posts }

let { users, posts } = result

let [x, y] = [1, 2] // 相当重要的初始化形式
```

## 新的数据结构

### 1. 类

```js
class Animal {
	yell() {
		console.log("yell")
	}
}

class Person extends Animal {
	constructor(name, gender, age) {
		super() // must call `super` before using `this` if this class has a superclass

		this.name = name
		this.gender = gender
		this.age = age
	}

	isAdult() {
		return this.age >= 18
	}
}

class Man extends Person {
	constructor(name, age) {
		super(name, "man", age)
	}
}

let me = new Man("iwillwen", 19)
console.log(me.isAdult()) //=> true
me.yell()
```

### 2. Set

Set不允许元素重复

```
const set = new Set([1,2,3,4,4])
console.log(set) // Set(4){1,2,3,4}
```

Set的方法有

* size
* add
* delete
* has
* clear（清除所有返回值）
* keys（返回所有键名的生成器）
* values（返回所有值的生成器）
* entries（返回所有键值对的生成器）
* forEach（回调函数）

### 3. Map

Map和基本对象基本一致，区别是普通对象的key只能是数字或者字符串，而Map则没有限制

```
const map = new Map();
const obj = {p: 'Hello World'};

map.set(obj, 'OK')
map.get(obj) // "OK"

map.has(obj) // true
map.delete(obj) // true
map.has(obj) // false
```

Map实例的方法有

* size
* set
* get
* has
* delete
* clear
* keys
* values
* entries
* forEach

## Generator 生成器

Generator函数具有两个特征

* function关键字和函数名之间有一个*
* 函数体内部使用yield表达式，定义不同的状态(yield表达式只能在generator函数里用)

!> js没有固定*应该写在哪，所以以下几种写法都能够通过

```js
function * foo(x, y) { ··· }
function *foo(x, y) { ··· }
function* foo(x, y) { ··· } // 推荐
function*foo(x, y) { ··· }
```

> 它的设计初衷最开始是为了提供一种简便横撑一些列对象的方法，比如计算斐波那契数列。我们直接调用generator函数是不会直接返回运行结果的，它返回一个指向内部状态的指针对象。yield表达式是暂停执行的标志，而next方法可以恢复执行。yield和ruturn相比，一个函数通过yield可以返回多个值

```js
function* fibo() {
	let [a, b] = [1, 1]

	yield a
	yield b

	while (true) {
		[a, b] = [b, a + b]
		yield b
	}
}

let generator = fibo()

for (var i = 0; i < 10; i++) console.log(generator.next().value) //=> 1 1 2 3 5 8 13 21 34 55
```

> 另外，generator函数可以不适用yield表达式，那么这种情况下，它就变成了单纯的暂缓执行函数

```js
function* f() {
  console.log('执行了！')
}

var generator = f();

setTimeout(function () {
  generator.next()
}, 2000);
```

!> yield表达式如果用在另外一个表达式之中，必须放在圆括号内

```js
function* demo() {
  console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + yield 123); // SyntaxError

  console.log('Hello' + (yield)); // OK
  console.log('Hello' + (yield 123)); // OK
}
```

> yield表达式也可以带参数，这个参数就会被当做上一个yield表达式的返回值

```
function* f() {
  for(var i = 0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}

var g = f();

g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }
```

## async函数

> async函数实际上是generator函数的语法糖

```
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

// =>

const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

### 特点

* 内置执行器，取消了*，执行过程也和普通函数一样，不需要调用next方法
* 更好的语义
* 返回值是promise对象

### 用法

因为它返回promise对象，所以可以和then配合

```
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```


js一直没有休眠的语法，我们可以通过async来实现

```
function sleep(interval) {
  return new Promise(resolve => {
    setTimeout(resolve, interval);
  })
}

// 用法
async function one2FiveInAsync() {
  for(let i = 1; i <= 5; i++) {
    console.log(i);
    await sleep(1000);
  }
}

one2FiveInAsync();
```

## 模块化

```js
import name from "module-name"
import * as name from "module-name"
import { member } from "module-name"
import { member as alias } from "module-name"
import { member1 , member2 } from "module-name"
import { member1 , member2 as alias2 , [...] } from "module-name"
import defaultMember, { member [ , [...] ] } from "module-name"
import defaultMember, * as alias from "module-name"
import defaultMember from "module-name"
import "module-name"
```

### import和require

* 这两者不是一种模块化规范，前者是官方定义的模块化解决方案，后者是node社区提出来的模块化方案
* require支持动态加载，比如我们可以require("a"+".js")这样的形式，原因就是require语法只在运行时起作用，而import则是编译时开始静态加载（这同时也意味着它会进行变量提升），不支持动态引用。而且正因为import是在编译时变量提升的，所以它一定要放在顶层上下文中
* require对基本类型的引用属于复制，对于引用类型的数据属于浅拷贝，比如说b.js引用了a.js中的obj，那么两个文件里的obj都是同一个对象
* import对于基本类型/引用类型的数据都是强绑定的，也就是说对于引用类型的数据，import和require都一样，源文件修改了，引用文件也会变化，对于基本类型的数据，import里的数据还是会随着源文件里数据的变化而变化，但require不会
* import里的值属于只读，不允许修改（对象的属性是可以修改的）
* require语法默认情况下是在node环境下运行的（CMD这种也可以，但是需要单独提供环境），import语法在script设置module为true时可用，但是要注意它import其他文件同样也会受到同源的限制，如果是file协议则请求被block，这种情况下的解决方案就是建一个服务器，改成http请求即可
* 但是绝大多数情况下，包括我们使用Vue/webpack，我们都是在以import的形式写，但实际运行的是require，因为有babel打包
* 要注意这个import和import()方法的区别，import鉴于是编译时就加载，属于静态加载，ES2020中提出了import()函数，开始支持动态加载，也不需要写在模块的顶层中了，它返回一个Promise，可以用于`按需加载`，`条件加载`，和`动态路径加载`。

## Promise

> Promise 是一种用于解决回调函数无限嵌套的工具（当然，这只是其中一种），其字面意义为“保证”。它的作用便是“免去”异步操作的回调函数，保证能通过后续监听而得到返回值，或对错误处理。它能使异步操作变得井然有序，也更好控制。我们以在浏览器中访问一个 API，解析返回的 JSON 数据

```js
// 创建promise
function fetchData() {
	return new Promise((resolve, reject) => {
		api.call("fetch_data", (err, data) => {
			if (err) return reject(err)

			resolve(data)
		})
	})
}

// 使用
fetchData()
	.then(data => {
		// ...

		return storeInFileSystem(data)
	})
	.then(data => {
		return renderUIAnimated(data)
	})
	.catch(err => console.error(err))
```

## Symbol

Symbol是ES6新增的一个`原始类型`，它的定义最核心的就是它是独一无二的值

* 不能new，因为它本质是原始类型，不过JS引擎对其也有拆箱和装箱，估计是语言定义的问题，让我们尽量以直接量的形式写它
* Symbol.toString经过改写，表示为"Symbol('a')",当然以Object.prototype.call(a)还是"[object Symbol]"
* 它的核心是独一无二，比如Symbol("a") === Symbol("a") // false,我们可能会想引用类型比如`{}==={}`也是false啊，但是要注意Symbol是原始类型，也就是说它是存储在栈里的，这就是语言特性了，模拟的话也有办法（Symbol函数内部作为工厂函数，目的是闭包保存一个uid用来唯一标输出的Symbol直接量，然后挟持symbol直接量的get方法，返回其构造值+uid的混合量）。总结Symbol最好的应用场景就是用来做内部属性，不会导致重名的情况
* Symbol是用来做防止重名属性的，而不是私有属性，它可以通过函数获取，但是因为它不会出现在遍历中，所以用来做属性名也很好用
* Symbol类型的值不可以隐式类型转换，但是可以显示转化
* Symbol.iterator可以用来包装一个对象成为一个迭代器，使其可以被for...of遍历到
* Symbol属性不会出现在对象自身以及原型链式的遍历中，不过它也不是私有属性，可以通过方法getOwnPropertySymbols获取到一个对象全部的Symbol属性值
* 我们可以通过Symbol.for的方法创建相同的Symbol直接量

## Proxy

Proxy 是 ECMAScript 中的一种新概念，它有很多好玩的用途，从基本的作用说就是：Proxy 可以在不入侵目标对象的情况下，对逻辑行为进行拦截和处理。

比如说我想记录下我代码中某些接口的使用情况，以供数据分析所用，但是因为目标代码中是严格控制的，所以不能对其进行修改，而另外写一个对象来对目标对象做代理也很麻烦。那么 Proxy 便可以提供一种比较简单的方法来实现这一需求

```js
let apiProxy = new Proxy(api, {
	get(receiver, name) {
		return function(...args) {
			min.sadd(`log:${name}`, args)
			return receiver[name].apply(receiver, args)
		}.bind(receiver)
	}
})

api.getComments(artical.id).then(comments => {
	// ...
})
```

## Reflect

* Object的`适配器`，本质上是Object的一个类实例，存放的是Object的一些底层但是有用的方法，比如defineProperty，此外以后新标准的方法都会加入到Reflect对象里
* Reflect的方法和Proxy保持一致


## ES2020

* Promise.allSettled,避免了Promise.all里面由于一个Promise的reject导致整体的reject
* 可选链 ?.
* 空值合并运算符。解决0或者""这种假值在逻辑判断时的逻辑错误，比如我们很多时候不能用if(name)这样的操作判断name是否存在因为其有可能为0，控制合并运算符就是做这个的。`name ?? "unnamed"`可以完美甄别上述区别
* 动态import
* globalThis，统一Node和浏览器环境下的全局对象
* BigInt.原来的JS中存在最大安全整数，类似于Infinity，最大安全整数+1还是等于最大安全整数，BigInt的出现就是为了操作这些大数运算的。使用方法可以通过BigInt构造函数或者在直接量后面+n即可。此外要尽量避免使用构造函数的方法，因为构造函数在传参的时候传入的还是整数，在运算时可能会有精度丢失
 