---
title: 防抖与节流
date: 2023-02-15 21:30:01
tags: 高频
categories: JavaScript
description: 在前端开发中会遇到一些频繁的事件触发，比如：resize、scroll、mousedown、mousemove、keyup、keydown。频繁触发很容易造成一些性能问题，目前解决这种性能问题最常用的就是防抖和节流。
---

# 防抖与节流

在前端开发中会遇到一些频繁的事件触发，比如：resize、scroll、mousedown、mousemove、keyup、keydown。频繁触发很容易造成一些性能问题，目前解决这种性能问题最常用的就是防抖和节流。



## 防抖

防抖(debounce)，**在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时**。

防抖的原理就是：你尽管触发事件，但是我一定在事件触发 n 秒后才执行，如果你在一个事件触发的 n 秒内又触发了这个事件，那我就以新的事件的时间为准，n 秒后才执行，总之，就是要等你触发完事件 n 秒内不再触发事件才执行。

实现一个简单的防抖：

```js
// 第二版
function debounce(func, wait) {
    var timeout;

    return function () {
        var context = this;
        var args = arguments;

        clearTimeout(timeout)
        timeout = setTimeout(function(){
            func.apply(context, args)
        }, wait);
    }
}
```



## 节流

节流(throttle)，**规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效**。

节流的原理：如果你持续触发事件，每隔一段时间，只执行一次事件。

实现一个简单的节流：

```js
// 使用时间戳实现
function throttle(func, wait) {
    var context, args;
    var previous = 0;

    return function() {
        var now = +new Date();
        context = this;
        args = arguments;
        if (now - previous > wait) {
            func.apply(context, args);
            previous = now;
        }
    }
}

// 使用定时器实现
function throttle(func, wait) {
    var timeout;
    var previous = 0;

    return function() {
        context = this;
        args = arguments;
        if (!timeout) {
            timeout = setTimeout(function(){
                timeout = null;
                func.apply(context, args)
            }, wait)
        }

    }
}
```



## 区别

相同点：

- 都可以通过 setTimeout 实现
- 都是为了降低回调执行频率，节省计算资源

不同点：

- 防抖一段时间内利连续触发事件，只在最后执行一次；节流一段时间内连续触发只执行一次。

例如，都设置时间频率为500ms，在2秒时间内，频繁触发函数，节流，每隔 500ms 就执行一次。防抖，则不管调动多少次方法，在2s后，只会执行一次。



## 应用场景

防抖在连续的事件，只需触发一次回调的场景有：

- 搜索框搜索输入。只需用户最后一次输入完，再发送请求
- 手机号、邮箱验证输入检测
- 窗口大小`resize`。只需窗口调整完成后，计算窗口大小。防止重复渲染。

节流在间隔一段时间执行一次回调的场景有：

- 滚动加载，加载更多或滚到底部监听
- 搜索框，搜索联想功能