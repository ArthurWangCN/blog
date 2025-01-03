# 第一篇 框架设计概览

## 第一章、权衡的艺术

在本章中，我们先讨论了**命令式和声明式**这两种范式的差异，其中命令式更加关注过程，而声明式更加关注结果。命令式在理论上可以做到极致优化，但是用户要承受巨大的心智负担；而声明式能够有效减轻用户的心智负担，但是性能上有一定的牺牲，框架设计者要想办法尽量使性能损耗最小化。

接着，我们讨论了虚拟 DOM 的性能，并给出了一个公式：**声明式的更新性能消耗 = 找出差异的性能消耗 + 直接修改的性能消耗**。虚拟 DOM 的意义就在于使找出差异的性能消耗最小化。我们发现，用原生 JavaScript 操作 DOM 的方法（如document.createElement）、虚拟 DOM 和 innerHTML 三者操作页面的性能，不可以简单地下定论，这与页面大小、变更部分的大小都有关系，除此之外，与创建页面还是更新页面也有关系，选择哪种更新策略，需要我们结合心智负担、可维护性等因素综合考虑。一番权衡之后，我们发现虚拟 DOM 是个还不错的选择。

最后，我们介绍了运行时和编译时的相关知识，了解纯运行时、纯编译时以及两者都支持的框架各有什么特点，并总结出 Vue.js 3 是一个编译时 + 运行时的框架，它在保持灵活性的基础上，还能够通过编译手段分析用户提供的内容，从而进一步提升更新性能。

## 第二章、框架设计的核心要素

### 2.1 提升用户的开发体验

在框架设计和开发过程中，提供友好的警告信息至关重要

优化控制台输出结果，是的ref等打印结果更直观：勾选 "Console" → "Enable custom formatters" 选项。

### 2.2 控制框架代码的体积

如果我们去看 Vue.js 3 的源码，就会发现每一个 warn 函数的调用都会配合 `__DEV__` 常量的检查

```js
if (__DEV__ && !res) {
    warn(`Failed to mount app: mount target selector "${container}"returned null.`)
}
```

这样我们就做到了在开发环境中为用户提供友好的警告信息的同时，不会增加生产环境代码的体积。

### 2.3 框架要做到良好的 Tree-Shaking

什么是 Tree-Shaking 呢？在前端领域，这个概念因 rollup.js 而普及。简单地说，Tree-Shaking 指的就是消除那些永远不会被执行的代
码，也就是排除 dead code，现在无论是 rollup.js 还是 webpack，都支持Tree-Shaking。

想要实现 Tree-Shaking，必须满足一个条件，即模块必须是 ESM（ES Module），因为 Tree-Shaking 依赖 ESM 的静态结构。

如果一个函数调用会产生副作用，那么就不能将其移除。而到底会不会产生副作用，只有代码真正运行的时候才能知道，JavaScript 本身是动态语言，因此想要静态地分析哪些代码是 dead code 很有难度。

```js
import {foo} from './utils'
/*#__PURE__*/ foo()
```

注意注释代码 `/*#**PURE***/`，其作用就是告诉 rollup.js，对于 foo 函数的调用不会产生副作用，你可以放心地对其进行 TreeShaking。

### 2.4 框架应该输出怎样的构建产物

```html
<script src="/path/to/vue.js"></script>
```

为了实现这个需求，我们需要输出一种叫作 IIFE 格式的资源。IIFE 的全称是 Immediately Invoked Function Expression，即“立即调用的函数表达式”。

```html
<script type="module" src="/path/to/vue.esm-browser.js"></script>
```

随着技术的发展和浏览器的支持，现在主流浏览器对原生 ESM 的支持都不错，需要提供 ESM 版本。

**为什么 vue.esm-browser.js 文件中会有 `-browser` 字样？**

其实对于 ESM 格式的资源来说，Vue.js 还会输出一个 vue.esm-bundler.js 文件，其中 `-browser` 变成了 `-bundler`。为什么这么做呢？我们知道，无论是 rollup.js 还是 webpack，在寻找资源时，如果 package.json 中存在 module 字段，那么会优先使用 module 字段指向的资源来代替 main 字段指向的资源。我们可以打开 Vue.js 源码中的 packages/vue/package.json 文件看一下：
```json
{
    "main": "index.js",
    "module": "dist/vue.runtime.esm-bundler.js",
}
```
其中 module 字段指向的是 vue.runtime.esm-bundler.js 文件，意思是说，如果项目是使用 webpack 构建的，那么你使用的 Vue.js 资源就是 vue.runtime.esm-bundler.js 也就是说，带有 -bundler 字样的 ESM 资源是给 rollup.js 或 webpack 等打包工具使用的，而带有 -browser 字样的 ESM 资源是直接给 `<script type="module">` 使用的。它们之间有何区别？这就不得不提到上文中的 `__DEV__` 常量。当构建用于 `<script>` 标签的 ESM 资源时，如果是用于开发环境，那么 `__DEV__` 会设置为 true；如果是用于生产环境，那么 `__DEV__` 常量会设置为 false，从而被 Tree-Shaking 移除。但是当我们构建提供给打包工具的 ESM 格式的资源时，不能直接把 `__DEV__` 设置为 true 或 false，而要使用 (process.env.NODE_ENV !== 'production') 替换 `__DEV__` 常量。

这样做的好处是，用户可以通过 webpack 配置自行决定构建资源的目标环境，但是最终效果其实一样，这段代码也只会出现在开发环境中。

```js
const Vue = require('vue')
```

我们还希望用户可以在 Node.js 中通过 require 语句引用资源，为什么会有这种需求呢？答案是“服务端渲染”。在 Node.js 环境中，资源的模块格式应该是 CommonJS，简称 cjs。


### 2.5 特性开关

- 对于用户关闭的特性，我们可以利用 Tree-Shaking 机制让其不包含在最终的资源中。
- 该机制为框架设计带来了灵活性，可以通过特性开关任意为框架添加新的特性，而不用担心资源体积变大。同时，当框架升级时，我们也可以通过特性开关来支持遗留 API，这样新用户可以选择不使用遗留 API，从而使最终打包的资源体积最小化。

### 2.6 错误处理

我们提供了 registerErrorHandler 函数，用户可以使用它注册错误处理程序，然后在 callWithErrorHandling 函数内部捕获错误后，把错误传递给用户注册的错误处理程序。

在 Vue.js 中，我们也可以注册统一的错误处理函数：

```js
import App from 'App.vue'
const app = createApp(App)
app.config.errorHandler = () => {
    // 错误处理程序
}
```

### 2.7 良好的 TypeScript 类型支持

## 第三章、Vue.js 3 的设计思路

### 3.1 初识渲染器

虚拟 DOM 其实就是用 JavaScript对象来描述真实的 DOM 结构。

渲染器的作用就是把虚拟 DOM 渲染为真实 DOM。

一个简单的渲染器 renderer 的实现思路：

1. 创建元素：把 vnode.tag 作为标签名称来创建 DOM 元素。
2. 为元素添加属性和事件：遍历 vnode.props 对象，如果 key 以on 字符开头，说明它是一个事件，把字符 on 截取掉后再调用 toLowerCase 函数将事件名称小写化，最终得到合法的事件名称，例如 onClick 会变成 click，最后调用addEventListener 绑定事件处理函数。
3. 处理 children：如果 children 是一个数组，就递归地调用 renderer 继续渲染，注意，此时我们要把刚刚创建的元素作为挂载点（父节点）；如果 children 是字符串，则使用 createTextNode 函数创建一个文本节点，并将其添加到新创建的元素内。

```js
function renderer(vnode, container) {
    // 使用 vnode.tag 作为标签名称创建 DOM 元素
    const el = document.createElement(vnode.tag)
    // 遍历 vnode.props，将属性、事件添加到 DOM 元素
    for (const key in vnode.props) {
        if (/^on/.test(key)) {
            // 如果 key 以 on 开头，说明它是事件
            el.addEventListener(key.substr(2).toLowerCase(), // 事件名称 onClick --->
            click vnode.props[key] // 事件处理函数
            )
        }
    }

    // 处理 children
    if (typeof vnode.children === 'string') {
        // 如果 children 是字符串，说明它是元素的文本子节点
        el.appendChild(document.createTextNode(vnode.children))
    } else if (Array.isArray(vnode.children)) {
        // 递归地调用 renderer 函数渲染子节点，使用当前元素 el 作为挂载点
        vnode.children.forEach(child = >renderer(child, el))
    }

    // 将元素添加到挂载点下
    container.appendChild(el)
}
```

### 3.2 组件的本质

组件就是一组 DOM 元素的封装，这组 DOM 元素就是组件要渲染的内容，因此我们可以定义一个函数来代表组件，而函数的返回值就代表组件要渲染的内容。

```js
const MyComponent = function() {
    return {
        tag: 'div',
        props: {
            onClick: () = >alert('hello')
        },
        children: 'click me'
    }
}
```

组件一定得是函数吗？当然不是，我们完全可以使用一个 JavaScript 对象来表达组件，例如：

```js
// MyComponent 是一个对象
const MyComponent = {
    render() {
        return {
            tag: 'div',
            props: {
                onClick: () = >alert('hello')
            },
            children: 'click me'
        }
    }
}
```

这里我们使用一个对象来代表组件，该对象有一个函数，叫作 render，其返回值代表组件要渲染的内容。

### 3.3 模板的工作原理

编译器的作用其实就是将模板编译为渲染函数：

```html
<div @click="handler">
    click me
</div>
<!-- 编译后： -->
render() {
    return h('div', { onClick: handler }, 'click me')
}
```

无论是使用模板还是直接手写渲染函数，对于一个组件来说，它要渲染的内容最终都是通过渲染函数产生的，然后渲染器再把渲染函数返回的虚拟 DOM 渲染为真实 DOM，这就是模板的工作原理，也是 Vue.js 渲染页面的流程。

### 3.4 Vue.js 是各个模块组成的有机整体

如前所述，组件的实现依赖于渲染器，模板的编译依赖于编译器，并且编译后生成的代码是根据渲染器和虚拟 DOM 的设计决定的，因此 Vue.js 的各个模块之间是互相关联、互相制约的，共同构成一个有机整体。

假设我们有如下模板：

```vue
<div id="foo" :class="cls"></div>
```

编译器会把这段代码编译成渲染函数：

```js
render() {
    // 为了效果更加直观，这里没有使用 h 函数，而是直接采用了虚拟 DOM 对象
    // 下面的代码等价于：
    // return h('div', { id: 'foo', class: cls })
    return {
        tag: 'div',
        props: {
            id: 'foo',
            class: cls
        }
    }
}
```

可以发现，在这段代码中，cls 是一个变量，它可能会发生变化。我们知道渲染器的作用之一就是寻找并且只更新变化的内容，所以当变量 cls 的值发生变化时，渲染器会自行寻找变更点。对于渲染器来说，这个“寻找”的过程需要花费一些力气。那么从编译器的视角来看，它能否知道哪些内容会发生变化呢？如果编译器有能力分析动态内容，并在编译阶段把这些信息提取出来，然后直接交给渲染器，这样渲染器不就不需要花费大力气去寻找变更点了吗？这是个好想法并且能够实现。Vue.js 的模板是有特点的，拿上面的模板来说，我们一眼就能看出其中 `id="foo"` 是永远不会变化的，而 `:class="cls"` 是一个 v-bind 绑定，它是可能发生变化的。所以编译器能识别出哪些是静态属性，哪些是动态属性，在生成代码的时候完全可以附带这些信息：

```js
render() {
    return {
        tag: 'div',
        props: {
            id: 'foo',
            class: cls
        },
        patchFlags: 1 // 假设数字 1 代表 class 是动态的
    }
}
```

如上面的代码所示，在生成的虚拟 DOM 对象中多出了一个 patchFlags 属性，我们假设数字 1 代表“ class 是动态的”，这样渲染器看到这个标志时就知道：“哦，原来只有 class 属性会发生改变。”对于渲染器来说，就相当于省去了寻找变更点的工作量，性能自然就提升了。

### 3.5 总结

在本章中，我们首先介绍了声明式地描述 UI 的概念。我们知道，Vue.js 是一个声明式的框架。声明式的好处在于，它直接描述结果，用户不需要关注过程。Vue.js 采用模板的方式来描述 UI，但它同样支持使用虚拟 DOM 来描述 UI。虚拟 DOM 要比模板更加灵活，但模板要比虚拟 DOM 更加直观。

然后我们讲解了最基本的渲染器的实现。渲染器的作用是，把虚拟 DOM 对象渲染为真实 DOM 元素。它的工作原理是，递归地遍历虚拟 DOM 对象，并调用原生 DOM API 来完成真实 DOM 的创建。渲染器的精髓在于后续的更新，它会通过 Diff 算法找出变更点，并且只会更新需要更新的内容。后面我们会专门讲解渲染器的相关知识。

接着，我们讨论了组件的本质。组件其实就是一组虚拟 DOM 元素的封装，它可以是一个返回虚拟 DOM 的函数，也可以是一个对象，但这个对象下必须要有一个函数用来产出组件要渲染的虚拟 DOM。渲染器在渲染组件时，会先获取组件要渲染的内容，即执行组件的渲染函数并得到其返回值，我们称之为 subtree，最后再递归地调用渲染器将 subtree 渲染出来即可。


