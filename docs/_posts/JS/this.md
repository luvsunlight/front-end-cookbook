## 1. 关于this

### 1.1 为什么要用this

* this提供了一种更加优雅的方式来传递一个对象的引用，因此可以将API设计得更加简洁和易于复用
* this当然可以用上下文的形式代替，但是随着我们使用模式的复杂，显示传递上下文的方法会让代码变得越来越混乱

### 1.2 对于this的误解

#### 1.2.1 this指向自身

```
function foo(num){
    this.count++ // 这里的this实际上绑定到了全部变量上
}
foo.count = 0
for(let i=0;i<5;i++){
    foo(i)
}
console.log(foo.count)
```

上面这个程序执行的结果，最后输出foo.count结果是0，foo.count根本没有增加(将this.count++ => foo.count++ 输出为5)

#### 1.2.2 this指向它的作用域

```
function foo(){
    var a = 2
    bar()
} 
function bar(){
    console.log(this.a) // ReferenceError
}

foo()
```

## 2. this全面解析

### 2.1 绑定规则

#### 2.1.1 默认绑定

默认绑定，我们可以认为是无法应用其他规则时的默认规则，如果我们是直接使用不带任何修饰符的函数引用进行调用，那么此时就是默认绑定，在非严格模式下，this会绑定到全局变量

```
function foo(){
    console.log(this.a)
}
var a = 2
foo() // 2
```

就算出现了如下的假象，this好像绑定到了函数foo里，但实际上this是绑定在了全局变量里

```
function foo(){
    this.a = 3
    console.log(this.a) // 3
}
var a = 2
foo() // 3
console.log(a) // 3

foo.a // undefined
```

如果使用了严格模式，则不用将全部变量用于默认绑定，因此this会绑定到undefined

```
function foo(){
    "use strict"
    
    console.log(this.a) // Type Error
}
var a = 2
foo()
```

但是下面这种情况，在非严格模式下定义的函数，在严格模式里调用可以正常默认绑定到全局作用域中。这种情况很罕见，但是它是用于处理正常代码和第三方库的兼容性的

```
function foo(){
    console.log(this.a) // Type Error
}
var a = 2
(function(){
    foo()
})()
```

#### 2.1.2 隐式绑定

##### 2.1.2.1 定义

隐式绑定即调用个位置是否有上下文对象,或者说是否被某个对象拥有或者包含。如下面的例子，单看foo是默认绑定，但是我们再调用foo的时候实际上是通过了obj这个上下文，这种情况下，隐式绑定规则会把函数调用中的this绑定到这个上下文对象

```
function foo(){
    console.log(this.a)
}
var obj = {
    a:"123",
    foo:foo
}
obj.foo()
```

> 对于对象属性因用力按中，只有上一层或者说最后一层在调用位置中才有作用
 
```
function foo(){console.log(a)}
var obj1 = {a:42,foo:foo}
var obj2 = {a:32,obj1:obj1}
obj2.obj1.foo() // 42
```

##### 2.1.2.2 隐式丢失

对于隐式绑定来说，最常见的一个问题就是函数会丢失对象。只要使用了别名，this的隐式绑定就会丢失

```
function foo(){
    console.log(this.a)
}
var obj = {
    a:"123",
    foo:foo
}
var bar = obj.foo

var a = "global"

obj.foo() // "123"
foo() // global
bar() // global
```

上面的例子是为了方便理解，但在实际开发中，更为常见而且出乎意料的情况是`回调函数`,只要是回调函数，哪怕是系统自带的setTimeout这种实际上都会发生隐式丢失的情况

```
function foo(){
    console.log(this.a)
}
var obj = {
    a:"123",
    foo:foo
}
var bar = obj.foo()

var a = "global"

function doFoo(fn){fn()}

doFoo(obj.foo) // "global"
setTimeout(obj.foo,0) // "global"
```

#### 2.1.3 显式绑定

我们可以通过call，apply等方法来实现显式绑定

```
function foo(){ console.log(this.a) }
var obj = {a:123}
foo.call(obj)
```

##### 2.1.3.1 硬绑定

硬绑定的典型应用场景就是创建一个包裹函数，负责接受参数并且返回值

```
function foo(){ console.log(this.a) }
var obj = {a:123}
var bar = function (){foo.call(obj)}
bar()

bar.call(window) // 硬绑定的bar不可能再修改它的this
```

es5中提供了内置的bind函数，它会返回一个硬编码的新函数，它会把你指定的参数设置为this的上下文并且调用原始函数

bind绑定有柯里化的意思，它可以把除了第一个参数之外的其他参数全部都传给下层的函数

```
function foo(p1,p2){
    this.val = p1+p2
}
var bar = foo.bind(null, "p1")

bar("p2") // "p1p2" 
```

##### 2.1.3.2 API调用的上下文

第三方库的许多函数，以及js语言和宿主环境汇总的许多新的内置函数一样，都提供了一个可选的参数，通常被称之为上下文，其作用和bind一样，确保你的函数使用指定的this

```
function foo(){ console.log(this.a) }
var obj = {a:123}
[1,2,3].forEach(foo, obj)// 将obj绑定到this上
```

#### 2.1.4 new绑定

我们使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作

* 创建一个新的对象
* 这个新对象会被执行[[Prototype]]连接
* 这个新对象会被绑定到函数调用的this
* 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象

```
function foo(a) {
    this.a = a
}
var bar = new foo(2)
console.log(bar.a) // 2
```

:::warning
注意new还有一个前提就是构造函数有没有返回function或者object，如果是的话那么this指向这个对象。但是这样和类模式有冲突，一般情况我们最好不要这么做
:::

```
function Constructor(age) {
    this.age = age
    let obj = {a: 2}
    return obj
}
let instance = new Constructor("hello")
instance.age // undefined
```

> 手写一个new

```
function newNew(fn, ...args) {
	var obj = {}
	Object.setPrototypeOf(obj, fn.prototype)
	Object.getPrototypeOf(obj).constructor = fn.prototype
	let result = fn.apply(obj, args)
	if(result && (typeof result === "object" || typeof result === "function")){
	   return result
	}else {
	   return obj
	}
}
function Foo(name) {
	this.name = name
}
Foo.prototype.myName = function() {
	console.log(this.name)
}
var bar = newNew(Foo, 1)
var baz = newNew(Foo, 2)
bar.myName()
baz.myName()
console.log(bar.__proto__ === Foo.prototype)
```

#### 2.1.5 绑定的例外

刚才我们介绍了四种this绑定的规则，默认，隐式，显式和new。接下来我们介绍几种例外的this绑定规则

##### 2.1.5.1 被忽略的this

> 如果我们把`null`或者`undefined`作为绑定对象传入call，apply，或者bind，那么它们就会在调用时被忽略，实际应用的是`默认绑定`

我们什么时候会用到这种用法呢？apply的话可以用来数据展开（es6中可以用...代替），bind的话可以用来柯里化

```
function foo(a,b){
    console.log(a+b)
}
foo.apply(null, [1,2]) // 3
foo(...[1,2]) // 3
var bar = foo.bind(null, 1)
bar(2) // 3
```

当然，一直这样使用可能会产生一些副作用，因为这样绑定了全局作用域，可能会污染。我们需要一种更安全的方法

我们可以创建一个DMZ（非军事区对象），它就是一个空的非委托的对象.一般可以用Object.create(null)来表示，符号用ø（option+o）。

:::tip
Object.create(null)和{}很像，但是并不会创建Object.prototype这个委托，所以它比{}更空
:::

```
var ø = Object.create(null)
var bar = foo.bind(ø, 1)
bar() 
```

##### 2.1.5.2 软绑定

硬绑定固然很好用，但是它最大的缺点就是不能隐式或者显式绑定来修改this，我么可以通过软绑定

```
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this
		var curried = [].slice.call(arguments, 1)
		var bound = function() {
			return fn.apply(
				!this || this === (window || globale) ? obj : this,
				curried.concat.apply(curried, arguments)
			)
		}
		bound.prototype = Object.create(fn.prototype)
		return bound
	}
}
```

##### 2.1.5.3 箭头函数

箭头函数不使用this的四种标准规则，而是根据外层（函数或者全局）作用域来决定this

```
function foo(){
    return a => {
        console.log(this.a)
    }
}
var obj1 = {a:2}
var obj2 = {a:3}
var bar = foo.bind(obj1)
bar.call(obj2) // 2
```

上例中，this绑定的是foo的作用域，foo绑定的是obj1，所以返回的是2。箭头函数最常用于回调函数中，我们对显示绑定里的例子稍作修改

```
function foo(){ setTimeout(()=>{console.log(this.a)}) }
foo.call({a:123}) // 123
```

### 2.2 优先级

> new > 显示 > 隐式 > 默认

### 2.3 总结

判断this的指向

* 函数是否在new中调用，是的话this绑定的是新创建的对象
* 是否通过call，apply，bind，如果绑定的对象不是空，this绑定的是显式指定的对象，如果为null或者undefined，则为默认绑定
* 函数是否在某个上下文中调用，如果是的话那this绑定的就是那个隐式的对象（注意隐式丢失）
* 箭头函数中的this绑定的是外层的作用域
* 如果都不是的话，那就是默认绑定了，在非严格模式下，this绑定的是全局作用域，在严格模式下，它绑定的是undefined

有些调用可能会在无意间绑定默认规则，为了不污染全局作用域，我们可以通过`var  ø = Object.create(null)`这种形式来赋予绑定的对象，这是一种没有prototype委托的空对象，比{}还要空

