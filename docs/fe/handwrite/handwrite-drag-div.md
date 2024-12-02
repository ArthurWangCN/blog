---
title: 写一个可拖拽的div
date: 2022-05-05 19:24:36
tags: [手写]
categories: JavaScript
description: 写一个可拖拽的div
---

# 写一个可拖拽的div

html：
```html
<div id="box"></div>
```

js：
```js
let isDragging = false;
let position = [0, 0];
let box = document.querySelector('#box');

box.addEventListener('mousedown', function(e) {
    isDragging = true;
    position = [e.clientX, e.clientY];
})

document.addEventListener('mousemove', function(e) {
    if (!isDragging) return;
    let deltaX = e.clientX - position[0];
    let deltaY = e.clientY - position[1];
    const left = parseInt(box.style.left || 0);
    const top = parseInt(box.style.top || 0);
    box.style.left = left + deltaX + 'px';
    box.style.top = top + deltaY + 'px';
    position = [e.clientX, e.clientY]
})

document.addEventListener('mouseup', function() {
    isDragging = false;
})
```

css：
```css
#box {
    width: 100px;
    height: 100px;
    background-color: brown;
    cursor: pointer;
    position: relative;
}
```



要点：

1. 注意监听范围，不能只监听div；
2. 不要使用drag事件，很难用；
3. 使用 transform 会比 top / left 性能更好，因为可以避免回流和重绘。





**transform版本：**

```js
document.addEventListener('mousemove', function(e) {
    if (!isDragging) return;
    let deltaX = e.clientX - position[0];
    let deltaY = e.clientY - position[1];

    // 需要获取到box的transform偏移量，最好改为正则匹配获取
    let transformArr = [0, 0];
    if (box.style.transform) {
        transformArr = box.style.transform.replace(/translate\(/g, '').split(',');
    }
    const left = parseInt(transformArr[0] || 0);
    const top = parseInt(transformArr[1] || 0);

    box.style.transform = `translate(${left + deltaX}px, ${top + deltaY}px)`;
    position = [e.clientX, e.clientY];
})
```

