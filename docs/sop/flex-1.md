---
title: flex 1 是怎么工作的
date: 2023-02-18 10:37:51
tags:
categories: CSS
description: 你有没有想过 CSS 中的 flex属性如何工作?它是 flex-grow，flex-shrink和flex-basis的简写。开发中最常见的写法是flex：1，它表示 flex 项目扩展并填充可用空间。
---

# flex 1

flex: 1 实际上是三个属性的缩写：flex-grow: 1; flex-shrink: 1 flex-basis: auto。开发中最常见的写法是flex：1，它表示 flex 项目扩展并填充可用空间。

## flex-grow

> flex-grow 设置 flex 项 主尺寸 的 flex 增长系数。

默认为0，即如果存在剩余空间，也不放大。

设置了 flex-grow 的项目宽度 = (( flex-grow / flex-grow 总个数) * 可用空间）+ 初始项目宽度。

有一个常见的误解，使用flex-grow会使项目的宽度相等。这是不正确的，flex-grow的作用是分配可用空间。正如在公式中看到的，每 flex 项目的宽度是基于其初始宽度计算的(应用flex-grow之前的宽度)。

如果你想让项目的宽度相等，可以使用flex-basis，这个在接下来的部分会对此进行讲解。


## flex-shrink

> 指定 flex 元素的收缩规则。flex 元素仅在默认宽度之和大于容器的时候才会发生收缩，其收缩的大小是依据 flex-shrink 的值。

举个例子：

```css
.content {
    width: 500px;
    display: flex; 
}
.box-1 {
    background: aqua;
    width: 400px;
    flex-shrink: 1;
}
.box-2 {
    background: pink;
    width: 200px;
    flex-shrink: 2;
}
```

这边设置flex下2个子元素的宽度分别是 400px 和 300px，可以明显的看出已经超出父元素设置的500px。

那么这是flex-shrink就会起到作用，它会根据flex-shrink设置的值进行收缩。

如果flex-shrink设置为0表示当前元素不会进行收缩，flex-shrink的默认值为1;

flex-shrink的收缩公式（以这个为例）：

 **子元素超出的宽度 * flex-shrink的值 * 子元素宽度 / 总值** 

总值的获取（以当前代码为例）：1(box-1的flex-shrink值) * 400(box-1的宽度) + 2(box-2的flex-shrink值) * 300(box-2的宽度) = 1000;

以当前代码为例计算：

子元素超出的值：500 - (400 + 300) = 200;

总值：1 * 400 + 2 * 300 = 1000;

.box-1收缩的宽度：200 * 1 * 400 / 1000 = 80;

.box-2收缩的宽度：200 * 2 * 300 / 1000 = 120;


## flex-basis

> 指定 flex 元素在主轴方向上的初始大小。如果不使用 box-sizing 改变盒模型的话，那么这个属性就决定了 flex 元素的内容盒（content-box）的尺寸。

浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为auto，即项目的本来大小。

flex-basis可以设为跟width或height属性一样的值(比如350px，默认值为 auto)，则项目将占据固定空间。

当一个元素同时被设置了 flex-basis (除值为 auto 外) 和 width (或者在 flex-direction: column 情况下设置了height) , flex-basis 具有更高的优先级。


## flex 属性

flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。这也说 flex 项目会根据其内容大小增长。


flex 项目的大小取决于内容。因此，内容越多的flex项目就会越大：

```css
.item { 
    /* 默认值，相当于 flex：1 1 auto */ 
    flex: auto; 
} 
```

当flex-basis属性设置为0时，所有flex项目大小会保持一致：

```css
.item { 
    /* 相当于  flex: 1 1 0% */ 
    flex: 1; 
} 
```


## 最后

参考链接：

- https://www.51cto.com/article/683878.html
- https://blog.csdn.net/m0_58875326/article/details/124444419
- https://blog.csdn.net/LueLueLue77/article/details/124860668