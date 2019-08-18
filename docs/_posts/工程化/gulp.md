## About gulp

> [gulp](https://www.gulpjs.com.cn/)是一种自动构建工具，说白了就是一个小弟，配置好之后在命令行上输一个指令就可以让它完成诸如压缩文件，babel编译，less编译等工作，当然它能完成的绝不止这些

:::tip
gulp的优点在于代码简介明了，一个从来没有接触过构建工具的人只要稍微懂点编程就能看明白gulp的代码流程。但是缺点也很明显，那就是gulp的中文资料不是很多
:::

## Quick Start

### 1. 安装node

```
node -v // 查看node的版本，如果不知道node怎么安装自己找教程
```

### 2. 全局安装gulp

```
npm install --global gulp
```

### 3.  在项目的依赖中安装

```
npm install --save-dev gulp gulp-uglify gulp-babel gulp-rename
```

### 4. 在项目的根目录下新建`gulpfile.js`，这就是我们gulp的配置文件

```
#gulpfile.js

gulp.task('default',function(){
	console.log('run gulp task...')
})
```

### 5. 在根目录下命令行输入`gulp`即可

## 常用功能

首先来看这样一段代码，这是我在写一个插件时用到的gulp流

```js
// gulpfile.js

var gulp = require("gulp"),
	minifycss = require("gulp-minify-css"),
	rename = require("gulp-rename"),
	uglify = require("gulp-uglify"),
	less = require("gulp-less"),
	sourcemap = require("gulp-sourcemaps"),
	babel = require("gulp-babel"),
	babelenv = require("@babel/preset-env")

gulp.task("less", () => {
	gulp.src("docs/src/style.less")
		.pipe(less())
		.pipe(minifycss())
		.pipe(rename({ suffix: ".min" }))
		.pipe(gulp.dest("docs/src"))
		.pipe(gulp.dest("dist"))
})

gulp.task("minifyjs", () =>
	gulp
		.src("docs/src/index.js")
		.pipe(gulp.dest("dist"))
		.pipe(sourcemap.init())
		.pipe(
			babel({
				presets: [babelenv]
			})
		)
		.pipe(uglify())
		.pipe(rename({ suffix: ".min" }))
		.pipe(sourcemap.write("../maps"))
		.pipe(gulp.dest("docs/src"))
		.pipe(gulp.dest("dist"))
)

gulp.task("watch", function() {
	gulp.watch("docs/src/*.js", ["minifyjs"])
	gulp.watch("docs/src/*.less", ["less"])
})

gulp.task("default", ["less", "minifyjs"])

```

### 1. gulp的机制

> 常用的gulp插件库

```
npm i gulp gulp-babel gulp-less gulp-concat gulp-less gulp-minify-css gulp-rename gulp-sourcemaps gulp-uglify @babel/core @babel/preset-env -D
```

`gulp`的基本单位是task，可以从代码中明显地看出来gulpfile文件基本上就是在写各种task。上面的less和minifyjs可以看做是`业务型task`，而最下面的default task则是我们在根目录下运行gulp时触发的task

### 2. gulp运行任务
可以直接`gulp`来执行编写好的default，也可以通过`gulp taskName`的方式来执行某一种任务

### 3. gulp监听任务
有一些任务是需要动态进行的，比如less转css，coffeescript转js，不可能说我们每次修改完源文件就执行一次命令行，这样做我们自己就首先烦死了。我们可以使用gulp的`watch`功能来监听文件，文件产生变化就做出相应的改变

```js
gulp.task("watch", function() {
	gulp.watch("docs/src/*.js", ["minifyjs"])
	gulp.watch("docs/src/*.less", ["less"])
})

```

### 4. 压缩css

```
npm i gulp-minify-css -D
```

```js
gulp.task("minifycss", () => {
    gulp.src("src/style/*.css")
        .pipe(minifycss())
        .pipe(rename({ suffix: ".min" }))
        .pipe(gulp.dest("dist/css"))
})
```

首先通过`src`获取文件源，然后进行`minifycss`操作，接着通过`rename`修改文件名后缀，最后`gulp.dest`指定输出压缩文件的位置

### 5. less编译

```
npm i gulp-less -D
```

```js
gulp.task("less", () => {
    gulp.src("src/style/*.less")
        .pipe(less())
        .pipe(gulp.dest("dist/css"))
```

### 6. 压缩js

```
npm i gulp-uglify -D
```

```js
gulp.task("minifyjs", () => {
    gulp.src("src/style/*.js")
        .pipe(uglify())
        .pipe(rename({ suffix: ".min" }))
        .pipe(gulp.dest("dist/css"))
```


### 7. es6编译

这个比较麻烦，需要首先安装一些包

```
npm i @babel/core @babel/preset-env gulp-babel -D
```

然后在gulpfile文件开头引入

```js
var babel = require("gulp-babel"),
	babelenv = require("@babel/preset-env")
```

最后在处理js时，加入如下代码

```js
gulp.task("minifyjs", () =>
	gulp
		.src("docs/src/index.js")
		.pipe(
			babel({
				presets: [babelenv]
			})
		)
		.pipe(uglify())
		.pipe(rename({ suffix: ".min" }))
		.pipe(gulp.dest("dist"))
```

### 8. 拼接文件

为了减少网络请求量，很多情况下我们需要将多个文件合并，我们就需要concat

```
npm i gulp-concat -D
```

```js
gulp.task("minifyjs", () =>
	gulp
		.src("docs/src/*.js")
		.pipe(concat('all.js'))
		.pipe(uglify())
		.pipe(rename({ suffix: ".min" }))
		.pipe(gulp.dest("dist"))
```

### 9. sourcemap
处于种种原因，我们在项目中引进的文件可能是经过压缩或者编译过的，这给我们的调试带来了巨大的麻烦，这里我们需要引入sourcemap

```
npm i gulp-sourcemaps -D
```

```js
gulp.task("minifyjs", () =>
	gulp
		.src("docs/src/index.js")
		.pipe(sourcemap.init())
		.pipe(
			babel({
				presets: [babelenv]
			})
		)
		.pipe(uglify())
		.pipe(rename({ suffix: ".min" }))
		.pipe(sourcemap.write("../maps"))
		.pipe(gulp.dest("dist"))
```

:::tip
注意，sourcemap有两步，第一个是在编译或者压缩前开启sourcemap，第二个是在输出文件前将sourcemap进行write记录
:::

## API
> 直接去翻阅[官网](https://www.gulpjs.com.cn/docs/api/)

## 使用技巧

> [技巧汇总](https://www.gulpjs.com.cn/docs/recipes/)
