## webpack基础概念

webpack本质上是一个`静态模块打包器`，它的原理简单来说就是在处理应用时，会递归地产生一个依赖关系链，然后将这些模块全部打包成一个或者多个bundle，在webpack中万物皆模块，webpack默认能够分析的模块就是JS/JSON，它在遇到无法处理的文件类型(html/css/img)等等，都会将其按照新的模块类型来进行处理，处理的方法就是loader

在掌握webpack的用法之前，我们需要好好地了解一下webpack中的几个核心概念，这个阶段的任务是掌握大概的原理

### entry

entry用来指定webpack使用哪些模块，来作为依赖关系图的入口，进入了该模块之后，再寻找哪些模块是该文件依赖的，然后依次递归下去

> 基础用法

```js
entry: “./src/index.js”
```

> 多入口文件用法

```js

entry: {
	index: “./src/index.js”,
	search: “./src/serach.js”
}
```

### output

output告诉webpack应该在什么路径下输出整合的bundle以及如何对这些文件命名

> 基础用法

```js
output: {
	path: path.resolve(__dirname, "dist"),
	filename: "bundle.js"
}
```

> 多入口文件用法

```js
output: {
	path: path.resolve(__dirname, "dist"),
	filename: “[name].js"
}
```

### loaders

webpack开箱即用只支持js和json，对于其他类型的文件我们需要通过loaders来进行支持。本质上webpack会将所有类型的文件转化为应用程序的依赖图中可以引用到的模块

> 基础用法

```js
const path = require('path');

const config = {
  output: {
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};

module.exports = config;
```

webpack的loaders调用是链式调用，即先使用最下方的loader再逐层往上。下面的例子中即先使用css-loader处理css模块，再使用style-loader，将样式文件插入head里

```
{
	test: /.css$/,
	use: [
		‘style-loader’,
		‘css-loader’
	]
}
```

图片，字体等资源都可以通过`file-loader`或者`url-loader`来加载，`url-loader`的优点是可以base64压缩

### plugins

loaders是让webpack可以以JS/JSON以外的规则去解析不同类型的文件并且将文件以模组的形式添加到依赖关系图中，那么plugins则是应用范围更广，从打包优化压缩到定义变量，插件可以用来处理非常多的功能

插件的使用需要先require，然后在配置中加入插件的实例对象，当然这个时候我们还需要进行配置

```js
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

const config = {
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```

### 5. mode

mode可以为`development`，`production`，`none`。选择不同的`mode`webpack会自动开启对应的插件

| mode | 描述 | 
| :--: | :--: |
|none|不开启任何优化选项|
|development|设置`process.env.NODE_ENV`值为`development`，开启`NamedChunksPlugin`和`NamedModulesPlugin`|
|production|设置`process.env.NODE_ENV`值为`production`，开启`FlagDependencyUsage`,`FlagIncludedChunksPlugin`,`ModuleConcatenationPlugin`,`NoEnitOnEoorPlugin`,`OccurrenceOrderPlugin`,`SideEffectFlagPlugin`和`TenserPlugin`|

!> webpack4在mode设置成`production`之后，默认使用了`tenser-webpack-plugin`插件，这个插件可以支持es6语法的uglify

## webpack进阶用法

### 1. 自动清理构建目录

> 其实这个主要是针对有指纹的文件，没有指纹的文件是会自动替代的

#### 方法1 npm script

我们可以在`scripts`里添加 `rm -rf ./dist && webpack`

#### 方法2 clean-webpack-plugin 插件

```
npm i clean-webpack-plugin -D
```

### 2. 自动补齐css前缀

```
npm i autoprefixer -D
```

### 3. px转rem

以前使用的方法是通过媒体查询来实现自适应。相对单位的`rem`比绝对单位的`px`要适应性强的很多

```
npm i -D px2rem-loader
```

### 4. 资源内联

资源内联可以减少http网络请求数

```
npm i -D raw-loader
```

### 5. 多页面应用打包

多页面（MPA）对SEO更加友好，而且各个页面之间是相对解耦的

#### 5.1 所有页面入口文件的路径以 `src/pagename/index.js` 的形式配置

#### 5.2 安装glob

```
npm i -D glob
```

#### 5.3 配置`webpack.config.js`

```
# webpack.config.js

const setMPA = () => {
    const entry = {}
    const htmlWebPackPlugins = []
    const entryFiles = glob.sync(path.join(__dirname, "./src/*/**.js"))
    
    Object.keys(entreyFiles).map(index => {
        const entryFile = entryFiles[index]
        const match = entryFile.match(/src\(.*)\/index.js/)
        const pageName= match && match[1]
        
        entry[pageName] = entryFile
        htmlWebPackPlugins.push(new HtmlWebpackPlugin({
            ...
        }))
    })
}

setMPA()
```

### 6. 提取页面公共资源

基础库分离，可以减少最后打包的体积

#### 方法1 使用`html-webpack-externals-plugin`将`react`这种基础库以CDN的形式引入这样基础库就不会进入最终的打包结果了

#### 方法2 使用`SpiltChunksPlugin`进行公共脚本分离

webpack4内置，替代`CommonsChunkPlugin`

chunks参数说明
    * asnyc 异步引入的库进行分离（默认）
    * inistial 同步引入的库进行分离
    * all 所有引入的库进行分离（推荐）

 
### 7. tree shaking

#### 使用

一个模块可能有多个方法，只要其中每个方法用到了，那么整个文件都会被搭载bundle里，tree shaking就是只把用到的方法打入bundle，没用到的方法会在uglify阶段被擦除

* webpack默认之处，在`.babelrc`里设置`modules: false`即可
* `production mode`下默认开启
* 必须是es6的语法

#### 原理

> 利用es6模块的特点

* 只能作为模块顶层的语句出现
* import的模块名只能是字符串变量
* import binding 是immutable的

### 8. scope hoisting

随着引入模块的增加，大量函数闭包包裹代码，导致体积增大。同时，运行代码时创建的函数作用域变多，内存开销变大

#### 原理

将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突。通过scope hoisting 可以减少函数声明代码和内存开销

### 9. 代码分割和动态import

#### 代码分割的意义

对应大的web应用来讲，将所有的代码都放在一个文件中是显然不够有效的，特别是你的代码是在一些特殊情况下才会被用到，webpack有一个功能就是将你的代码库分割成chunks，当代码运行到需要的时候再加载

#### 适用的场景

* 抽离相同代码到一个共享块
* 脚本懒加载，使得初始下载的代码更小

#### 懒加载js的方式

* cjs: require.ensure
* es6: 动态import（原生没有实现，需要babel转换）

#### 如何使用动态import

> 安装babel插件

```
npm i -D @babel/plugin-syntax-dynamic-import
```

> 设置`.babelrc`

```
"plugins": ["@babel/plugin-syntax-dynamic-import"]
...
```

### 10. webpack+eslint

* 不重复造轮子，基于eslint:recommend配置并改进
* 能够帮助发现代码错误的规则，全部开启
* 能够帮助保持团队的代码风格统一，而不是限制开发体验 

### 11. webpack打包基础库和组件

* 打包压缩版和非压缩版
* 支持AMD/ejs/cjs的版本

其实打包库更推荐rollup

### 12. webpack实现ssr打包

客户端渲染

* 开始加载html
* html加载成功，开始加载数据
* 数据加载成功，渲染成功开始，加载图片资源
* 图片加载成功，页面可交互

ssr服务端渲染

* 所有模板等资源都存储在服务端
* 内网机器拉取数据更快
* 一个html返回所有数据 

优点

* 减少白屏时间
* 对SEO更加友好  

服务端渲染的核心是减少请求，渲染的缓解都在服务端完成，客户端只需要请求一次，服务端直接返回html文件

### 13. 优化构建时命令行的显示日志

#### 方法1 原生方法：设置stats

| errors-only | 只在错误时输出 |
| --- | --- |
| minimal | 只在错误或者有新的编译时输出 |
| none | 没有输出 |
| normal | 标准输出 |
| verbose | 全部输出 |

#### 方法2 插件 friendly-errors-webpack-plugin

```
npm i -D friendly-errors-webpack-plugin
```

### 14. 构建异常和中断处理

> 怎么判断构建成功

echo $？ 返回值为错误码

nodejs中`process.exit`

* 0表示成功完成，回调函数里，err为null
* 非0表示执行失败，回调函数中，err不为null，err.code就是传给exit的数字

在webpack4中，我们可以在plugins里通过hooks.done这个钩子来处理构建完成这个事件

```
plugins:[
    ...
    function() {
        this.hooks.done...
    }
]
```

### 15. 文件指纹

* hash 和整个项目的构建相关，只要项目文件有修改，整个项目的构建的hash都会更改
* chunkhash 和webpack打包的chunk有关，不同的entry会生成不同的chunkhash值
* contenthash 根据文件内容来定义hash，文件内容不变，则contenthash不变

js可以用`chunkhash`，css可以有`contenthash`


## 编写可维护的webpack构建配置

### 构建配置包设计

如果是要做一个团队可用的流程工具链的话，最好的办法是发布成一个工具包

通用性
    * 业务开发者无需关注构建配置
    * 统一团队构建脚本

可维护性
    * 构建配置合理的拆分
    * README 文档，ChangeLog文档等

质量
    * 冒烟测试，单元测试，测试覆盖率
    * 持续集成

#### 构建配置管理的可选方案

* 通过多个配置文件管理不同环境的构建， 通过`webpack --config`参数进行控制
* 将构建配置设计成一个库，比如hjs-webpack，neutrino，webpack-blocks（适合小团队，10-20人）
* 抽成一个工具进行管理，比如create-react-app，nwb（适合较大团队）
* 将所有的配置放在一个文件，通过--env参数控制分支选择

#### 构建配置包设计

> 通过多个配置文件管理不同环境的webpack配置（我们可以通过webpack-merge来组合配置）

* 基础配置 webpack.base.js
* 开发环境 webpack.dev.js
* 生产环境 webpack.prod.js
* ssr环境 webpack.ssr.js
* pwa环境 webpack.pwa.js

> 抽离成一个npm包统一管理

* 规范 git commit， readme， eslint， semever
* 质量 冒烟测试，单元测试，测试覆盖率和CI

### 功能模块设计和目录结构

#### 功能模块设计

构建包功能设计
* 基础配置
    * 资源解析
        * 解析es6
        * 解析react
        * 解析css
        * 解析less
        * 解析图片
        * 解析字体
    * 样式增强
        * 样式前缀补齐
        * csspx 转换成rem
    * 目录清理
    * 多页面打包
    * 命令行信息显示优化
    * 错误捕获和处理
    * css提取成单独的文件
* 开发配置
    * 代码热更新
        * css热更新
        * js热更新
    * sourcemap
* 生产配置
    * 代码压缩
    * 文件指纹
    * tree shaking
    * scope hoisting
    * 速度优化
        * 基础包CDN
    * 体积优化
        * 代码分割
* ssr配置
    * output的libraryTarget设置
    * css解析ignore

#### 目录结构设计

* test
* lib
    * webpack.dev.js
    * webpack.prod.js
    * webpack.ssr.js
    * webpack.base.js
* README.md
* CHANGELOG.md
* .eslintrc.js
* package.json
* index.js

### 使用eslint规范构建脚本

使用`eslint-config-airbnb-base`

eslint --fix 可以自动处理空格

### 冒烟测试和实际应用

> 冒烟测试是指对提交测试的软件在进行详细深入的测试之前而进行的与测试，这种预测试的主要目的是暴露导致软件需要重新发布的基本功能失效等严重问题

#### 冒烟测试执行

* 构建是否成功
* 每次构建完成后build目录是否有内容输出
    * 是否有js，css等静态资源文件
    * 是否有html文件

### 单元测试和测试覆盖率

### 持续集成和travis CI

### 发布构建包到npm社区

### git commit规范和changelog生成

### 语义化版本规范格式

## webpack构件速度和体积优化策略

### 使用webpack内置的stats

```
"scripts": {
    "build:stats": "webpack -- env production --json > stats.json"
}
```

但是这样的话颗粒度太粗了

### 使用speed-measure-webpack-plugin

```
const SMP = require("speed-measure-webpack-plugin")

const smp = new SMP()

cons webpackConfig = smp.wrap({
    plugins: [
        ...
    ]
})
```

> 可以看到引入的各个loader和插件分别耗时了多少时间

### 使用webpack-bundle-analyzer

这个插件会以树图的形式来分析最后打包的体积分析

### 使用高版本的webpack和nodejs

最好使用最新版本的webpack和nodejs，性能可以提升非常巨大

### 多进程/多实例构建

> 可选方案

* thread-loader(webpack4.x原生支持·)
* parallel-webpack
* happypack

### 多进程并行压缩代码

* 使用parallel-uglify-plugin插件
* 使用uglify-webpack-plugin开启parallel参数
* 使用terser-webpack-plugin（webpack4.x默认支持）

### 进一步分包:预编译资源模块

之前我们提到了，可以使用`html-webpack-external-plugin`或者`splitChunksPlugin`插件来进行分包，但是我们这里更推荐一种做法

因为前面的方法优缺点就是，一个基础库需要一个指定的CDN，需要引入很多`script标签`

更好的解决方法是使用预编译资源模块。使用dllplugin进行分包，DllReferencePlugin对manifest.json引用

### 充分利用缓存提升二次构建速度

缓存思路

* babel-loader开启缓存
* terser-webpack-plugin开启缓存
* 使用cache-loader或者hard-source-webpack-plugin·

### 缩小构建目标

尽可能地少构建模块，多用exclude属性

比如babel-loader不解析node_modules

减少文件搜索范围

* 优化resolve.modules配置（减少模块搜索层级）
* 优化resolve.mainFields配置
* 优化resolve.extensions配置
* 合理使用alias

### 使用tree-shaking擦除无用的js和css

* purifycss 遍历代码，识别已经用到的css class
* uncss 需要通过jsdom加载，所有的样式通过postcss加载，通过document.queryselector来识别在HTML文件里不存在的选择器

### 使用webpack进行图片压缩

配置image-webpack-loader

### 使用动态polyfill服务


| 方案 | 优点 | 缺点 |
| --- | --- | --- |
| babel-polyfill | react16官方推荐 | 包体积过大 |
| babel-plugin-transform-runtime | 能只polyfill用到的类或者方法，相对体积小 | 不能polyfill原型上的方法，不适用于业务项目的复杂开发环境 |
| 自己写map，set的polyfill | 定制化高，体积小 | 重复造轮子，容易年久失修 |
| polyfill-service | 只能给用户返回需要的polyfill，社区维护 | 部分国内奇葩浏览器UA可能无法识别，但是可以降级返回所需全部的polyfill |

动态polyfill => 识别UA，下发不同的polyfill

## 通过源码掌握webpack打包原理

### webpack启动过程

cli实际上是调用了node_nodules/.bin里面去找对应的bin文件，如果存在则调用，不存在则抛出错误

### webpack-cli源码阅读

webpack-cli做的事

* 引入yargs，对命令行进行定制
* 分析命令行参数，对各个参数进行转换，组成编译配置项
* 引入webpack，根据配置项进行编译和构建 

### tapable插件架构和hooks设计

### tapable是如何和webpack进行关联起来了

### webpack流程

#### 准备阶段

#### 模块构建和chunk生成

#### 文件生成

### 动手编写一个建议的webpack

## 编写loader和插件

### loader的链式调用和执行顺序

### 使用loader-runner高效进行loader的调试

### 更复杂的loader的开发场

### 实战开发一个自动合成雪碧图的loader

### 插件基本结构介绍

### 更复杂的插件开发场景

### 实战开发一个压缩构建资源为zip包的插件