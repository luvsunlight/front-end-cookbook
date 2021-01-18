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

loaders是让webpack可以以JS/JSON以外的规则去解析不同类型的文件并且将文件以模组的形式添加到依赖关系图中，那么plugins则是应用范围更广，从打包优化压缩到定义变量，插件可以用来处理非常多的功能,它可以用于除了加载资源以外的其他的自动化工作，这里的话，感觉就像是Gulp中的各式各样的插件了

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

### mode

mode可以为`development`，`production`，`none`。选择不同的`mode`webpack会自动开启对应的插件

| mode | 描述 | 
| :--: | :--: |
|none|不开启任何优化选项|
|development|设置`process.env.NODE_ENV`值为`development`，开启`NamedChunksPlugin`和`NamedModulesPlugin`|
|production|设置`process.env.NODE_ENV`值为`production`，开启`FlagDependencyUsage`,`FlagIncludedChunksPlugin`,`ModuleConcatenationPlugin`,`NoEnitOnEoorPlugin`,`OccurrenceOrderPlugin`,`SideEffectFlagPlugin`和`TenserPlugin`|

!> webpack4在mode设置成`production`之后，默认使用了`tenser-webpack-plugin`插件，这个插件可以支持es6语法的uglify

## Webpack基础使用

### 无配置使用

Webpack4之后支持无配置进行打包，它默认入口为`src/index.js`，默认打包出口为`dist/main.js`，我们只需要如下操作

```js
yarn add webpack webpack-cli

yarn webpack
```

就可以借助webpack来帮助你完成模块化和打包的任务

但是我们要注意Webpack默认只对JS和JSON类型的文件进行处理，对于其他文件需要不同的Loader 

### CSS文件的打包

我们在webpack配置文件中加入

```js
const path = require('path')

module.exports = {
  mode: 'none',
  entry: './src/main.css',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      }
    ]
  }
}
```

其中css-loader负责将CSS文件转化为JS模块，但是转化为模块之后还是没有添加进HTML中，于是我们还需要style-loader来帮助我们完成这个操作

### 文件资源的打包

比如图片（样式文件也可以）可以通过这样的形式基于Webpack进行打包

```js
import createHeading from './heading.js'
import './main.css'
import icon from './icon.png'
```

这时当然也需要特定的Loader来帮助我们顺利导入图片文件

```js
const path = require('path')

module.exports = {
  mode: 'none',
  entry: './src/main.js',
  output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      },
      {
        test: /.png$/,
        use: 'file-loader'
      }
    ]
  }
}
```

但是这个时候如果我们直接在项目根目录下启动服务器是会有问题的，因为我们会发现打包后的文件图片的地址引用的是根目录下的，原来Webpack默认认为文件的相对路径是相对根目录的，我们需要在Webpack设置中进行配置

```js
output: {
    filename: 'bundle.js',
    path: path.join(__dirname, 'dist'),
    publicPath: 'dist/'
  },
```

### DataURLS

DataURLS是一种文件传输格式，传统的URL是服务器中存在某种资源的文件，前端通过URL去发动HTTP请求去获取这些文件，而DataURL是通过某种特定过的格式，将文件的内容用URL表示，格式如下

> data:[<mediatype>][;base64],<data(文件内容)>

比如，我们在浏览器的导航中输入下列内容，可以直接在浏览器中呈现HTML内容

```
data:text/html;charset=UTF-8,<h1>html</h1>
```

对于图片或者字体这类无法直接用文字表示的二进制内容，可以用Base64编码

```
data:image/png;base64,iVB0...SuQmCG
```

在Webpack中我们可以通过这个原理，将图片和文字转化为BASE64格式的DATAURLS  ，我们只需要对png格式的文件用url-loader导入即可

另外，我们可以在url-loader中设置limit属性，比如

```js
{
    test: /.png$/,
    use: {
      loader: 'url-loader',
      options: {
        limit: 10 * 1024 // 10 KB
      }
    }
  }
```

这里表示小于10kb的文件用url-loader处理，大于的则还是用file-loader处理，在配置文件中不需要显式注明file-loader，url-loader会自动做这一点，但是我们必须安装file-loader

!> 值得注意的是，这个方法比较适合一些体积小的文件，因为大的文件打包起来非常复杂，影响效率，同样用数据表示文件可以减少网络请求次数

### ES6

```js
yarn add -D babel-loader @babel/core @babel/preset-env
```

配置

```js
{
    test: /.js$/,
    use: {
      loader: 'babel-loader',
      options: {
        presets: ['@babel/preset-env']
      }
    }
  },
```

### 加载资源的方式

* 符合ES6语法的导入导出方式
* 符合CJS规范的require（注意，require的文件还需要调用其default属性才能正常获取值）
* 符合AMD标准的define函数和require函数
* CSS中的@import和url也会触发Webpack的加载资源格式
* HTML中图片的src属性也会触发

## Webpack的开发体验优化

### 自动编译

即修改完源代码之后，Webpack监测到文件变化，自动进行编译工作，当然这个时候还需要在浏览器中手动刷新，实现伪自动化

要实现这一点，只需要在Cli中使用`webpack --watch`即可

### DevServer

即实现自动编译+自动刷新浏览器（该功能可由browser-sync提供，但是在这里选择Webpack集成的功能更好）的功能

```js
yarn add webpack-dev-server

yarn webpack-dev-server
```

于此同时，我们可以在webpack的配置文件中加入对DevServer的额外配置

```js
devServer: {
    contentBase: './public'
}
```

这里的配置是给当前的DevServer**指定额外的静态资源路径**，加入了之后，在Serve阶段，在contentBase目录下的文件也能被serve了

### 代理API 

我们可以通过devServer中的配置来实现代理API

```js
devServer: {
    contentBase: './public',
    proxy: {
      '/api': {
        // http://localhost:8080/api/users -> https://api.github.com/api/users
        target: 'https://api.github.com',
        // http://localhost:8080/api/users -> https://api.github.com/users
        pathRewrite: {
          '^/api': ''
        },
        // 不能使用 localhost:8080 作为请求 GitHub 的主机名
        changeOrigin: true
      }
    }
  }
```

### HMR

Webpack默认的刷新只支持全量刷新，可能会导致页面状态丢失，那么有没有一种办法是保留状态的同时进行页面的增量刷新，热更新（HMR）就是这样一种技术

Webpack中的HMR默认不支持对JS的热更新，因为JS的变化千差万别，没有办法去判断具体会对页面的哪一个部分产生影响

!>注意CSS等资源可能会默认有热更新的情况，这是因为Webpack中CSS是由Loader处理过的，而这个处理过程就已经加入了热更新

> 我们需要手动处理对JS的热更新

在Webpack中提供了一系列API，来进行热替换

```js
import createEditor from './editor'
import background from './better.png'
import './global.css'

const editor = createEditor()
document.body.appendChild(editor)

const img = new Image()
img.src = background
document.body.appendChild(img)
// ================================================================
// HMR 手动处理模块热更新
// 不用担心这些代码在生产环境冗余的问题，因为通过 webpack 打包后，
// 这些代码全部会被移除，这些只是开发阶段用到
if (module.hot) {
  let hotEditor = editor
  module.hot.accept('./editor.js', () => {
    // 当 editor.js 更新，自动执行此函数
    // 临时记录编辑器内容
    const value = hotEditor.innerHTML
    // 移除更新前的元素
    document.body.removeChild(hotEditor)
    // 创建新的编辑器
    // 此时 createEditor 已经是更新过后的函数了
    hotEditor = createEditor()
    // 还原编辑器内容
    hotEditor.innerHTML = value
    // 追加到页面
    document.body.appendChild(hotEditor)
  })

  module.hot.accept('./better.png', () => {
    // 当 better.png 更新后执行
    // 重写设置 src 会触发图片元素重新加载，从而局部更新图片
    img.src = background
  })

  // style-loader 内部自动处理更新样式，所以不需要手动处理样式模块
}
```

### 不同环境的配置文件

简单的做法可以在Webpack的配置目录下通过env来基于不同的配置

```js
const webpack = require('webpack')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CopyWebpackPlugin = require('copy-webpack-plugin')

module.exports = (env, argv) => {
  const config = {
    mode: 'development',
    entry: './src/main.js',
    output: {
      filename: 'js/bundle.js'
    },
    devtool: 'cheap-eval-module-source-map',
    devServer: {
      hot: true,
      contentBase: 'public'
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        },
        {
          test: /\.(png|jpe?g|gif)$/,
          use: {
            loader: 'file-loader',
            options: {
              outputPath: 'img',
              name: '[name].[ext]'
            }
          }
        }
      ]
    },
    plugins: [
      new HtmlWebpackPlugin({
        title: 'Webpack Tutorial',
        template: './src/index.html'
      }),
      new webpack.HotModuleReplacementPlugin()
    ]
  }

  if (env === 'production') {
    config.mode = 'production'
    config.devtool = false
    config.plugins = [
      ...config.plugins,
      new CleanWebpackPlugin(),
      new CopyWebpackPlugin(['public'])
    ]
  }

  return config
}
```

更复杂的可以根据环境创建多个Webpack配置文件，通过merge来融合配置

```js
#webpack.dev.js
const webpack = require('webpack')
const merge = require('webpack-merge')
const common = require('./webpack.common')

module.exports = merge(common, {
  mode: 'development',
  devtool: 'cheap-eval-module-source-map',
  devServer: {
    hot: true,
    contentBase: 'public'
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
})
```

## webpack进阶用法

### DefinePlugin

用于注入全局变量

```js
plugins: [
    new webpack.DefinePlugin({
      // 值要求的是一个代码片段
      API_BASE_URL: JSON.stringify('https://api.example.com')
    })
  ]
```

### 自动清理构建目录

> 其实这个主要是针对有指纹的文件，没有指纹的文件是会自动替代的

#### 方法1 npm script

我们可以在`scripts`里添加 `rm -rf ./dist && webpack`

#### 方法2 clean-webpack-plugin 插件

```
npm i clean-webpack-plugin -D
```

### 自动补齐css前缀

```
npm i autoprefixer -D
```

### px转rem

以前使用的方法是通过媒体查询来实现自适应。相对单位的`rem`比绝对单位的`px`要适应性强的很多

```
npm i -D px2rem-loader
```

### 资源内联

资源内联可以减少http网络请求数

```
npm i -D raw-loader
```

### 多页面应用打包

多页面（MPA）对SEO更加友好，而且各个页面之间是相对解耦的

#### 所有页面入口文件的路径以 `src/pagename/index.js` 的形式配置

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

### 提取页面公共资源

基础库分离，可以减少最后打包的体积

#### 方法1 使用`html-webpack-externals-plugin`将`react`这种基础库以CDN的形式引入这样基础库就不会进入最终的打包结果了

#### 方法2 使用`SpiltChunksPlugin`进行公共脚本分离

webpack4内置，替代`CommonsChunkPlugin`

```js
mode: 'none',
  entry: {
    index: './src/index.js',
    album: './src/album.js'
  },
  output: {
    filename: '[name].bundle.js'
  },
  optimization: {
    splitChunks: {
      // 自动提取所有公共模块到单独 bundle
      chunks: 'all'
    }
  },
```

chunks参数说明
    * asnyc 异步引入的库进行分离（默认）
    * inistial 同步引入的库进行分离
    * all 所有引入的库进行分离（推荐）

### tree shaking

#### 使用

一个模块可能有多个方法，只要其中每个方法用到了，那么整个文件都会被搭载bundle里，tree shaking就是只把用到的方法打入bundle，没用到的方法会在uglify阶段被擦除

> 要清除的就是 Dead Code，不止是引用的文件，也是各种代码片段

* webpack默认之处，在`.babelrc`里设置`modules: false`即可
* `production mode`下默认开启
* 必须是ES6的语法

#### 原理

> 利用es6模块的特点

* 只能作为模块顶层的语句出现
* import的模块名只能是字符串变量
* import binding 是immutable的

如果要在Webpack中模拟tree-shaking的操作，可以这样设置

```js
optimization: {
    // 使用该选项之后，Webpack打包的文件里该模块就只导出被使用的成员
    usedExports: true,
    // 压缩输出结果，清除掉未被使用的成员
    // minimize: true
  }
```

其中usedExports也可以理解为标记处需要被清除掉的部分，然后minimize就是负责摇掉这些部分的操作

#### tree-shaking和babel

理论上babel和tree-shaking在一起会失效，因为tree-shaking的基础就是基于ES6的模块机制。当然现在babel-loader已经默认关闭了module的转化机制

```js
module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              // 如果 Babel 加载模块时已经转换了 ESM，则会导致 Tree Shaking 失效
              // ['@babel/preset-env', { modules: 'commonjs' }]
              // ['@babel/preset-env', { modules: false }]
              // 也可以使用默认配置，也就是 auto，这样 babel-loader 会自动关闭 ESM 转换
              ['@babel/preset-env', { modules: 'auto' }]
            ]
          }
        }
      }
    ]
  }
```

#### SideEffects

tree-shaking的过程可以帮助我们顾虑掉模块里未被引用的方法，sideEffects则可以更进一步提升tree-shaking的效率

SideEffects的意思是标注某个模块是否具有副作用，如果没有副作用，那么Webpack将移除其全部的未被引用到的导出

我们首先在Webpack的配置中加入SideEffects的判断，表示Webpack在打包时，会考虑到SideEffect，那么它以什么为参考呢，那就是在当前项目的`package.json`里的sideEffects，如果该值为false，那么Webpack就会认为当前项目中用到的模块都没有副作用，也就不会全量打包了

常见的副作用比如某个模块里是各种原型的拓展，它当然没有导出，但是这个模块却不能够被shake掉。除此之外，CSS文件也是有副作用的文件

解决办法是手动指定有副作用的模块，那么其他的模块就会被shake下没有被引用的部分
 
```js
"sideEffects": [
    "./src/extend.js",
    "*.css"
  ]
```
  
### scope hoisting

随着引入模块的增加，大量函数闭包包裹代码，导致体积增大。同时，运行代码时创建的函数作用域变多，内存开销变大

#### 原理

将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突。通过scope hoisting 可以减少函数声明代码和内存开销

### 代码分割和动态import

#### 代码分割的意义

对应大的web应用来讲，将所有的代码都放在一个文件中是显然不够有效的，特别是你的代码是**在一些特殊情况下才会被用到**，webpack有一个功能就是将你的代码库分割成chunks，当代码运行到需要的时候再加载

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

```js
if (hash === '#posts') {
    // mainElement.appendChild(posts())
    import(/* webpackChunkName: 'components' */'./posts/posts').then(({ default: posts }) => {
      mainElement.appendChild(posts())
    })
  } else if (hash === '#album') {
    // mainElement.appendChild(album())
    import(/* webpackChunkName: 'components' */'./album/album').then(({ default: album }) => {
      mainElement.appendChild(album())
    })
  }
```

### webpack+eslint

* 不重复造轮子，基于eslint:recommend配置并改进
* 能够帮助发现代码错误的规则，全部开启
* 能够帮助保持团队的代码风格统一，而不是限制开发体验 

### webpack打包基础库和组件

* 打包压缩版和非压缩版
* 支持AMD/ejs/cjs的版本

其实打包库更推荐rollup

### webpack实现ssr打包

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

### 优化构建时命令行的显示日志

#### 方法1 原生方法：设置stats

| errors-only | 只在错误时输出 |
| --- | --- |
| minimal | 只在错误或者有新的编译时输出 |
| none | 没有输出 |
| normal | 标准输出 |
| verbose | 全部输出 |

#### 插件 friendly-errors-webpack-plugin

```
npm i -D friendly-errors-webpack-plugin
```

### 构建异常和中断处理

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

### 文件指纹

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

## Webpack原理

> 本质上，Webpack是一个现代的JS应用程序的静态模块打包器。当Webpack处理应用程序时，它会递归地构建一个依赖关系图，其中包含应用程序需要的每一个模块，然后将所有这些模块都打包成一个或者多个bundle

## 编写loader和插件

### 编写一个Loader

一个Loader就是一个转化器，它接受数据的原始格式，输出被处理后的格式，需要注意的是，在Loader调用链中，最终能够被Webpack认可的一定是JS/JSON格式的数据

```js
const marked = require('marked')

module.exports = source => {
  // console.log(source)
  // return 'console.log("hello ~")'
  const html = marked(source)
  // return html
  // return `module.exports = "${html}"`
  // return `export default ${JSON.stringify(html)}`

  // 返回 html 字符串交给下一个 loader 处理
  return html
}
```

### 编写一个插件

Webpack中的插件是通过在生命周期的钩子中挂在函数来实现扩展的

```js
class MyPlugin {
  apply (compiler) {
    console.log('MyPlugin 启动')

    compiler.hooks.emit.tap('MyPlugin', compilation => {
      // compilation => 可以理解为此次打包的上下文
      for (const name in compilation.assets) {
        // console.log(name)
        // console.log(compilation.assets[name].source())
        if (name.endsWith('.js')) {
          const contents = compilation.assets[name].source()
          const withoutComments = contents.replace(/\/\*\*+\*\//g, '')
          compilation.assets[name] = {
            source: () => withoutComments,
            size: () => withoutComments.length
          }
        }
      }
    })
  }
}
```
