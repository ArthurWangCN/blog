---
title: 单击点一些常用的坐标及宽高信息
date: 2022-05-14 20:58:22
tags:
categories: JavaScript
description: 单击点一些常用的坐标及宽高信息汇总
---

## 单击点坐标

+ x、y：单击点距离**屏幕左上角**的距离；
+ screenX、screenY：单击点距离**屏幕左上角**的距离；
+ clientX、clientY：单击点距离**页面可显示视图区域左上角**的距离；
+ pageX = clientX + scrollLeft; pageY = clientY + scrollTop;
+ offsetX、offsetY 属性，是单击点相对单击对象左上角的偏移量，取决于单击的对象位于哪是谁；



其中clientX、clientY是W3C标准的属性。



## 一些宽高

+ **clientWidth** = width + 左右padding；
+ **clientHeigh** = height + 上下padding；
+ **clientTop** = boder.top(上边框的宽度)；
+ **clientLeft** = boder.left(上边框的宽度)；



+ **offsetWidth** = width + 左右padding + 左右boder
+ **offsetHeith** = height + 上下padding + 上下boder
+  **offsetTop**：当前元素上边框外边缘到最近的已定位父级（offsetParent）上边框内边缘的距离。如果父              级都没有定位，则分别是到body顶部和左边的距离
+ **offsetLeft**：当前元素左边框外边缘到最近的已定位父级（offsetParent）左边框内边缘的距离。如果父级都没有定位，则分别是到body顶部和左边的距离



+ **scrollWidth**：获取指定标签内容层的真实宽度（可视区域宽度+被隐藏区域宽度）。
+ **scrollHeight**：获取指定标签内容层的真实高度（可视区域高度+被隐藏区域高度）。
+ **scrollTop** ：内容层顶部 到 可视区域顶部的距离。
+ **scrollLeft**：内容层左端 到 可视区域左端的距离。



## 扩展

### 获取scrollTop
```js
var scrollTop = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;
```

浏览器差异：

+ IE6/7/8：
  + 对于没有doctype声明的页面里可以使用  document.body.scrollTop 来获取 scrollTop高度 ； 
  + 对于有doctype声明的页面则可以使用 document.documentElement.scrollTop； 
+ Safari: safari 比较特别，有自己获取scrollTop的函数 ： window.pageYOffset ； 
+ Firefox: 火狐等等相对标准些的浏览器就省心多了，直接用 document.documentElement.scrollTop ；



### 滚动加载更多的原理

+ scrollHeight：文档内容实际高度，包括超出视窗的溢出部分；
+ scrollTop：滚动条滚动距离；
+ clientHeight：窗口可视范围高度。

当 `clientHeight + scrollTop >= scrollHeight` 时，表示已经抵达内容的底部了，可以加载更多内容。

+ scrollHeight：可以通过 document.documentElement.scrollHeight 、document.body.scrollHeight 获取;
+ clientHeight：可以通过window.innerHeight 、 document.documentElement.clientHeight 获取;
+ scrollTop：可以通过window.pageYOffset 、 document.documentElement.scrollTop 获取;
