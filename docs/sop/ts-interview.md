---
title: ts interview
date: 2023-02-14 10:40:45
tags: TypeScript
categories: JavaScript
description: TypeScript一些常见的问题。
---

# ts interview

## type 和 interface的区别

1. 声明事，type 后面有 `=`，interface 没有；
2. type 可以描述任何类型组合，interface 只能描述对象结构；
3. interface 可以继承自（extends）interface 或对象结构的 type。type 也可以通过 `&` 做对象结构的继承；
4. 多次声明的同名 interface 会进行声明合并，type 则不允许多次声明；

大多数情况下，我更推荐使用 interface，因为它扩展起来会更方便，提示也更友好。`&` 真的很难用。

