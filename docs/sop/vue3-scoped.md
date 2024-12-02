---
title: vue3 scoped 和 样式穿透
date: 2023-03-21 21:23:07
tags: Vue.js
categories: JavaScript
description: vue 中的 scoped 通过在DOM结构以及css样式上加唯一不重复的标记 data-v-hash 的方式，以保证唯一，达到样式私有化模块化的目的。
---

# vue3 scoped 和 样式穿透

## scoped原理

vue 中的 scoped 通过在DOM结构以及css样式上加唯一不重复的标记: `data-v-hash` 的方式，以保证唯一（而这个工作是由过PostCSS转译实现的），达到样式私有化模块化的目的。

总结一下scoped三条渲染规则：

1. 给HTML的DOM节点加一个不重复data属性(形如：`data-v-123`)来表示他的唯一性
2. 在每句css选择器的末尾（编译后的生成的css语句）加一个当前组件的data属性选择器（如 `[data-v-123]`）来私有化样式
3. 如果组件内部包含有其他组件，只会给其他组件的最外层标签加上当前组件的data属性

PostCSS会给一个组件中的所有dom添加了一个独一无二的动态属性data-v-xxxx，然后，给CSS选择器额外添加一个对应的属性选择器来选择该组件中dom，这种做法使得样式只作用于含有该属性的dom——组件内部dom, 从而达到了'样式模块化'的效果。


## 样式穿透

在vue3中项目中，使用深度选择器可能会出现如下错误：

`[@vue/compiler-sfc] the >>> and /deep/ combinators have been deprecated. Use :deep() instead.`

意思是 `>>>` 和 `/deep/` 已经被弃用，要用 `:deep()` 代替：

```css
.父元素class :deep(.class){ xxx }
```

他的作用是用来改变属性选择器的位置，由 `.father input[data-v-9ew234df]` 改为 `.father[data-v-9ew234df] input`。