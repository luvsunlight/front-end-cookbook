## 简介

> PWA（Progressive web apps，渐进式Web应用），运用现代的WebAPI和传统的渐进式增强策略来创建跨平台Web应用程序

### NativeApp的缺点

* 开发成本高（ios & android）
* 软件上线需要审核
* 版本更新需要将新版本上传至不同的应用商店
* 必须下载才可以使用app

### 传统webApp的缺点

* 缺少离线使用能力
* 缺少消息推送的能力
* Web应用缺少一级入口

### PWA的主要特点

> PWA是一套理念，渐进式增强Web的优势，并通过技术手段缩短和本地应用或者小程序的距离

针对Web的缺陷，PWA提出了两种解决方案，引入了ServiceWorker来解决离线存储和消息推送的问题，引入manifest.json来解决一级入口的问题

* 可靠 - 可离线加载，在不稳定的网络环境下，也能瞬间加载并展现
* 体验 - 快速响应，拥有平滑的动画响应用户的体验
* 黏性 - 类似native app，具有沉浸式的体验
* 及时 - 实现了消息推送功能
* 渐进性 - 让Web应用能逐步具有本地应用的能力，降低站点改造的代价，使得站点逐步支持各项新技术，而不是一步到位
* 持续更新 - 始终是最新版本的应用

> PWA强调渐进式，并不要求一次达到安全，性能和体验上的所有要求

## 渐进式开发

### PWA改造

* 第一步是安全。全站https化，这也是PWA的基础，没有它就没有service worker
* 第二步是service worker，来提升基础性能，离线提供静态文件，把用户的首屏体验提升上来
* 第三部，App manifest
* 最后，考虑其他特性

## 安全

* HTTPS化
* Web页面默认的安全策略
* 储入同源策略
* 内容安全策略CSP

## Service worker

### 架构

Service Worker概念主要思想是`在页面和网络之间增加一个拦截器`，用来缓存和拦截请求。WebApp安装了SW模块后，在请求资源时，会先通过SW，让它哦按段是返回SW缓存的资源还是去重新向网络请求资源

我们已经知道了WebWorker是一个可以在渲染进程中主线程以外单独开启的线程，不可以使用DOM和CSSOM的API，适合用于处理耗时的与DOM无关的任务。核心思想是`运行在主线程之外`

SW是运行在浏览器进程中的，因为其生命周期最长

### 消息推送

消息推送也是基于SW实现的，因为消息推送时，浏览器页面可能并没有启动，这个时候需要SW来接受服务器推送的消息

## Refer

* [MDN PWA](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps)
* [LAVAS PWA](https://lavas.baidu.com/pwa)
* [Segmaentfault 讲讲PWA](https://segmentfault.com/a/1190000012353473)
* [PWA 电子书](https://lavas-project.github.io/pwa-book/chapter01/2-what-is-pwa.html)