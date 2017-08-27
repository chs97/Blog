---
title: 微信小程序request headers
date: 2017-01-29 16:46
tags:
  - 微信小程序
categories: 前端
---

## 微信小程序 爬坑记录 request-header

---

Labrador 组件 封装了 小程序原版的request。

<!--more-->

源码：
```javascript
/**
 * @copyright Maichong Software Ltd. 2016 http://maichong.it
 * @date 2016-11-19
 * @author Liang <liang@maichong.it>
 */

import wx from 'labrador';
import stringify from 'qs/lib/stringify';

/**
 * 默认获取SessionID方法
 * @returns {string}
 */
function defaultGetSession() {
  return wx.app.sessionId;
}

/**
 * 默认设置SessionID方法
 * @param {string} sessionId
 */
function defaultSetSession(sessionId) {
  wx.app.sessionId = sessionId;
}

// 有效HTTP方法列表
const methods = ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS', 'HEAD', 'TRACE', 'CONNECT'];

/**
 * 创建API Request客户端
 * @param {Object} options 选项
 * @returns {Function}
 */
export function create(options) {
  options = options || {};

  /**
   * 通用Alaska RESTFUL风格API请求,如果alaska接口返回错误,则抛出异常
   * @param {string} [method] 请求方法,可选默认GET,有效值：OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT
   * @param {string} apiName  API名称,必选
   * @param {object} [data]   数据,可选,如果方法为GET或DELETE,则此对象中的所有数据将传入URL query
   * @param {object} [header] HTTP头对象,可选
   * @returns {*}
   */
  async function request(method, apiName, data, header) {
    const apiRoot = options.apiRoot || {};
    const updateKey = options.updateKey || '_session';
    const headerKey = options.headerKey || 'Session';
    const getSession = options.getSession || defaultGetSession;
    const setSession = options.setSession || defaultSetSession;
    const defaultHeader = options.defaultHeader;

    if (methods.indexOf(method) === -1) {
      header = data;
      data = apiName;
      apiName = method;
      method = 'GET';
    }

    let url = apiRoot + apiName;
    // 对非POST 和 PUT 的data进行处理
    if (['POST', 'PUT'].indexOf(method) === -1 && data) {
      let querystring = stringify(data);
      if (url.indexOf('?') > -1) {
        url += '&' + querystring;
      } else {
        url += '?' + querystring;
      }
      data = undefined;
    }

    // 设置默认header
    header = Object.assign({}, defaultHeader, header);

    if (options.session !== false) {
      let sessionId = getSession();
      if (sessionId) {
        header[headerKey] = sessionId;
      }
    }
    // 调用微信的wx.request
    let res = await wx.request({
      method,
      url,
      data,
      header
    });


    if (options.session !== false && res.data && res.data[updateKey]) {
      if (res.data && res.data[updateKey]) {
        setSession(res.data[updateKey]);
      }
    }

    if (res.data && res.data.error) {
      throw new Error(res.data.error);
    }

    return res.data;
  }

  methods.forEach((method) => {
    request[method.toLowerCase()] = function (...args) {
      return request(method, ...args);
    };
  });

  request.setOptions = function (newOptions) {
    options = newOptions || {};
  };

  return request;
}

/**
 * 导出默认API客户端
 */
export default create({
  apiRoot: typeof API_ROOT === 'undefined' ? '' : API_ROOT
});
```
接下来就是一个大坑：

朋神写了一个demo给我。

![](http://od690gqhu.bkt.clouddn.com/2017129163652.png)

需要set header

源码里面的API是这样的  request(menthod, url, data, header);

然后我试了一下。没有报错。。network也没有显示发送任何东西。第一感觉是API有问题

尝试 request.post(url, data, header)一样

request.setOptions({defaultHeader}) 也一样

但是朋神说他demo测试没问题。。

然后尝试着把header删掉。。可以发送请求，加上header就不行。

尝试把header的数据全部换成 '1' 也可以

得出结论 是header的问题

然后删掉header有一部分很长的

```
let res = yield wx.login();
let data = yield wx.getUserInfo();
let Header = {
  'X-Wechat-Code': res.code,
  'X-Wechat-Data': data.rawData,
  'X-Wechat-Signature': data.signature
};
```
即 data.rawData

正常可以发送请求，加上就不行。。

那就是这个的锅了。截图给朋神看，朋神说 加个encodeURIComponent试试。。成功发送请求。
代码如下：
```
let res = yield wx.login();
let data = yield wx.getUserInfo();
let Header = {
  'X-Wechat-Code': res.code,
  'X-Wechat-Data': encodeURIComponent(data.rawData),
  'X-Wechat-Signature': data.signature
};
```

总结：header的这个字段有特殊字符和一些中文。所以要转义一下。。
