---
title: 前端怎么实现电子签名呢
date: 2024-05-28 12:15:52
tags:
categories: 前端
description: 在许多Web应用中，用户签名功能是一个常见需求，比如在电子合同、反馈表单或者用户认证中。本文将详细介绍如何在前端实现一个简单的签名功能，这个时候一般使用Canvas元素和JavaScript。
---

在许多Web应用中，用户签名功能是一个常见需求，比如在电子合同、反馈表单或者用户认证中。本文将详细介绍如何在前端实现一个简单的签名功能，这个时候一般使用Canvas元素和JavaScript。

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Signature Pad</title>
  <style>
    .canvas-container {
      border: 1px solid #000;
      position: relative;
    }
    .canvas-container canvas {
      display: block;
    }
    .controls {
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div class="canvas-container">
    <canvas id="signatureCanvas" width="600" height="400"></canvas>
  </div>
  <div class="controls">
    <button id="clearButton">Clear</button>
    <button id="saveButton">Save</button>
  </div>
  <script src="signature.js"></script>
</body>
</html>
```

signature.js:

```js
window.onload = function() {
  const canvas = document.getElementById('signatureCanvas');
  const ctx = canvas.getContext('2d');
  const clearButton = document.getElementById('clearButton');
  const saveButton = document.getElementById('saveButton');

  let drawing = false;
  let lastX = 0;
  let lastY = 0;

  canvas.addEventListener('mousedown', (e) => {
    drawing = true;
    [lastX, lastY] = [e.offsetX, e.offsetY];
  });

  canvas.addEventListener('mousemove', draw);
  canvas.addEventListener('mouseup', () => drawing = false);
  canvas.addEventListener('mouseout', () => drawing = false);

  function draw(e) {
    if (!drawing) return;
    ctx.strokeStyle = '#000';
    ctx.lineJoin = 'round';
    ctx.lineCap = 'round';
    ctx.lineWidth = 2;

    ctx.beginPath();
    ctx.moveTo(lastX, lastY);
    ctx.lineTo(e.offsetX, e.offsetY);
    ctx.stroke();
    [lastX, lastY] = [e.offsetX, e.offsetY];
  }

  clearButton.addEventListener('click', () => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  });

  saveButton.addEventListener('click', () => {
    const dataURL = canvas.toDataURL('image/png');
    const link = document.createElement('a');
    link.href = dataURL;
    link.download = 'signature.png';
    link.click();
  });
};
```

代码解释

- HTML部分：
  - 创建一个canvas元素，用于绘制签名。
  - 创建两个按钮，一个用于清除签名，一个用于保存签名。
- CSS部分：
  - 给canvas元素添加边框和其他样式。
- JavaScript部分：
  - 获取canvas元素及其上下文，以及两个按钮。
  - 添加鼠标事件监听器，实现绘制功能。
  - 绘制功能通过mousemove事件触发，使用Canvas API进行绘制。
  - 清除按钮通过调用clearRect方法清除canvas内容。
  - 保存按钮将canvas内容转换为图片并触发下载。