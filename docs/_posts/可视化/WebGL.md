## WebGL的组成

核心概念

* 项目
* 着色器
* 纹理

## WebGL的特点

* 可以实现更细粒度，更复杂的可视化效果呈现
* 可以满足大数据量并且对实时性有要求的可视化效果呈现
* 海量数据的处理(WebWorker也可以做到，但是我觉得WebGL可以做的更好)

## ArrayBuffer

这是BOM提供的一套API接口，是一些内置类型。主要用于WebGL相关的数据操作。相关核心概念有`ArrayBuffer`,`TypedArray`,`DataView`

ArrayBuffer即代表内存中的二进制数据段，我们可以通过new ArrayBuffer这样的形式来声明，表示我们创建了一段连续的数据区段，大小为长度个字节，但是ArrayBuffer无法直接读写，需要通过TypedArray才能对数据进行操作

TypedArray即视图，有很多种比如`Int8`，`Float32`等，它基本特性和数组一致，区别在于存储于其中的数据在内存中以特定的格式存储，我们可以以如下的形式来操作ArrayBuffer中声明的数据量

```
var buffer = new ArrayBuffer(12);

var x1 = new Int32Array(buffer);
x1[0] = 1;
```

当然，很多时候没有这么麻烦，因为TypedArray可以接受一个数组作为参数，它会自动创建对应的ArrayBuffer，并且完成对内存的赋值，这才是比较高效的做法

但实际上，你在做WebGL开发的过程，这些都不太重要，你只需要创建一个视图，然后主要麻烦的是和WebGL的交互

DataView即复合视图，如下

```
var buffer = new ArrayBuffer(24);

var idView = new Uint32Array(buffer, 0, 1);
var usernameView = new Uint8Array(buffer, 4, 16);
var amountDueView = new Float32Array(buffer, 20, 1);
```