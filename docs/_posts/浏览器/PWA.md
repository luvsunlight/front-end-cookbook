## 简介

> PWA（Progressive web apps，渐进式Web应用），运用现代的WebAPI和传统的渐进式增强策略来创建跨平台Web应用程序

传统WebAPP的缺点

* 缺少应用一级入口
* 缺少服务端推送
* 没有离线缓存能力

### PWA的主要特点

> PWA是一套理念，渐进式增强Web的优势，并通过技术手段缩短和本地应用或者小程序的距离，强调渐进式，并不要求一次达到安全，性能和体验上的所有要求

针对Web的缺陷，PWA提出了两种解决方案，引入了ServiceWorker来解决离线存储和消息推送的问题，引入manifest.json来解决一级入口的问题

* 可靠 - 可离线加载，在不稳定的网络环境下，也能瞬间加载并展现
* 体验 - 快速响应，拥有平滑的动画响应用户的体验
* 黏性 - 类似native app，具有沉浸式的体验
* 及时 - 实现了消息推送功能
* 渐进性 - 让Web应用能逐步具有本地应用的能力，降低站点改造的代价，使得站点逐步支持各项新技术，而不是一步到位
* 持续更新 - 始终是最新版本的应用

## Service worker

SW的核心概念就是在WebAPP外部设置了一个HTTP拦截层，它可以拦截所有来自WebAPP的HTTP请求，并且内部有自己的逻辑处理

有一个额外的HTTP拦截层，首先解决了`网页离线使用能力`，意味着资源的缓存不再受到服务端的限制（不需要协商缓存）

其次，SW的内部可以使用WebWorker来实现一个与页面渲染线程不冲突的JS线程，除了不能访问DOM，其余的JS操作都可以进行

最后还解决了`消息推送`，在WebAPP中注册SW，并且订阅服务端的消息，服务端就可以单向和SW层推送消息，然后SW层接收到消息之后再将网络请求传给WebAPP（以消息通知的形式）

## PWA的缺点

之所以还没有大面积推行，主要还是兼容的问题，而且该标准还没有被社区承认，此外还有比如flutter，rn小程序这样的解决方案的竞争对手

## Refer

* [MDN PWA](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps)
* [LAVAS PWA](https://lavas.baidu.com/pwa)
* [Segmaentfault 讲讲PWA](https://segmentfault.com/a/1190000012353473)
* [PWA 电子书](https://lavas-project.github.io/pwa-book/chapter01/2-what-is-pwa.html)