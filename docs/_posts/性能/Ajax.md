## 1. 数据传输

## 2. 数据格式

### 2.1 JSONP

jsonp的原理是什么，我们都知道我们在网页里请求远程资源会受到同源策略的限制

但是我们首先发现script标签的url设置为远程连接则可以绕开这个限制，比如我们平时用的CDN根本不存在限制，我们就想到了用script标签来获取数据

1.我们在本地设置好想要从远程服务器获得的回调函数，拟定为callback,并且在本地就已经假定callback会传回来的参数，如下

```
// local
function callback(data) {
    process(data)
}
```

2.将回调函数的名字传给服务器，同时以script的形式向服务器请求

```
// local
const url = "remote.com/getData?callback=callback"
let script = document.createElement('script')
script.src = url
document.getElementByTagName('head')[0].appendChild(script)
```

3.服务器端根据传过来的回调参数名称，返回对应的回调函数

```
// remote
callback([
    {
        name: "jack",
        age: 11
    },
    ...
])
```

JSONP一个是可以绕开浏览器同源限制，另外一个是解析数据非常快，因为它本来就是js类型的数据，而不像json还需要进行反序列化等操