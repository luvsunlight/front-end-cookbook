## 简介

> PWA（Progressive web apps，渐进式Web应用），运用现代的WebAPI和传统的渐进式增强策略来创建跨平台Web应用程序

### NativeApp的缺点

* 开发成本高（ios & android）
* 软件上线需要审核
* 版本更新需要将新版本上传至不同的应用商店
* 必须下载才可以使用app

### 传统webApp的缺点

* 手机入口不够边界
* 没有网络即无响应
* 不能像app一样进行消息推送

### PWA的主要特点

* 可靠 - 可离线加载，在不稳定的网络环境下，也能瞬间加载并展现
* 体验 - 快速响应，拥有平滑的动画响应用户的体验
* 黏性 - 类似native app，具有沉浸式的
* 及时 - 实现了消息推送功能
* 渐进性 - 适用于所有浏览器
* 持续更新 - 始终是最新版本的应用

> PWA强调渐进式，并不要求一次达到安全，性能和体验上的所有要求

## 渐进式开发

### PWA改造

* 第一步是安全。全站https化，这也是PWA的基础，没有它就没有service worker
* 第二步是service worker，来提升基础性能，离线提供静态文件，把用户的首屏体验提升上来
* 第三部，App manifest
* 最后，考虑其他特性，比如

## 技术指南

### Service worker

## Refer

* [MDN PWA](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps)
* [LAVAS PWA](https://lavas.baidu.com/pwa)
* [Segmaentfault 讲讲PWA](https://segmentfault.com/a/1190000012353473)
* [PWA 电子书](https://lavas-project.github.io/pwa-book/chapter01/2-what-is-pwa.html)