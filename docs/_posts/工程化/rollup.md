## rollup是什么

[官网教程](https://www.rollupjs.com/guide/zh#-bundle-creating-your-first-bundle-)

:::tip
Rollup 是一个 JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码，例如 library 或应用程序。Rollup 对代码模块使用新的标准化格式，这些标准都包含在 JavaScript 的 ES6 版本中，而不是以前的特殊解决方案，如 CommonJS 和 AMD。ES6 模块可以使你自由、无缝地使用你最喜爱的 library 中那些最有用独立函数，而你的项目不必携带其他未使用的代码。ES6 模块最终还是要由浏览器原生实现，但当前 Rollup 可以使你提前体验。
:::

在前端打包构建中，我们提到了`webpack`更适合打包前端项目，而`rollup`更适合打包第三方库

!> webpack 在打包成第三方库的时候只能导出 amd/commonjs/umd，而 rollup 能够导出 amd/commonjs/umd/es6。使用 rollup 导出 es6 模块，就可以在使用这个库的项目中构建时使用 tree-shaking 功能

相比Webpack，Rollup打包的结果惊人地简洁，和我们正常手写的没有太大区别。同时它默认开启tree-shaking，删除掉没有被引用的代码

## 特点

> 优点

* 输出结果更加扁平
* 天然支持ES6，默认有tree-shaking，自动移除未引用代码
* 打包结果依然完全可读

> 缺点

* 加载非ESM的第三方模块比较复杂
* 模块最终被打包到一个函数中，无法实现HMR
* 浏览器环境中，代码拆分功能依赖AMD的库

所以，对于应用程序，Rollup的功能很欠缺

而对于开发类库的时候，Rollup这些优点就显得很有必要

## 快速上手

### 安装

```
npm i rollup -g
```

> 对于浏览器环境

```
# compile to a <script> containing a self-executing function ('iife')
$ rollup main.js --file bundle.js --format iife
```

> 对于Nodejs环境

```
# compile to a CommonJS module ('cjs')
$ rollup main.js --file bundle.js --format cjs

```

> 对于浏览器和Nodejs环境

```
# UMD format requires a bundle name
$ rollup main.js --file bundle.js --format umd --name "myBundle"
```

## Tree-shaking

> Tree-shaking, 也被称为 “live code inclusion,” 它是清除实际上并没有在给定项目中使用的代码的过程，但是它可以更加高效

而事实上，天然支持treeshaking也正是rollup的一大亮点，rollup工具和es6的紧密结合能够让你快速爱上这款工具

比如，在使用commonjs时，必须导入完整的工具或者对象

```
// 使用 CommonJS 导入(import)完整的 utils 对象
var utils = require( 'utils' );
var query = 'Rollup';
// 使用 utils 对象的 ajax 方法
utils.ajax( 'https://api.example.com?search=' + query ).then( handleResponse );
```

而在使用es6时，你也可以只导入我们需要的部分，从而精简代码量，这对于一个第三方库来说是至关重要的一点

:::danger
为什么es模块比commonjs更好

ES模块是官方标准，也是JavaScript语言明确的发展方向，而CommonJS模块是一种特殊的传统格式，在ES模块被提出之前做为暂时的解决方案。 ES模块允许进行静态分析，从而实现像 tree-shaking 的优化，并提供诸如循环引用和动态绑定等高级功能。
:::

## 正式使用

我们可以在命令行中使用

```
rollup src/main.js -o bundle.js -f cjs
```

如果配置项越来越多，只使用cli是非常不美观的，我们可以在项目的`根目录`下新建`rollup.config.js`

```
// rollup.config.js
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
```

紧接着我们使用`--config`或者`-c`来使用配置文件

```
rm bundle.js # so we can check the command works!
rollup -c
```

:::danger
注意 Rollup 本身会处理配置文件，所以可以使用 export default 语法——代码不会经过 Babel 等类似工具编译，所以只能使用所用 Node.js 版本支持的 ES2015 语法
:::


## 配置

查看[api](https://www.rollupjs.com/guide/zh#-core-functionality-)

## rollup和其他工具集成

### 1. npm packages

#### 1.1 rollup-plugin-node-resolve

> 默认情况下，rollup不知道该如何处理从npm中下载的依赖。如果直接`import`会报错

我们需要安装插件

```
npm install --save-dev rollup-plugin-node-resolve
```

并且修改配置文件

```
# rollup.config.js
import resolve from 'rollup-plugin-node-resolve';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [ resolve() ]
};
```

#### 1.2 [rollup-plugin-commonjs
](https://github.com/rollup/rollup-plugin-commonjs)

前面的模组只是解决了rollup对npm包的依赖问题，还有个问题是，npm库中大部分是commonjs的形式出现的，我们还需要一个插件将它们转化成es6模式的模块

安装

```
npm install --save-dev rollup-plugin-commonjs
```

配置,详细信息可以参考[链接](https://github.com/rollup/rollup-plugin-commonjs)

```
// rollup.config.js
import commonjs from 'rollup-plugin-commonjs';
import nodeResolve from 'rollup-plugin-node-resolve';

export default {
  input: 'main.js',
  output: {
    file: 'bundle.js',
    format: 'iife'
  },
  plugins: [
    nodeResolve({
      jsnext: true,
      main: true
    }),

    commonjs({
      // non-CommonJS modules will be ignored, but you can also
      // specifically include/exclude files
      include: 'node_modules/**',  // Default: undefined
      exclude: [ 'node_modules/foo/**', 'node_modules/bar/**' ],  // Default: undefined
      // these values can also be regular expressions
      // include: /node_modules/

      // search for files other than .js files (must already
      // be transpiled by a previous plugin!)
      extensions: [ '.js', '.coffee' ],  // Default: [ '.js' ]

      // if true then uses of `global` won't be dealt with by this plugin
      ignoreGlobal: false,  // Default: false

      // if false then skip sourceMap generation for CommonJS modules
      sourceMap: false,  // Default: true

      // explicitly specify unresolvable named exports
      // (see below for more details)
      namedExports: { './module.js': ['foo', 'bar' ] },  // Default: undefined

      // sometimes you have to leave require statements
      // unconverted. Pass an array containing the IDs
      // or a `id => boolean` function. Only use this
      // option if you know what you're doing!
      ignore: [ 'conditional-runtime-dependency' ]
    })
  ]
};
```

### 2. babel

如果说我们需要同时用上`rollup`和`babel`，我们需要遇到一个问题，应该怎么安排两者的顺序

* 先用babel转码，注意将module排除转译
* 先用rollup，再将代码交给babel

两者各有利弊，前者，我们需要增加许多针对babel的设置，对于后者，打包速度可能更慢，因为对于babel来说，编译一个大文件比编译许多小文件要慢得多，同时增加`sourcemap`成为了一个巨大的痛点

这时，使用[rollup-babel-plugin](https://github.com/rollup/rollup-plugin-babel)成为了一个好的选择

#### 2.1 老版本

```
npm i -D rollup-plugin-babel@3 
```

配置

```

// rollup.config.js
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';

export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  },
  plugins: [
    resolve(),
    babel({
      exclude: 'node_modules/**' // 只编译我们的源代码
    })
  ]
};
```

别急，我们还需要进行配置，新建`src/.babelrc`

```
{
  "presets": [
    ["latest", {
      "es2015": {
        "modules": false
      }
    }]
  ],
  "plugins": ["external-helpers"]
}
```

首先，我们要设置`modules: false`，否则babel会在rollup有机会做处理之前，将我们的模块转化为commonjs，导致rollup的一些失败

其次，我们使用`external-helpers`插件，它允许 Rollup 在包的顶部只引用一次 “helpers”，而不是每个使用它们的模块中都引用一遍（这是默认行为）。

第三，我们将.babelrc文件放在src中，而不是根目录下。 这允许我们对于不同的任务有不同的.babelrc配置，比如像测试，如果我们以后需要的话 - 通常为单独的任务单独配置会更好。

我们还需要安装两个插件

```
npm i -D babel-preset-latest babel-plugin-external-helpers
```

#### 2.2 新版本

`npm i -D rollup-plugin-babel`

