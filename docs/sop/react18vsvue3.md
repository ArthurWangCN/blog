---
date: 2022.11.22 11:11:11
---

# React18 与 Vue3 全方面对比
## 编程风格 & 视图风格
### 编程风格

1、React 语法少、难度大；Vue 语法多，难度小：

例如指令：

Vue3
```html
<input v-model="username"/>
<ul>
  <li v-for="(item,index) in list" :key="index">{{ item }}</li>
</ul>
```

React18
```html
<input value={username} onChange={e => setUsername(e.target.value)}/>
<ul>
  { list.map((item,index) => <li key={index}>{item}</li>) }
</ul>
```

Vue 给我们提供了很多的指令功能，而这些功能在 React 中基本都需要我们使用原生 js 来实现。

所以会有很多人说: "使用 Vue 实际上你就是在操作 Vue，使用 React 实际上你是在操作 js"。

2、React 魔改少，手动实现；Vue 魔改多，自动完成。

例如事件：

Vue
```html
<button @click="handleClick('hello')">点击</button>
const handleClick = (msg) => {
  console.log('msg')
}
```

React
```html
<button onClick="handleClick('hello')">点击</button>
const handleClick = (msg) => {
  return () => {
    console.log(msg)
  }
}
```

像在点击事件中传参数这种功能：

- 我们知道 dom 的点击事件是需要我们传递一个函数过去的，就像在 React 中例子一样，你的 handleClick 肯定需要返回一个函数（或者在 jsx 中写箭头函数调用 handleClick）。
- 而在 Vue 中可以在 @click 中直接调用 handleClick 函数，而这个函数又没有返回一个新的函数，按道理这样调用 handleClick 是会返回 undefined 的，但是由于 Vue 底层做了魔改优化，使得我们不再需要在返回一个函数。

上面两个例子中，我们说不上哪种好哪种不好，只能说你更喜欢哪一种。React 中的实现更符合 js 的逻辑但却稍显麻烦，Vue 中的实现简单但却没有遵循原生 js 的特点。
编程风格上的总结：就像我们前面讲的，Vue 写起来更像是写 Vue 代码，React 写起来更像是写 JavaScript 代码。

### 视图风格
1. Vue 采用 `<template>` 字符串模板。更贴近 HTML，学习成本低，但有时候不灵活。
2. React 采用 JSX 语法，更类似于 js ，限制比较多，（像一些关键字 class、for，单标签要闭合、属性要驼峰、组件名要大写等等这些都要注意），但是可以跟模板语法很好的进行结合

比如下面是一个通过 level 的值来渲染不同的标签在 Vue 和 React 中的不同实现

vue

```html
<template>
    <h1 v-if="level === 1">标题1</h1>
    <h2 v-if="level === 2">标题2</h1>
</template>
```

react

```js
let App = () => {
    const level = 1
    const Tag = 'h' + level
    return (
        <div>
            { <Tag>标题{level}</Tag>}
        </div>
    )
}
```

可以想象，如果当我们的条件判断很多时，使用 JSX 的方式会比使用模版字符串要灵活的多。

注意: Vue 一开始并不直接支持 JSX ，在 Vue 2.1.0 版本中，Vue 引入了 render 函数来代替模板，这使得使用 JSX 作为组件渲染函数成为可能。在Vue 2.1.0版本后的 create-vue 和 Vue CLI 都有预置的 JSX 语法支持。所以说在 Vue 中如果你想写 JSX 这个它也是支持的，但是在 React 是没办法用字符串模板的方式写。



https://juejin.cn/post/7210918245993611301#heading-8
