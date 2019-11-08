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

隐式绑定即调用个位置是否有上下文对象,或者说是否被某个对象拥有或者包含。如下面的例子，单看foo是默认绑定，但是我们再调用foo的时候实际上是通过了obj这个上下文，这种情况下，隐式绑定骨子额会把函数调用中的this绑定到这个上下文对象

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

## 3. 对象

### 3.1 语法

对象可以通过声明形式（{}）和构造形式(new Object)来定义

### 3.2 类型

#### 3.2.1 基本类型

js中有7种基本类型

* String
* Number
* Boolean
* Object
* Undefined
* Null
* Symbol

有一种常见的错误是“js中万物皆对象”，这显然是错误的，实际上js中有很多特殊的对象子类型，我们可以称之为`复杂基本类型`

#### 3.2.2 内置对象

js中还有一些对象子类型，通常被称之为内置对象。有一些和基本类型的亲兄弟，不过它们的关系更加复杂

* String
* Number
* Boolean
* Object
* Function
* Array
* Date
* RegExp
* Error

js会将字面直接量先装箱再拆箱，所以对字面量也可以使用基本对象的方法

### 3.3 内容

对象的内容是由一些存储在特定命名位置的值组成的，我们称之为属性

在对象中，属性名永远都是字符串，如果我们使用string类型以外的其他值作为属性名，那么它首先会被转换为一个字符串

```
var obj = {}
obj[true] = "foo"
obj[3] = "bar"
obj[obj] = "baz"

obj["true"] // "foo"
obj["3"] // "bar"
obj["[object Object]"] // "baz"
```

#### 3.3.1 可计算属性名

我们可以通过`obj[prefix + name]`这样的形式来进行访问，主要用于Symbol

#### 3.3.2 属性和方法

在js中其实没有方法这个说法，因为没有哪个函数式真正属于一个对象的，它只是绑定到了这个对象上,它们只是对相同函数对象的多个引用

#### 3.3.3 数组

数组也支持[]访问形式，不过数组有一套更加结构化的值存储机制，数组期望的是数值下标，也就是说值存储位置是非负整数

数组也是对象，我们可以跟数组添加属性，我们给数组添加了非数值类型的属性,数组的长度不会发生变化

:::warning
鉴于这个特点，我们当然可以把数组当成一个正常的对象，但是这并不是一个好主意，因为数组和对象都根据其对应的行为和用途进行了相应的优化，最好我们都让这些数据结构各司其职
:::

如果我们试图往数组添加一个看起来很像数字的属性，那么它就会变成一个下标

```
var a = [1,2,3]
a["6"] = 3
a // [1,2,3,empty*3,3]
```

#### 3.3.4 复制对象

分为深复制和浅复制

#### 3.3.5 属性描述符

```
Object.defineProperty(obj, "a", {
    value: 2,
    writable: true,
    enumerable:true,
    configurable:true
})
```

* value 对象对应属性的值
* writbale 该属性是否可以修改
* configurable 是否可以通过defineProperty来修改，同时也会禁止delete该属性
* enumerbale 该属性是否可以会出现在对象的属性枚举中

#### 3.3.6 不变性

* writable:false && configurable:false 就可以创建一个真正的常量属性（不可以被修改，重定义和修改）
* Object.preventExtensions() 禁止给一个对象添加新属性并且保留已有属性
* Object.seal()创建一个密封的对象，实际上这个方法不仅会在现有对象上调用preventExtension，还会给所有现有属性标记为configurable:false
* Object.freeze() 在seal的基础上给所有属性添加writable:false，这样就无法修改它们的值

#### 3.3.7  [[Get]] [[Put]]

在js中的属性访问操作并不像它看起来的那么简单，它实际上是实现了一次[[Get]]操作。它会先在对象中查找是否具有该属性，没有则在原型链中查找，实在没有就返回undefined

[[Put]]操作也类似

#### 3.3.8 Setter和Getter

有两种形式

```
var obj = {
    get a(){
        return 2
    }
}
Object.defineProperty(obj, "b", {
    get: function () {return this.a * 2}
    enumerable: true
})
```

这两种方法都会在对象中创建一个不包含值的属性，对于这个属性的访问会自动调用一个隐藏函数，它的返回值会被当做属性访问的返回值

#### 3.3.9 存在性

object.a属性访问返回值可能是undefined，但是我们还不清楚是不存在这个属性还是这个值本来就是undefined

* `"a" in obj` 可以检查属性是否存在于对象及其原型链中
* `hasOwnProperty`只会检查是否在对象中，如果遇到如Object.create(null)就可以通过call来强行调用

#### 3.3.10 遍历

* for...in 会枚举对象和原型链上的所有可枚举属性，所以在遍历数组时可能会有奇怪的表现
* Object.keys() 会返回一个数组，包含对象但没有原型链上所有可枚举属性
* Object.getOwnPropertyNames()会返回一个数组，包含对象但没有原型链上所有属性，无论是否可以枚举

### 3.4 总结

js的对象有字面直接量形式（如var a = "2"），和构造形式（比如var a = new Array() || var a = Array()这两个是一样的），有时候构造形式可以提供更多选项

js万物皆对象是错误的，对象只是7种基本类型之一。对象有包括function在内的子类型，不同子类型具有不同的行为

对象就是键值对的集合。可以通过属性法或者下标法来获取属性值。访问属性时，引擎实际上是调用[[Get]]的操作

属性的特性可以通过defineProperty，除此之外还有四种方法可以冻结对象

我们有很多方法可以遍历一个对象的属性值

## 4. 混合对象“类”

### 4.1 类理论

在很长一段内，js中只有一些近似类的语法元素，比如new，instanceof和class

这是不是意味着js实际上有类了呢？不是的

虽然有近似类的语法，但是js的机制似乎一直在阻止你使用类设计语言

### 4.2 类的机制

#### 4.2.1 构造函数

类实例是由一个特殊的类方法构造的，这个方法名通常和类名相同，被称为构造函数，这个方法的任务就是初始化实例需要的所有信息。类构造函数属于类，且通常和类同名，此外够高函数大多数需要new来调，这样语言引擎才知道你想要构造一个新的类实例

#### 4.2.2 类的继承

类继承完之后，子类和父类就是两个完全不同的个体了

#### 4.2.3 多态

#### 4.2.4 多重继承

#### 4.2.5 混入

在继承或者实例化的时候，js的对象机制并不会自动执行复制行为，简单来说，js中只有对象，并不存在可以被实例化的类，一个对象并不会被复制到其他对象，它们会关联起来

##### 4.2.5.1 显式混入

可以手动for in ，也可以Object.assign

```
function mixin(source, target) {
    Object.assign(target, source)
}
var Vehicle = {
    engines: 1,
    ignition: ...,
    drive: ...
}
var Car = mixin(Vehicle, {
    wheels: 4,
    drive(){
        Vehicle.drive.call(this)
        ...
    }
})
```

js中的函数没法真正地赋值，所以你只能复制对共享函数对象的引用

一定要注意，只有在能够提高代码可读性的前提下使用显示混入，避免使用增加嗲am理解难度或者让对象关系更加复杂的模式

如果使用混入感觉越来越困难，或许我们应该停止使用了

##### 4.2.5.2 隐式混入

### 4.3 总结

类是一种设计模式，许多语言提供了面向类软件设计的原生语法。js中也有类似的语法，但是和其他语言中的类完全不一样

类意味着复制

传统的类被实例化，它的行为会被复制到实例中，类被继承时，行为也会被复制到子类中

js中并不会自动创建对象的副本

混入模式可以模拟类的复制行为，但是通常会产生丑陋并且脆弱的语法，这会让代码更加难以维护

此外，显式混入实际上只能复制一个类方法的引用，无法真正复制

总的来说，在js中模拟类是得不偿失的

## 5. 原型

## 6. 行为委托