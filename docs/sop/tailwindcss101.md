---
title: Tailwind CSS 快速入门
date: 2023-03-21 21:26:23
tags: CSS
categories: CSS
description: Tailwind CSS 是一个利用公用程序类（Utilize Class）的 CSS 框架。
---

# Tailwind CSS 快速入门

## 什么是 Tailwind CSS 

Tailwind CSS 是一个利用公用程序类（Utilize Class，下文皆称Utilize Class）的 CSS 框架。许多人会想到 CSS 框架，有很多，例如 Bootstrap、Bulma 和 Material UI。Bootstrap 和 Bulma 等框架利用预先准备好的组件（例如按钮、菜单和面包屑）进行设计。在 Tailwind CSS 中，没有准备任何组件，而是使用Utilize Class来创建和设计自己的组件。


## 为什么需要 Tailwind CSS 

传统情况下，当您需要在网页上设置样式时，都需要编写 CSS。使用传统方式时，定制的设计需要定制的 CSS：

```html
<div class="chat-notification">
  <div class="chat-notification-logo-wrapper">
    <img class="chat-notification-logo" src="/img/logo.svg" alt="ChitChat Logo">
  </div>
  <div class="chat-notification-content">
    <h4 class="chat-notification-title">ChitChat</h4>
    <p class="chat-notification-message">You have a new message!</p>
  </div>
</div>

<style>
  .chat-notification {
    display: flex;
    max-width: 24rem;
    margin: 0 auto;
    padding: 1.5rem;
    border-radius: 0.5rem;
    background-color: #fff;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  }
  .chat-notification-logo-wrapper {
    flex-shrink: 0;
  }
  .chat-notification-logo {
    height: 3rem;
    width: 3rem;
  }
  .chat-notification-content {
    margin-left: 1.5rem;
    padding-top: 0.25rem;
  }
  .chat-notification-title {
    color: #1a202c;
    font-size: 1.25rem;
    line-height: 1.25;
  }
  .chat-notification-message {
    color: #718096;
    font-size: 1rem;
    line-height: 1.5;
  }
</style>
```

使用 Tailwind，可以通过直接在 HTML 中应用预先存在的类来设置元素的样式。使用功能类构建自定义设计而无需编写 CSS：

```html
<div class="p-6 max-w-sm mx-auto bg-white rounded-xl shadow-md flex items-center space-x-4">
  <div class="flex-shrink-0">
    <img class="h-12 w-12" src="/img/logo.svg" alt="ChitChat Logo">
  </div>
  <div>
    <div class="text-xl font-medium text-black">ChitChat</div>
    <p class="text-gray-500">You have a new message!</p>
  </div>
</div>
```

在上面的示例中，我们使用了：

- 使用 Tailwind 的 flexbox 和 padding 功能类 (flex, flex-shrink-0, 和 p-6) 来控制整体的卡片布局
- 使用 max-width 和 margin 功能类 (max-w-sm 和 mx-auto) 来设置卡片的宽度和水平居中
- 使用 background color, border radius, 和 box-shadow 功能类 (bg-white, rounded-xl, 和 shadow-md) 设置卡片的外观样式
- 使用 width 和 height 功能类 (w-12 and h-12) 来设置 logo 图片的大小
- 使用 space-between 功能类 (space-x-4) 来处理 logo 和文本之间的间距
- 使用 font size，text color，和 font-weight 功能类 (text-xl，text-black，font-medium 等等) 给卡片文字设置样式

这种方法使我们无需编写一行自定义的 CSS 即可实现一个完全定制的组件设计。

你可能觉得这样比较乱，写法丑陋，但是他也有很多优点：

- **您没有为了给类命名而浪费精力**。 不需要仅仅为了设置一些样式而额外添加一些像 sidebar-inner-wrapper 这样愚蠢的类名，不必再为了一个 flex 容器的完美抽象命名而倍受折磨。
- **您的 CSS 停止增长**。 使用传统方法，每次添加新功能时 CSS 文件都会变大。使用功能类，所有内容都是可重用的，因此您几乎不需要编写新的CSS。
- **更改会更安全**。 CSS 是全局性的，您永远不知道当您进行更改时会破坏掉什么。您 HTML 中的类是本地的，因此您可以更改它们而不必担心其他问题。


## 快速开始

创建 vite + vue-ts 项目：

```bash
yarn create vite
```

安装 Tailwind CSS 依赖：

```bash
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
```

生成 tailwind 和 postcss 配置文件：

```bash
npx tailwindcss init -p
```

tailwindcss 3.x 版本的配置文件：

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  darkMode: 'media',
  theme: {
    extend: {},
    container: {
      center: true,
    }
  },
  plugins: [],
}
```

在全局样式文件中导入tailwind：

src\styles\index.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

src\styles\index.scss

```scss
@import "tailwindcss/base";
@import "tailwindcss/components";
@import "tailwindcss/utilities";
```

全局样式文件在main.ts中导入：

```js
import { createApp } from 'vue'
import App from './App.vue'

// 导入全局样式文件
import './styles/index.scss'

createApp(App).mount('#app')
```

安装VSCode插件：

- Tailwind CSS IntelliSense支持自动完成、语法高亮、悬停预览、语法分析功能。
- PostCSS Language Support支持css未知规则如tailwind中的 @tailwind、@apply、@screen。
- 在.vue、.html文件中使用@apply仍提示未知规则，建议在已安装以上插件后再添加工作区配置禁止掉这个提示：`{ "css.lint.unknownAtRules": "ignore" }`。 


## 具体使用及API

详见 [tailwindcss官网](https://www.tailwindcss.cn/docs/utility-first)。

## 最后

个人感觉 Tailwind CSS 并没有太多让人「惊讶」的特性，但是在打造一个完整的设计系统这件事情上，Tailwind CSS 在每个方面都做了精心的设计，所以让人用起来很「舒服」。如果你还没有使用过 Tailwind CSS，建议可以尝试一下，相信会给你带来耳目一新的感受！

提高可维护性的方法：

从上面的例子可以看出，使用 Tailwind 后代码的风格趋于内联样式的编写，也带来的阅读的烦恼，解决这样的问题提供了下面两个常用的方法：

1. 抽取相同、类似的布局为公共组件、模板，提高复用性；
2. 对于没有必要或不应该提取为组件的简单元素，可以使用@apply抽象CSS类，就跟我们以前编写 class 一样来组合 Tailwind 功能类；