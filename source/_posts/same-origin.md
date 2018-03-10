---
title: 跨域与同源策略探究
date: 2018-03-10 11:16:53
tags: 
---



### 背景

跨域这个问题前端开发者都接触过，网上的文章也非常多，但是昨天的腾讯二面给我留了非常深刻的印象，原来跨域能问出那么多花样，难怪所有面试官都喜欢和面试者来探讨这个问题。

### 跨域

#### 一、什么是跨域

简单的来说就是浏览器限制了向不同域发送ajax请求。

不同域体现在：域名、端口、协议不同

<!--more-->

#### 二、怎么解决跨域

##### 1.JSONP

JSONP在CORS出现之前，是最常见的一种跨域方案，在IE10以下的版本，是不兼容CORS的，所以如果有需要兼容IE10一下的，都会使用JSONP去解决跨域问题。

###### JSONP的基本原理：

动态向页面添加一个**<script>**标签，浏览器对脚本请求是没有域限制的，浏览器会请求脚本，然后解析脚本，执行脚本。通过这个我们就可以实现跨域请求。

```javascript
  function handleResponse(response) {
    console.log(response)
  }
  var script = document.createElement("script")
  script.src = "http://localhost:3000/?callback=handleResponse"
  document.body.insertBefore(script, document.body.firstChild)
```

研究一下这段代码，新建一个script标签，设置标签的src属性，动态插入标签到body后面。插入以后浏览器会请求src的内容，下载下来并执行。

那怎么通过回调handleResponse获得数据？src后面那段querystring又是干什么的？

如果请求得到的脚本里面的代码长这样

```js
handleResponse('hello world')
```

执行的时候是不是就可以通过回调得到data -> 'hello world' 

所以jsonp其实也需要后端的支持，这个queryString就是让后端知道你前端的回调方法，然后要返回怎样的脚本给前端。

``` javascript
const koa = require('koa')
const app = new koa()
app.use(async ctx => {
  ctx.body = `${ctx.query.callback}('hello world')`;
});
app.listen(3000)
```

这是node的一段代码，简单的体现出了jsonp后端的处理方式。

###### JSONP的缺点

只能发送get请求，感觉这不是正经的手段，而是一个奇淫巧技。

##### 2.CORS

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

它允许浏览器向跨源服务器，发出[`XMLHttpRequest`](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)请求，从而克服了AJAX只能[同源](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)使用的限制。

###### 什么是CORS？

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

拿Koa举个栗子。在Koa中，我们只需要在Koa中加个中间件

```javascript
const koa = require('koa')
const cors = require('koa2-cors')
const app = new koa()
app.use(cors())
app.use(async ctx => {
  ctx.body = 'hello world';
});
app.listen(3000)
```

这样就能实现服务端CORS接口。

###### 两种请求

关于为什么有这两种请求的区别，我个人认为，浏览器发送预请求所消耗的资源会比简单请求还多，所以浏览器发送简单请求不需要发送预请求。而非简单请求，如果发送以后，然后被服务器拒绝了，所消耗的资源比预请求还多，所以在发送非简单请求之前，使用一个预请求来判断服务器是否允许跨域。

1.简单请求

同时满足一下条件为简单请求

（1) 请求方法是以下三种方法之一：

- HEAD
- GET
- POST

（2）HTTP的头信息不超出以下几种字段：

- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

如果content-type的值为`application/json`那么这个请求就是非简单请求，需要发送预请求。

2.非简单请求

不属于简单请求都是非简单请求，非简单请求就需要预请求。

###### 预请求

非简单请求会在正式请求之前发送一次预请求，这个请求是浏览器发的。浏览器先向服务器询问，当前的请求域是否在服务器的许可名单中，已经服务器允许哪一些方法，哪一些请求头。只有得到服务器的答复，并且之后发送的那个正式请求是被允许的（方法和请求头），浏览器才会发送这个正式请求。

"预检"请求用的请求方法是`OPTIONS`，表示这个请求是用来询问的。头信息里面，关键字段是`Origin`，表示请求来自哪个源。

除了`Origin`字段，"预检"请求的头信息包括两个特殊字段。

**Access-Control-Request-Method**

字面意思，内容是允许的请求方法

**Access-Control-Request-Headers**

同字面意思，内容是允许的额外的请求头

#####3.通过iframe

#### 三、为什么浏览器需要同源策略

考虑一下这个场景：你打开了A网站，并且登录了A网站，A网站也记录了你的cookie信息，然后你打开一个B网站，如果没有同源策略，B网站是可以直接请求A网站的接口的，有一些比如个人信息，他就可以通过get等方法，获得到你的信息，甚至可以post等操作去修改你的信息，这样你的账户安全是受到很严重的威胁的。所以浏览器需要同源策略来保证一定的安全，攻击手段例如CSRF和XSS，下面会详细讲。

#### 四、跨源网络访问

##### 1.跨域写操作

##### 2.跨域资源嵌入

### CSRF与XSS

#### 一、XSS

##### 1.XSS是什么

##### 2.怎么预防XSS

#### 二、CSRF

##### 1.CSRF是什么

##### 2.怎么预防

#### 参考链接

[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)

[浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

[HTTP访问控制（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)