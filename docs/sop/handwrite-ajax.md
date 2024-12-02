---
title: 手写一个Ajax
date: 2022-05-10 20:31:27
tags: 手写
categories: JavaScript
description: Ajax（Asynchronous JavaScript and XML），即异步JavaScript和XML，可以无需刷新页面即可从服务器取得数据。
---

# 手写一个Ajax

> Ajax（Asynchronous JavaScript and XML），即异步JavaScript和XML，可以无需刷新页面即可从服务器取得数据。



**一个简单的Ajax（get）请求：**

```js
var xhr = new XMLHttpRequest(); 
xhr.open("GET", "/xxx");
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    if (xhr.status >= 200 && xhr.status < 300 || xhr.status === 304) {
      alert(xhr.responseText);
    } else {
      alert("Error:" + xhr.status);
    }
  }
}
xhr.send();
```



说明：

+ `new XMLHttpRequest()` 创建 xhr 对象，IE通过 `new ActiveXObject("Microsoft.XMLHTTP")` 创建；
+ `open` 方法初始化一个请求，原始API：`open(method, url, async)`；
+ `onreadystatechange` 方法只要 readystate 属性变化，就会调用相应的函数，readystate的值：
  + 0 - （未初始化）还没有调用send()方法
  + 1 - （载入) 已调用 send()方法，正在发送请求
  + 2 - （载入完成） send()方法执行完成，已经接受到全部响应内容
  + 3 - （交互）正在解析响应内容
  + 4 - （完成）响应内容解析完成，可以在客户端调用
+ status 状态码：
  + 2xx - 表示成功处理请求，如200
  + 3xx - 需要重定向，浏览器直接跳转，如301（永久重定向，永远都直接从A地址跳转到B址）302（临时重定向，只有一次重定向） 304（资源未改变，没有新资源使用缓存中的资源）
  + 4xx - 客户端请求错误，如 404（找不到） 403（没有访问权限）
  + 5xx - 服务器端错误
+ `XMLHttpRequest.send(body)` 方法用于发送 HTTP 请求。





**如果是 post 请求：**

+ open 方法第一个参数改为 `POST`；
+ `XMLHttpRequest.send(body)` 方法参数可以传数据体；
+ `XMLHttpRequest.setRequestHeader(header, value)` 方法设置请求头；



---

*XMLHttpRequest MDN：https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest*

