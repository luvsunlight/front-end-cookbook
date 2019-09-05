> npm 是一个 node 的包管理工具，可以说它是前端工程化中最核心的库之一。因为它不仅能够做到前端自动构建的任务，还能管理前端的库之间的依赖。npm 的包一旦发布之后，[unpkg](unpkg.com)也会自动添加 CDN,可以说是一个功能非常齐全的现代化工具

## Quick start

> 自行查找[网络教程](https://www.runoob.com/nodejs/nodejs-npm.html)，类似的 npm 起步教程非常多，这里不再赘述
>
> 注：重点关注`全局安装`和`局部安装`的区别，更换 npm 源

## 常用命令

### 查看版本

```
npm -v
```

### 升级 nodejs 至最新稳定版

```
n stable
```

### 使用淘宝源

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 设置 npm 初始设置

```
npm set init-author-name "<yourName>"
npm set init-author-email "<yourEmail>"
npm set init-author-url "https://github.com/<yourUserName>"
npm set init-license "MIT"
```

### 初始化 npm 环境

```
npm init
```

### 安装模块

```
npm i/install packageName

npm i/install packageName -g // 全局安装

npm i/install packageName -D // 开发依赖
```

全局安装就是安装在系统环境里，每一个项目都可以调用。但是一般我们不这么做，因为一个是可能会污染一些项目，还有一个可能是不同项目可能要用的依赖库版本不一样。开发依赖则是只在开发环境下会引用到的库，在生产环境中则被排除以尽可能减少产品体积，比如`gulp`，`webpack`等构件库

### 查看安装信息

```
npm list/ls // 查看当前项目下安装的模块
npm list/ls -g // 查看所有全局安装的模块
npm list/ls grunt // 查看某个模块的版本号
```

### 发布一个 npm 包

```
npm login

npm publish

npm unpublish packageName // 撤销发布
```

### 查看包的信息

```
npm info packageName
```

### 查看帮助

```
npm help
```

## package.json 属性说明

-   name - 包名

-   version - 包的版本号。如果是要发布的包的话，每个版本更迭 version 都要比之前高

-   description - 包的描述。

-   homepage - 包的官网 url 。

-   author - 包的作者姓名。

-   contributors - 包的其他贡献者姓名。

-   dependencies - 依赖包列表。如果依赖包没有安装，npm 会自动将依赖包安装在 node_module 目录下。

-   repository - 包代码存放的地方的类型，可以是 git 或 svn，git 可在 Github 上。

-   main - main 字段指定了程序的主入口文件，require('moduleName') 就会加载这个文件。这个字段的默认值是模块根目录下面的 index.js。

-   files - npm 包发布时上传至 npm 仓库里的文件有哪些

-   keywords - 关键字，如果你的包发布了，那么这些关键词就是你的包的描述

-   license - 包的版权

-   scripts - 一些命令行脚本，能够极大提高开发体验和效率

## npm 脚本命令

### 不知道有什么 npm 脚本？

```
npm run
```

### 通配符和转义

```
// *表示任意文件名，**表示任意一层子目录
"lint": "jshint *.js"
"lint": "jshint **/*.js"
// 将通配符传入原始命令，防止被 Shell 转义，要将星号转义
"test": "tap test/\*.js"
```

### 传参

向 npm 脚本传参时，要使用`--`标明

```
# package.json
"deploy": "gulp deploy",
# 命令行
$ npm run deploy -- --test
```

### 一个命令执行多个任务

```
npm run task1 & npm run task2 // 并行执行
npm run task1 && npm run task2 // 继发执行，即只有执行完第一个任务才能执行下一个
```

### 默认脚本

即不需要任何设置就会默认可以运行的脚本

```
"start": "node server.js"，
"install": "node-gyp rebuild"
```

### 钩子

npm 有`start`和`post`两个钩子

```
# package.json
"prebuild": "echo I run before the build script",
"build": "cross-env NODE_ENV=production webpack",
"postbuild": "echo I run after the build script",
# 命令行
$ npm run build
# 等同于执行
$ npm run prebuild && npm run build && npm run postbuild
```

:::tip
自定义的脚本命令也可以加上 pre 和 post 钩子。比如，myscript 这个脚本命令，也有 premyscript 和 postmyscript 钩子。不过，双重的 pre 和 post 无效，比如 prepretest 和 postposttest 是无效的
:::

### npm_lifecycle_event 变量 (返回当前正在运行的脚本名称,pretest、test、posttest)

```
#利用这个变量，在同一个脚本文件里面，为不同的npm scripts命令编写代码
const TARGET = process.env.npm_lifecycle_event;

if (TARGET === 'test') {
  console.log(`Running the test task!`);
}

if (TARGET === 'pretest') {
  console.log(`Running the pretest task!`);
}

if (TARGET === 'posttest') {
  console.log(`Running the posttest task!`);
}
```

### npm 的内部变量

> 通过 npm*package*前缀，npm 脚本可以拿到 package.json 里面的字段
> (如果是 Bash 脚本，可以用\$npm*package*前缀取值)

```
// package.json
{
  "name": "foo",
  "version": "1.2.5",
  "config" : { "port" : "8080" },
  "scripts": {
    "view": "node view.js",
    "start" : "node server.js"
  }
}

// view.js
console.log(process.env.npm_package_name); // foo
console.log(process.env.npm_package_version_view); // node view.js
```

npm 脚本还可以通过 npm*config*前缀，拿到 npm 的配置变量，即 npm config get xxx 命令返回的值。
比如，当前模块的发行标签，可以通过 npm_config_tag 取到。

### 常用脚本示例

```
// 删除目录
"clean": "rimraf dist/*",

// 本地搭建一个 HTTP 服务
"serve": "http-server -p 9090 dist/",

// 打开浏览器
"open:dev": "opener http://localhost:9090",

// 实时刷新
 "livereload": "live-reload --port 9091 dist/",

// 构建 HTML 文件
"build:html": "jade index.jade > dist/index.html",

// 只要 CSS 文件有变动，就重新执行构建
"watch:css": "watch 'npm run build:css' assets/styles/",

// 只要 HTML 文件有变动，就重新执行构建
"watch:html": "watch 'npm run build:html' assets/html",

// 部署到 Amazon S3
"deploy:prod": "s3-cli sync ./dist/ s3://example-com/prod-site/",

// 构建 favicon
"build:favicon": "node scripts/favicon.js",

```

## nvm

### nvm是什么

> 在我们的日常开发中经常会遇到这种情况：手上有好几个项目，每个项目的需求不同，进而不同项目必须依赖不同版的 NodeJS 运行环境。如果没有一个合适的工具，这个问题将非常棘手。

nvm 应运而生，nvm 是 Mac 下的 node 管理工具，有点类似管理 Ruby 的 rvm，如果需要管理 Windows 下的 node，官方推荐使用 nvmw 或 nvm-windows。不过，nvm-windows 并不是 nvm 的简单移植，他们也没有任何关系。但下面介绍的所有命令，都可以在 nvm-windows 中运行

### nvm的使用

```
nvm install 4.2.2 
```

nvm遵循语义化版本命名规则

```
nvm use 4.2.2
```

```
nvm ls // 列出已安装node
```


## nrm

### nrm是什么

nrm是一个npm源管理器，允许你快速在如下npm源之间切换

```
npm install -g nrm

```

```
nrm ls 
```

```
nrm use taobao
```

```
nrm add  <registry> <url> [home]
```

```
nrm del <registry>
```

## npx

参考[阮一峰 npx使用教程](http://www.ruanyifeng.com/blog/2019/02/npx.html)

### 使用

如果我们在一个项目本地安装的模块，比如`mocha`，我们想要调用它，只能在项目脚本或package.json的scripts里，如果想在命令行下调用，必须像这样

```
# 项目的根目录下执行
$ node-modules/.bin/mocha --version
```

npx就是想解决这个问题，让项目内部的模块用起来更方便，只需要这样调用即可

```
npx mocha --version
```

npx的原理很简单，就是运行的时候会到`node_modules/.bin`路径和环境变量`$PATH`里面检查命令是否存在

### 避免全局安装模块

我们可以将一些诸如`vue-cli`这样的全局模块下载到一个临时目录，使用后再删除

### --no-install 参数和--ignore-existing 参数

如果想让 npx 强制使用本地模块，不下载远程模块，可以使用--no-install参数。如果本地不存在该模块，就会报错。

```
$ npx --no-install http-server
```

反过来，如果忽略本地的同名模块，强制安装使用远程模块，可以使用--ignore-existing参数。比如，本地已经全局安装了create-react-app，但还是想使用远程模块，就用这个参数。