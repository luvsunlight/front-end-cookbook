## 应用

### 定义和作用

VueRouter是专门为Vue.js设计的一个路由管理器，我们可以将组建和路由构建映射，然后通过变化路由来实现页面渲染不同的组件

我们首先在router中先Vue.use(VueRouter),这个过程和Vuex是一样的，本质上是调用其内部的install方法，然后通过applyMixin，在beforeCreate钩子里注入router对象。然后是定义route（即组件和路由的映射关系），创建Router实例对象，最后在newVue中注入该router对象。此外我们还需要在HTML或者模板里加入router-link元素，用来做路由渲染的容器

有区别的是这次是注入两个对象，一个是router即路由管理器，它有方法，比如push方法，另一个是route，它则是目前路由的信息，不包含方法

VueRouter包含一些方法和特性

* 动态路由匹配
    * 路由加参数
    * 匹配通配符（正则）
    * props（将路由的参数变成组件的props，目的是减少组件和路由的强依赖，实现方法是在路由规则里配置props属性）
* 路由嵌套
* 编程式的导航，即我们的router对象
    * push
    * replace（不会被添加到history中）
    * go
* 命名路由
* 命名视图。通过给router-link命名，我们可以指定某个路由下哪个视图渲染哪个组件
* 重定向和alias
* 路由组件传参。比如我们在组件里使用$route.params.id，这样做会让组件和路由耦合在一起，我们可以先设置组件的prop为id，然后在路由设置中加入`prop:true`,这样路由参数的传入就是以prop而不是通过route对象的属性
* history模式。hash模式指浏览器的路由采用当前url的hash值，看起来非常丑
* 导航守卫（NProgress），可以注册全局的，也可以注册单个路由的
* 路由元信息（官方给出来的示例是用来鉴权的，比如在某个路由下载meta中注明要激活该组件需要登录状态，这么做比是减少了路由守卫和鉴权判断的的耦合性）
* 路由懒加载

## 原理

### 前端路由中的History模式和Hash模式

首先要了解，这两个技术手段都是前端路由的实现方式，即尽可能地在不请求服务器新资源（请求新HTML的话页面就会强制刷新了）的情况下，通过路由的改变而改变页面渲染的内容

Hash模式要用到的是BOM对象document.location里的hash值，它表现为`#hashValue`，其原本的作用是页面定位，比如我们将hash值设为#app，那么id为app的DOM元素就会快速出现在页面可视范围内。那么它和前端路由有什么关系呢？

浏览器事件里没有监听url变化的函数（因为正常情况下url变化浏览器会开始缓存/网络请求），但是对于hash值有一个onhashchange方法可以监听其变化，而且hash值和url还是解耦的,我们可以通过修改hash值来修改路由，同时onhashchange函数还可以监听其变化。

为什么hash模式下的url会很丑陋呢？我觉得一个是为了保密，比如url的参数里有很重要的信息，不能直接出现在url中，另外一个是为了缩短长度，如果路由很长或者参数很多，那么hash值就会很长，会导致很多麻烦

history模式则是H5之后新增的，它拥有go，back，forward方法，即控制浏览器的前进后退，最重要的是它拥有`pushState`和`replaceState`方法（push是新建一个状态，replace是修改当前状态），这两个方法可以直接修改当前url但是不触发请求，此外还可以修改state。浏览器的浏览记录是以栈的形式存储的，state就是栈存储的浏览器浏览状态，每一个状态包含url和一些存储信息，这些信息我们就可以拿来存储参数，可以说就是为了前端路由而生的操作。但是history模式有一个重大的问题，那就是不能刷新，像hash的变化是不会触发浏览器重新发起请求的，但是history模式是直接修改url，这就需要后端做出验证

hash模式
* 丑陋
* 支持性好

history模式
* 好看
* 需要**服务端支持**
* 参数更友好，hash参数有限
* 有一些情况必须要history模式，比如一些特殊的业务会对url做验证，删除里面的特殊字符，这个时候只能采取history模式

默认情况下，VueCLI启动的服务器已经兼容了History模式，对于Node搭建的服务器，可以如下配置来兼容History模式

```js
const path = require("path")
// 导入处理 history 模式的模块
const history = require("connect-history-api-fallback")
// 导入 express
const express = require("express")

const app = express()
// 注册处理 history 模式的中间件
// app.use(history())
// 处理静态资源的中间件，网站根目录 ../web
app.use(express.static(path.join(__dirname, "../web")))

// 开启服务器，端口是 3000
app.listen(3000, () => {
    console.log("服务器开启，端口：3000")
})
```

### 路由懒加载

原理使用到了动态import的技术，这是一个标准，但是目前还没有实现，可以通过babel插件或者相应的垫片来实现

动态import用法如下

```
import("./conf.js").then(...)
```

路由懒加载原理如下

```
#router.js

component: () => import('...')
```

## Vue-Router实现

### Vue-Router使用过程回顾

最基础的VueRouter的使用包含以下几个步骤

* `Vue.use`来注册路由组件
* 生成相应的**路由规则**
* 基于路由规则生成VueRouter对象
* 在Vue实例中注入VueRouter对象