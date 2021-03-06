# CORS

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

它允许浏览器向跨源服务器，发出[`XMLHttpRequest`](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)请求，从而克服了AJAX只能[同源](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)使用的限制。

## 简介

整个CORS通信过程，都是浏览器自动完成，浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

## 两种请求

浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

**简单请求**满足条件：

1. 请求方法是以下几种方法之一：
   * GET
   * POST
   * HEAD
2. HTTP的头信息不超出以下几种字段：
   * Accept
   * Accept-Language
   * Content-Language
   * Last-Event-ID
   * Content-Type：只限于三个值`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

凡不满足上述条件的都属于**不简单请求**

## 简单请求

浏览器直接发出CORS请求，即在头部区域增加一个**Origin**字段

**Origin**: 用来说明本次请求来自哪个源（协议 + 域名 + 端口）

### 服务器判断

* 不在许可范围内

  浏览器检测响应头没有`Access-Control-Allow-Origin`字段便会报错，该错误无法使用状态码识别

* 在许可范围内

  服务器响应会多出几个头信息字段：

  * **Access-Control-Allow-Origin**

    该字段是必须的。它的值要么是请求时`Origin`字段的值，要么是一个`*`，表示接受任意域名的请求。

  * **Access-Control-Allow-Credentials**

    该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。

  * **Access-Control-Expose-Headers**

    CORS请求时，`XMLHttpRequest`对象的`getResponseHeader()`方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在`Access-Control-Expose-Headers`里面指定。

### withCredentials 属性

开发者必须在AJAX请求中打开`withCredentials`属性,否则即使服务器同意请求可以携带Cookie，浏览器也不会携带Cookie

注意：如果要发送Cookie，`Access-Control-Allow-Origin`就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的`document.cookie`也无法读取服务器域名下的Cookie。

## 非简单请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是`PUT`或`DELETE`，或者`Content-Type`字段的类型是`application/json`。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为**"预检"请求**（preflight）。

### 预检请求头信息

```http
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```

* 预检请求的方法为**OPTIONS**

* **Access-Control-Request-Method**

  用来列出浏览器的CORS请求会用到哪些HTTP方法

* **Access-Control-Request-Headers**

### 预检请求的响应

```http
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```

**失败**

如果服务器否定了"预检"请求，会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被`XMLHttpRequest`对象的`onerror`回调函数捕获。

**成功**

```http
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

* **Access-Control-Allow-Origin**

  表示服务器允许该源的请求

* **Access-Control-Allow-Methods**

  表明服务器支持的所有跨域请求的方法。

* **Access-Control-Allow-Headers**

  表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

* **Access-Control-Allow-Credentials**

  同简单请求

* **Access-Control-Max-Age**

  用来指定本次预检请求的有效期，单位为秒。在有效期间内浏览器不用发出另一条预检请求。