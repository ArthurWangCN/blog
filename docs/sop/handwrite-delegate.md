---
title: JavaScript 事件委托
date: 2022-05-06 23:31:18
tags: 手写
categories: JavaScript
description: 事件委托，也叫事件代理，即把原本需要绑定在子元素的响应事件委托给父元素，让父元素担当事件监听的职务。
---

# JavaScript 事件委托

## 事件委托

> 事件委托，也叫事件代理，即把原本需要绑定在子元素的响应事件委托给父元素，让父元素担当事件监听的职务。



事件委托的好处：

+ 节省资源, 当子元素个数较多, 且都要设置监听事件时, 可以统一设置到父元素上, **节省大量事件注册函数**；
+ **对异步插入的子元素仍然生效**。如果不使用事件委托, 那么异步插入的子元素默认不会有任何事件注册, 需要显式声明. 而使用事件委托, 即便是异步插入的元素, 事件仍然可以处理到。



事件委托的坏处:

+ 调试比较复杂，**不容易确定监听者**



示例：

```js
ul.addEventListener("click", function(e) {
    if (e.target && e.target.tagName.toLowerCase() === "li") {
        // ...
    }
})
```



这样写的问题在于，如果用户点击的是 li 里面的 span，就没法触发 fn，这显然不对。





## 手写

解决以上问题，思路是点击 span 后，递归遍历 span 的祖先元素看其中有没有 ul 里面的 li。

```js
function delegate(element, eventType, selector, fn) {
  element.addEventListener(eventType, e = >{
    let el = e.target
    while (!el.matches(selector)) {
      if (element === el) {
        el = null
        break
      }
      el = el.parentNode
    }
    el && fn.call(el, e, el)
  }) return element
}
```



使用：

```js
delegate(ul, 'click', 'li', f1)
```

