---
title: CORS 跨域问题
date: 2016-10-24 21:43
tags:
  - JavaScript
  - Vue.js
categories: 前端
---

## 解决跨域问题

---

1.同源策略

![选区_001.png][1]

> 第一，如果是协议和端口造成的跨域问题“前台”是无能为力的，<br>第二：在跨域问题上，域仅仅是通过“URL 的首部”来识别而不会去尝试判断相同的 ip 地址对应着两个域或两个域是否在同一个 ip 上。<br>
> “URL 的首部”指 window.location.protocol +window.location.host，也可以理解为“Domains, protocols and ports must match”。

这里提供了一些解决方法

[JChen 博客](http://www.cnblogs.com/JChen666/p/3399951.html)

在我自己开发的项目中，由于是前后端分离的 API 在 weafung 的服务器上，本地调试的时候就会产生跨域问题(vue-cli 脚手架 载体是 express)
在本地上 浏览器的 域名是 localhost:8081 端口和域名都不一样 就产生了跨域问题。本地的解决办法是 由于 express 使用了一个叫[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)的中间件 这个中间件是 node 的代理模块。如果是使用 vue-cli 脚手架的话，只要在 config/index.js 中

<!--more-->

```javaScript
dev: {
  env: require('./dev.env'),
  port: 8081,
  assetsSubDirectory: 'static',
  assetsPublicPath: '/',
  proxyTable: {
      '/index.php': {
          target: 'http://api.example.com',
          changeOrigin: true,
          pathRewrite: {
              '^/index.php': '/index.php'
          }
      }
  },
```

这样之后就把所有 http://localhost:8081/index.php/的API转成 http://api.example.com/index.php/ 跨域问题就解决了。

那么在 website 的服务器上，我使用的是 SSL，所以跨域的原因是 不同域名不同协议。web 服务使用的 nginx 只要在 conf 文件加上

```
location ^~ /index.php/ {
    proxy_pass http://api.example.com
}
```

这样也就解决的跨域问题。原理大概都是一样的。

反向代理 反向代理（Reverse Proxy）方式是指以代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

相当于把服务器端的 API 代理到本地服务器上，然后 website 使用的是本地的 API 这样就解决掉了 跨域的问题。。。

以上观点仅供参考。。

[1]: https://www.hs97.cn/usr/uploads/2016/10/1494334995.png
