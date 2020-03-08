## babel的定义

ES6中有很多特性会让开发过程高效且优雅，但是并不是所有浏览器都支持ES6的语法，babel就是这样的一个编译工具，它可以让你用ES6的语法写代码，然后将你的代码编译成相应的ES5中的实现

## babel的原理

和正常语言一样，有parsing,transforming,generating三个步骤

## babel和polyfill

babel是将新标准的语法编译成旧标准的语法，但是它只支持将新的语法编译，而不支持新的API，比如Array的新方法，这些API的实现需要polyfill来实现，即垫片

bable是将高语法编译成低语法。polyfill是让低语法支持高语法