---
title: Canvas 与 SVG
date: 2022-05-04 00:27:24
tags:
categories: JavaScript
description: Canvas 和 SVG 都允许您在浏览器中创建图形，但是它们在根本上是不同的。
---

> Canvas 和 SVG 都允许您在浏览器中创建图形，但是它们在根本上是不同的。

## Canvas

> Canvas API 提供了一个通过 JavaScript 和 HTML的 `<canvas>`元素来绘制图形的方式。它可以用于动画、游戏画面、数据可视化、图片编辑以及实时视频处理等方面。

### 简介

+ `HTML5 <canvas>` 元素用于图形的绘制，通过脚本 （通常是JavaScript）来完成；
+ `<canvas>` 标签只是图形容器，必须使用脚本来绘制图形；
+ 浏览器支持：Internet Explorer 9+, Firefox, Opera, Chrome, 和 Safari ；
+ Canvas 坐标：canvas 是一个二维网格，左上角坐标为 (0,0)。

实例：

```js
// 1. 获取HTML canvas 元素的引用
const canvas = document.getElementById('canvas');
// 2. 方法获取这个元素的context——图像稍后将在此被渲染。
const ctx = canvas.getContext('2d');
// 由 CanvasRenderingContext2D 接口完成实际的绘制。
// 3. fillStyle 属性让长方形变成绿色。
ctx.fillStyle = 'green';
// 4. 将它的左上角放在(10, 10)，把它的大小设置成宽150高100。
ctx.fillRect(10, 10, 150, 100);
```

### Canvas路径

在Canvas上**画线**，我们将使用以下两种方法：

- `moveTo(x,y)` 定义线条开始坐标
- `lineTo(x,y)` 定义线条结束坐标

实例：

```js
var c = document.getElementById("myCanvas");
var ctx = c.getContext("2d");
ctx.moveTo(0,0);
ctx.lineTo(200,100);
ctx.stroke();
```

在canvas中**绘制圆形**, 我们将使用以下方法:

- `arc(x,y,r,start,stop)`

实例：

```js
var c = document.getElementById("myCanvas");
var ctx = c.getContext("2d");
ctx.beginPath();
ctx.arc(95,50,40,0,2*Math.PI);
ctx.stroke();
```

### Canvas文本

使用 canvas 绘制文本，重要的属性和方法如下：

- font - 定义字体
- `fillText(text,x,y)` - 在 canvas 上绘制实心的文本
- `strokeText(text,x,y)` - 在 canvas 上绘制空心的文本

实例：

```js
var c = document.getElementById("myCanvas");
var ctx = c.getContext("2d");
ctx.font = "30px Arial";
ctx.fillText("Hello World",10,50);
```

### Canvas渐变

渐变可以填充在矩形, 圆形, 线条, 文本等等, 各种形状可以自己定义不同的颜色。

以下有两种不同的方式来设置Canvas渐变：

- `createLinearGradient(x,y,x1,y1)` - 创建线条渐变
- `createRadialGradient(x,y,r,x1,y1,r1)` - 创建一个径向/圆渐变

当我们使用渐变对象，必须使用两种或两种以上的停止颜色。

`addColorStop()` 方法指定颜色停止，参数使用坐标来描述，可以是0至1。

使用渐变，设置fillStyle或strokeStyle的值为 渐变，然后绘制形状，如矩形，文本，或一条线。

实例：

```js
// 线性渐变
var c = document.getElementById("myCanvas");
var ctx = c.getContext("2d");

// Create gradient
var grd = ctx.createLinearGradient(0,0,200,0);
grd.addColorStop(0,"red");
grd.addColorStop(1,"white");

// Fill with gradient
ctx.fillStyle = grd;
ctx.fillRect(10,10,150,80);
```

```js
// 圆渐变
var c = document.getElementById("myCanvas");
var ctx = c.getContext("2d");

// Create gradient
var grd = ctx.createRadialGradient(75,50,5,90,60,100);
grd.addColorStop(0,"red");
grd.addColorStop(1,"white");

// Fill with gradient
ctx.fillStyle = grd;
ctx.fillRect(10,10,150,80);
```

### Canvas图像

把一幅图像放置到画布上, 使用以下方法:

+ `drawImage(image,x,y)`

实例：

```js
var c = document.getElementById("myCanvas");
var ctx = c.getContext("2d");
var img = document.getElementById("scream");

img.onload = function() {
	ctx.drawImage(img,10,10);
} 
```



## SVG

> 可缩放矢量图形（Scalable Vector Graphics，SVG），是一种用于描述二维的矢量图形，基于 XML 的标记语言。

### 什么是SVG？

+ SVG 指可伸缩矢量图形 (Scalable Vector Graphics)
+ SVG 用于定义用于网络的基于矢量的图形
+ SVG 使用 XML 格式定义图形
+ SVG 图像在放大或改变尺寸的情况下其图形质量不会有损失
+ SVG 是万维网联盟的标准
+ 浏览器支持：Internet Explorer 9+, Firefox, Opera, Chrome, 和 Safari

示例：

```html
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" height="190">
   <polygon points="100,10 40,180 190,60 10,60 160,180"
   style="fill:lime;stroke:purple;stroke-width:5;fill-rule:evenodd;">
</svg>
```

### SVG优势

与其他图像格式相比（比如 JPEG 和 GIF），使用 SVG 的优势在于：

- SVG 图像可**通过文本编辑器来创建和修改**；
- SVG 图像可被**搜索**、索引、脚本化或压缩；
- SVG 是**可伸缩**的；
- SVG 图像可在**任何的分辨率下被高质量地打印**；
- SVG 可在**图像质量不下降**的情况下被放大；



## SVG 与 Canvas两者间的区别

+ SVG 是一种使用 XML 描述 2D 图形的语言；Canvas 通过 JavaScript 来绘制 2D 图形。
+ SVG 基于 XML，这意味着 SVG DOM 中的每个元素都是可用的。您可以为某个元素附加 JavaScript 事件处理器。
+ 在 SVG 中，每个被绘制的图形均被视为对象。如果 SVG 对象的属性发生变化，那么浏览器能够自动重现图形。
+ Canvas 是逐像素进行渲染的。在 canvas 中，一旦图形被绘制完成，它就不会继续得到浏览器的关注。如果其位置发生变化，那么整个场景也需要重新绘制，包括任何或许已被图形覆盖的对象。

| **Canvas**                                         | **SVG**                                                 |
| -------------------------------------------------- | ------------------------------------------------------- |
| 依赖分辨率                                         | 不依赖分辨率                                            |
| 不支持事件处理器                                   | 支持事件处理器                                          |
| 弱的文本渲染能力                                   | 最适合带有大型渲染区域的应用程序（比如谷歌地图）        |
| 能够以 .png 或 .jpg 格式保存结果图像               | 复杂度高会减慢渲染速度（任何过度使用 DOM 的应用都不快） |
| 最适合图像密集型的游戏，其中的许多对象会被频繁重绘 | 不适合游戏应用                                          |

