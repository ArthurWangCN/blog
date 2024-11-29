---
title: CSS8个常见的布局
date: 2022-06-21 23:17:22
tags:
categories: 前端
description: 记录CSS中8个常见的布局
---

## 水平居中

**1. 使用 text-align**

```css
.parent { text-align: center; }
.child { display: inline-block; }
```

**2. 定宽块级元素**

方法一：

```css
.child { margin: 0 auto; }
```

方法二：

```css
.child {
    position: relative;
    left: 50%;
    margin-left: -150px;	/* margin-left 值为宽度的一半 */
}
```

方法三：

```js
.parent {
    height: 300px;
  	position: relative;
}
.child {
    position: absolute;
  	left: 0;
  	right: 0;
  	margin: 0 auto;
}
```

方法四：

```css
.parent { position: relative; }
.child {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
}
```

**3. flex 布局**

```css
.parent {
    height: 300px;
    display: flex;
    justify-content: center;
}
```

**4. grid 布局**

```css
.parent {
    height: 300px;
    display: grid;
    /* 方法一 */
    justify-items: center;
    /* 方法二 */
    justify-content: center;
}
.child {
    /* 方法三 */
    margin: auto;
}
```



## 垂直居中

**1. 行内块级元素**

```css
.parent {
    line-height: 500px;
}
.child {
    display: inline-block;
    vertical-align: middle;
}
```

**2. 定位方式**

方法一：

```css
.parent {
    position: relative;
}
.child {
    position: absolute;
    top: 50%;
    margin-top: -150px; /* margin-top 值为高度的一半 */
}
```

方法二：

```css
.parent {
  	position: relative;
}
.child {
    height: 300px;
    position: absolute;
  	top: 0;
  	bottom: 0;
  	margin: auto;
}
```

方法三：

```css
.parent {
    position: relative;
}
.child {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
}
```

**3. flex 布局**

```css
.parent {
    display: flex;
    /* 方法一 */
    align-items: center;
}
.child {
    /* 方法二 */
    margin: auto;
}
```

**4. grid 布局**

```css
.parent {
    display: grid;
    /* 方法一 */
    align-items: center;
    /* 方法二 */
    align-content: center;
}
.child {
    /* 方法三 */
    margin: auto;
    /* 方法四 */
    align-self: center;
}
```



## 水平垂直居中

**1. 行内块级元素**

```css
.parent {
    line-height: 500px;
    text-align: center;
}
.child {
    diaplay: inline-block;
    vertical-align: middle;
}
```

**2. 定位方式**

方法一：

```css
.parent {
    position: relative;
}
.child {
    position: absolute;
    left: calc(50% - 150px);
    top: calc(50% - 150px);
}
```

方法二：

```css
.parent {
    position: relative;
}
.child {
    position: absolute;
    left: 50%;
    top: 50%;
    margin-left: -150px;
    margin-top: -150px;
}
```

方法三：

```css
.parent {
    position: relative;
}
.child {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    margin: auto;
}
```

方法四：

```css
.parent {
    position: relative;
}
.child {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
}
```

**3. flex 布局**

```css
.parent {
    display: flex;
    align-items: center;
    justify-content: center;
}
.child {
    /* 方法二 */
    margin: auto;
}
```

**4. grid 布局**

```css
.parent {
    display: grid;
    /* items属性 */
    align-items: center;
    justify-items: center;
    /* items的缩写 */
    place-items: center;
    
    /* content属性 */
    align-content: center;
    justify-content: center;
    /* content的缩写 */
    place-content: center;
}
.child {
    /* 或者 margin auto */
    margin: auto;
    
    /* 或者self属性 */
    align-self: center;
    justify-self: center;
    /* self的缩写 */
    place-self: center;
}
```



## 两列布局

所谓的两列布局就是一列定宽(也有可能由子元素决定宽度)，一列自适应的布局。

![04_两列布局.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9ab7fc2f1eb471eb1a82838921b3aae~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

**1. float + calc()**

```css
.left {
    float: left;
}
.right {
    float: left; 
    width: calc(100% - 200px);
}
```

**2. float + margin-left**

```css
.left {
    float: left;
}
.right {
    margin-left: 200px;
}
```

**3. absolute + margin-left**

```css
.left {
    position: absolute;
}
.right {
    margin-left: 200px;
}
```

**4. float + overflow**

```css
.left {
    float: left;
}
.right {
    /* overflow创建一个BFC完成自适应 */
    overflow: hidden;
}
```

**5. flex 布局**

```css
.container {
    display: flex;
}
.right { flex: 1; }
```

**6. grid 布局**

```css
.container {
    display: grid;
    grid-template-columns: auto 1fr;
}
```



## 三列布局

三列布局主要分为两种：

- 第一种是前两列定宽，最后一列自适应，这一种本质上与两列布局没有什么区别，可以参照两列布局实现。
- 第二种是前后两列定宽，中间自适应。

![05_三列布局.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70c32186e66b4d7992a5a2f795b664ca~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

**1. position**

```css
.left {
    position: absolute;
    left: 0;
    top: 0;
}
.center {
    width: calc(100% - 400px);
    margin-left: 200px;
    margin-right: 200px;
}
.right {
    position: absolute;
    right: 0;
    top: 0;
}
```

**2. flex 布局**

```css
.container { display: flex; }
.center { flex: 1; }
```

**3. grid 布局**

```css
.container {
    display: grid;
    grid-template-columns: auto 1fr auto;
}
```



## 等分布局

等分布局就是将一个容器平均分成几等份，这里以 4 等分为例。

**1. 浮动 + 百分比**

```css
.item {
    float: left;
    width: 25%;
}
```

**2. 行内块级 + 百分比**

```css
.container {
    font-size: 0;
}
.item {
    display: inline-block;
    width: 25%;
}
```

**3. flex 布局**

```css
.container { display: flex; }
.item { flex: 1; }
```

**4. grid 布局**

```css
.container {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
}
```



## Sticky Footer布局

Sticky Footer布局是页面内容不够长时，底部栏就会固定到浏览器的底部；如果足够长时，底部栏就后跟随在内容的后面。

![07_Sticky Footer布局.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b4571c2f11445d983243d4bbb7648e0~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

**1. 绝对定位**

```css
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
}
.main {
  position: relative;
  min-height: 100%;
}
.bottom {
  position: absolute;
  width: 100%;
  height: 100px;
  bottom: 0;
}
```

**2. calc 函数**

``` css
.container {
    min-height: calc(100vh - 200px);
}
```

**3. flex 布局**

```css
.main {
    display: flex;
    flex-flow: column;
    min-height: 100vh;
}
.container {
    flex: 1;
}
```

**4. grid**

```css
.main {
    display: grid;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
}
```





## 全屏布局

全部布局主要应用在后台，主要效果如下所示：

![08_全屏布局.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d73b7dae92dd44f5b7c80fb39509f2d2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

**1. calc 函数**

```css
.content {
    overflow: hidden;
    height: calc(100vh - 200px);
}
.left {
    height: 100%;
}
.right {
    overflow: auto;
    height: 100%;
}
```

**2. flex**

```css
.container {
    display: flex;
    flex-flow: column;
}
.content {
    overflow: auto;
    flex: 1;
}
```

**3. gird**

```css
.container {
    display: grid;
    grid-template-rows: auto 1fr auto;
}
.content {
    overflow: auto;
}
```
