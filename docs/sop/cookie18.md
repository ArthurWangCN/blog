---
title: cookie 18问
date: 2022-03-20 21:09:25
tags: [浏览器, JavaScript]
categories: 浏览器
description: 很多场景下，我们需要知道下一次的会话和上一次的会话的关系（比如登陆之后我们需要记住登陆状态），这个时候就引入了Cookie和Session体系。
---

# cookie 18问

我们知道，HTTP是一种无状态协议，无状态是指服务端对于客户端每次发送的请求都认为它是一个新的请求，上一次会话和下一次会话没有联系。

但是，很多场景下，我们需要知道下一次的会话和上一次的会话的关系（比如登陆之后我们需要记住登陆状态），这个时候就引入了Cookie和Session体系。

Cookie： 客户端请求服务器时，如果服务器需要记录该用户状态，就通过response Headers向客户端浏览器颁发一个Cookie，而客户端浏览器会把Cookie保存起来。当浏览器再请求服务器时，浏览器把请求的网址连同该Cookie一同提交给服务器。服务器通过检查该Cookie来获取用户状态。

Session： 当服务器接收到请求时，就从存储在服务器上的无数session信息中去查找客户端请求时带过来的cookie的状态。如果服务器中没有这条session信息则添加一条session信息。

通常Cookie中存的是session信息经过计算后的唯一Id（sessionId）。

## 1. cookie存储在哪里？

cookie一般是被浏览器以txt的形式存储在电脑硬盘中，供该浏览器进行读、写操作。


## 2. cookie是如何工作的？

- request：当浏览器发起一个请求时，浏览器会自动检查是否有相应的cookie，如果有则将cookie添加到Request Headers的Cookie字段中（cookie字段是很多name=value以分号分隔的字符串）。
- response：当服务端需要种cookie时，在http请求的Response Headers中添加Set-Cookie字段，浏览器接收到之后会自动解析识别，将cookie种下。


## 3. cookie和session的区别？

- 存储位置不同： cookie数据存放在客户的浏览器上，session数据放在服务器上。
- 存储大小不同： 一般单个cookie保存的数据不能超过4k, 单个域名最多保存30个cookie（不同浏览器有差异）；session则无大小和数量限制。


## 4. 什么是session级别的cookie？

session级别的cookie只针对当前会话存在，会话终止则cookie消失。

当cookie没有设置expires的时候，该cookie只会在网页会话期间存在，当浏览器退出的时候，该cookie就会消失。

浏览器中的表现为Expires/Max-Age的内容为Session。


## 5. cookie可以被谁来操作？

服务端和js都可以读/写cookie。


## 6. cookie各属性详解

- Name: cookie名
- Value: cookie值。
- Domain: cookie的域名。如果设成.example.com，那么子域名a.example.com和b.example.com，都可以使用.example.com的cookie;反之则不可以。
- Path: 允许读取cookie的url路径，一般设置为/。
- Expires： cookie过期时间。不设置，则为Session会话期，页面退出时cookie失效。
- HttpOnly: 设置为true时，只有http能读取。js只能读取未设置HttpOnly的cookie。
- Secure: 标记为Secure的cookie，只有https的请求可以携带。
- SameSite: 限制第三方url是否可以携带cookie。有3个值：Strict/Lax(默认)/None。（chrome51新增属性，chrome80+强制执行）
  - Strict: 仅允许发送同站点请求的的cookie
  - Lax: 允许部分第三方请求携带cookie，即导航到目标网址的get请求。包括超链接 ，预加载和get表单三种形式发送cookie
  - None: 任意发送cookie，设置为None，（需要同时设置Secure，也就是说网站必须采用https）
- Priority：优先级，chrome的提案（firefox不支持），定义了三种优先级，Low/Medium/High，当cookie数量超出时，低优先级的cookie会被优先清除。


## 7. js和服务端对cookie的操作有什么不同？
cookie有一个属性是HttpOnly，HttpOnly被设置时，表明该cookie只能被http请求读取，不能被js读取，具体的表现是:document.cookie读取到的内容不包含设置了HttpOnly的cookie。


## 8. js如何操作cookie

js操作读/写cookie的api是document.cookie。

读cookie `document.cookie`

```js
console.log(document.cookie); 
// cna=8fDUF573wEQCAWoLI8izIL6X; xlly_s=1; t=4a2bcb7ef27d32dad6872f9124694e19; _tb_token_=e3e11308ee6fe; hng=CN%7Czh-CN%7CCNY%7C156; thw=cn;  v=0; 
```

读取后的cookie是一个字符串，包含了所有cookie的name和value（用分号分隔），需要我们自行将cookie解析出来。

写cookie

```js
document.cookie = 'uid=123;expires=Mon Jan 04 2022 17:42:40 GMT;path=/;secure;'
document.cookie = 'uid=123;expires=Mon Jan 04 2022 17:42:40 GMT;path=/caikuai;domain=edu.360.cn;secure;samesite=lax' 
```

一次只能写一个cookie，想要写多个cookie需要操作多次。

```

删除cookie 只需要将一个已经存在的cookie名字过期时间设置为过去的时间即可。

​```js
document.cookie = 'uid=dkfywqkrhkwehf23;expires=' + new Date(0) + ';path=/;secure;'
```

修改cookie 重新赋值就好，旧值会覆盖新值。

注意：需要保证path和domain这两个值不变，否则会添加一个新的cookie。


## 9. 服务端如何读写cookie

第2节“cookie是如何工作的”告诉我们，服务端是可以读和写cookie的，从图中我们可以看到，写cookie时，一条Set-Cookie写入一条cookie。读cookie时，所有的cookie信息都被放进了cookie字段中。

以express使用为例：

写cookie

```js
res.setHeader('Set-Cookie', ['uid=123;maxAge: 900000; httpOnly: true']);
// 或者
res.cookie("uid",'123',{maxAge: 900000, httpOnly: true});
```

读取cookie

```js
console.log(req.getHeader('Cookie')); // 拿到所有cookie
// 或者
console.log(req.cookie.name);
```

## 10. Cookie 大小和数量的限制

不同的浏览器对Cookie的大小和数量的限制不一样，一般，单个域名下设置的Cookie不应超过30个，且每个Cookie的大小不应超过4kb，超过以后，Cookie将会被忽略，不会被设置。


## 11. 查看浏览器是否打开 Cookie 功能

```js
window.navigator.cookieEnabled // true
```


## 12. cookie 的共享策略是什么

domain和path共同决定了cookie可以被哪些url访问。

访问一个url时，如果url的host与domain一致或者是domain的子域名，并且url的路径与path部分匹配，那么cookie才可以被读取。


## 13. 跨域请求（CORS）中的cookie

首先cookie的SameSite需要设置为None。

其次对于将Access-Control-Allow-Credentials设置为true的接口（表示允许发送cookie），需要我们在发送ajax请求时，将withCredentials属性设为true。


## 14. cookie的编码

```js
document.cookie = 'uid=123;expires=Mon Jan 04 2022 17:42:40 GMT;path=/caikuai;domain=edu.360.cn;secure;samesite=lax'
```

从document.cookie的赋值中我们可以看到，存在等于号、冒号、分号、空格、逗号、斜杠等特殊字符，因此当cookie的key和value中存在这几种符号时，需要对其进行编码，一般会用encodeURIComponent/decodeURIComponent进行操作。


Q: 为什么不使用escape和encodeURI进行编码呢？
A: 因为相比这两个api，encodeURIComponent可以将更多的字符进行编码，比如"/"。


## 15. cookie与web安全

### cookie如何应对XSS漏洞

XSS漏洞的原理是，由于未对用户提交的表单数据或者url参数等数据做处理就显示在了页面上，导致用户提交的内容在页面上被做为html解析执行。

常规方案：对特殊字符进行处理，如"<"和">"等进行转义。

cookie的应对方案：对于用户利用script脚本来采集cookie信息，我们可以将重要的cookie信息设置为HttpOnly来避免cookie被js采集。

### cookie如何应对CSRF攻击

CSRF，中文名叫跨站请求伪造，原理是，用户登陆了A网站，然后因为某些原因访问了B网站（比如跳转等），B网站直接发送一个A网站的请求进行一些危险操作，由于A网站处于登陆状态，就发生了CSRF攻击（核心就是利用了cookie信息可以被跨站携带）！

常规方案：采用验证码或token等。

cookie的应对方案：由于CSRF攻击核心就是利用了cookie信息可以被跨站携带，那么我们可以对核心cookie的SameSite设置为Strict或Lax来避免。


## 16. 不同的浏览器共享cookie吗?

不同的浏览器互相不通信，彼此是独立的cookie。因此，我们在一个浏览器上登陆了某个网站，换到另一个浏览器上的时候需要重新登陆。


## 17. cookie和localStorage/sessionStorage的差异


| 特性           | cookie                                       | localstorage             | sessionstorage                               |
| -------------- | -------------------------------------------- | ------------------------ | -------------------------------------------- |
| 操作者         | 服务器和js                                   | js                       | js                                           |
| 数据的生命期   | 可设置失效时间                               | 除非被清除，否则永久保存 | 仅在当前会话下有效，关闭页面或浏览器后被清除 |
| 存放数据大小   | 4K左右                                       | 一般为5M                 | 一般为5M                                     |
| 与服务器端通信 | 每次都会携带在HTTP头中每次都会携带在HTTP头中 | 不与服务器通信           | 不与服务器通信                               |
| 易用性         | 原生的Cookie接口不友好，需要封装             | 接口易用                 | 接口易用                                     |


## 18. 哪些信息适合放到cookie中

cookie的增多无疑会加重网络请求的开销，而且每次请求都会将cookie完整的带上，因此对于那些“每次请求都必须要携带的信息（如身份信息、A/B分桶信息等）”，才适合放进cookie中，其他类型的数据建议放进localStorage中。