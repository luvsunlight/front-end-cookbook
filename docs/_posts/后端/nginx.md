Nginx是一个高效的HTTP和反向代理的web服务器

## 安装

这里分享Mac下安装Nginx的过程

> 安装nginx

```js
brew install nginx
```

> 测试安装是否成功

```js
nginx -v // nginx version: nginx/1.19.6
```

## nginx指令

> 启动nginx服务（默认在8080端口,可以通过修改nginx）

```js
nginx
```

默认是没有反馈的，如果8080端口被占用了就清除掉该端口，访问localhost:8080就可以看到nginx的服务了

```
nginx -s quit  #快速停止nginx
nginx -V #查看版本，以及配置文件地址
nginx -v #查看版本
nginx -s reload|reopen|stop|quit   #重新加载配置|重启|快速停止|安全关闭nginx
nginx -h #帮助
```

> 修改nginx文件配置

如果我们想让nginx服务器正常代理我们的文件，就需要修改相应的nginx配置，该配置文件位于`/usr/local/etc/nginx/nginx.conf`

## 常见的问题记录

### nginx: [error] open() "/usr/local/var/run/nginx.pid" failed (2: No such file or directory)

nginx -s reload时可能会报上述错误

> 原因

没有nginx.pid 这个文件，每次当我们停止nginx时(nginx -s stop) ,nginx 会把 /usr/local/var/run/ 路径下名为nginx.pid 的文件删掉

> 解决方法

```
nginx -c /usr/local/etc/nginx/nginx.conf
nginx -s reload
```

### nginx 403

原因是Mac下需要相应的权限

在配置文件中加入

```
user root owner;
```

然后`sudo nginx -s reload`

### nginx配置历史模式

在配置文件中加入

```
location / {
            root   ...
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }
```
