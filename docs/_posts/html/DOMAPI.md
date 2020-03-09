## onclick & addEventListener

onclick

* 简单直接，一个函数代表DOM节点绑定的点击事件
* 因为它是直接和DOM节点绑定的，所以后定义的onclick事件会覆盖前面的
* 只支持冒泡触发，不支持捕获（这意味着子元素和父元素同时触发时先调用子元素的onclick事件）

addEventListener

* 本质上是一个策略模式，可以应对多种事件
* 一个节点可以绑定多个addEventListener
* 可以自己选择冒泡或者捕获