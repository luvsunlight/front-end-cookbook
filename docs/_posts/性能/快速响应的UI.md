## 1. 浏览器UI线程

用来执行js和更新用户界面的进程通常被称为浏览器UI线程，UI线程的工作基于一个简单的队列系统，这个队列要么运行js代码，要么执行UI更新，包括重绘和重排

```
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="UTF-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1.0" />
		<meta http-equiv="X-UA-Compatible" content="ie=edge" />
		<title>Document</title>
	</head>
	<body>
		<button>button</button>
		<script>
			while (1) {}
		</script>
	</body>
</html>
```

为了验证事件循环队列是同时执行js和UI渲染的，上述代码甚至连页面也渲染不出来，原因就是事件循环队列被占用了

当所有的UI线程都执行完毕，进程进入空闲状态，并且等待更多任务加入队列，这种状态是理想的，因为任何交互都会触发UI更新，如果用户试图在任务运行期间和页面交互，不仅没有即时的UI更新，甚至可能新的UI更新任务都不会被创建并加入队伍中

### 1.1 浏览器限制

浏览器限制了js任务的运行时间，这种限制是很有必要的，它可以确保某些恶意代码不能通过永不停止的密集操作锁住用户的浏览器或计算机。一共有两种限制

* 调用栈大小限制
* 长时间运行运行脚本限制

## 2. 使用定时器让出时间片段

### 2.1 定时器的精度

js定时器通常不太精确，相差大约几毫秒，你设置了250ms的定时器并不意味着任务一定会在调用定时器过250ms时准确加入队列

### 2.2 使用定时器处理数组

常见的一种造成长时间运行脚本的起因就是耗时过长的数组，如果我们的处理过程

* 不需要同步
* 不需要按顺序处理

那么，我们就可以用定时器来分解任务

```
var todos = items.concat()
setTimeout(function tick(){
    process(items.splice(...))
    if(todos.length > 0) {
        setTimeout(tick, 25)
    } else {
        ...
    }
}, 25)
```

> 这么做的好处是可以避免因为大批量数据的读取而阻塞浏览器的糟糕的交互体验

## 3. Web Worker

每个worker都在自己的线程中运行代码，这意味着worker运行代码不仅不会影响浏览器UI，也不会影响其他worker中运行的代码

worker没有绑定UI线程，所以它们不能访问浏览器的很多资源，worker从外部线程中修改DOM会导致用户界面出现错误，但是每个worker都有自己的全局运行环境，其功能只是js的一个子集，其环境包含如下

* 一个navigator对象，有四个属性，appName,appVersion,user Agent,platform
* 一个location对象，一个只读的window.location
* 一个self对象，指向全局worker对象
* 一个importScripts方法，用来加载worker用到的外部js文件
* 所有的ECMAScript对象，比如Object,Array...
* XMLHttpRequest构造器
* setTimeout和setInterval
* 一个close方法，可以使worker停止运行

### 3.1 与worker通信

```
// main.js
let worker = new Worker("code.js")
worker.onmessage = function(e) { 
    alert(e)
}
worker.postMessage("hi")

// worker.js
self.onmessage = e => self.postMessage('hello')
```

如上的消息系统是网页和worker通信的唯一途径，只有特定类型的数据可以使用postMessage传递，你可以传递原始值或者Object和Array的实例，但是其他类型就不行了

### 3.2 实际应用

webWorker适用于那些处理纯数据，或者与浏览器UI无关的长时间运行脚本

## 4. SIMD

单指令多数据是一种数据并行的方式，和WebWorker的任务并行相对，因为这里的重点实际上不再是把程序的逻辑分成并行的块，而是并行处理数据的多个位 

SIMD打算把CPU级的并行数学运算映射到js的api上，以获得高性能的数据并行运算，比如在大数据集上的数字处理

## 5. asm.js

asm.js描述了一个js的很小的子集，它避免了js难以优化的部分（比如垃圾回收和强制类型转换），并且让js引擎识别并通过激进的优化运行这样的代码