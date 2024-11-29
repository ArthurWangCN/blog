---
tag: plugins
---

# vite-plugin-legacy 插件介绍

## 背景

项目是用 vite + vue3 写的，部署完在客户电脑上打不开，进入不了系统里。经过排查，因为客户浏览器版本过低（chrome39），不支持 type="module" 的形式引入js文件，所以用 vite-plugin-legacy 对程序进行兼容性处理。

## 简介

vite 的runtime是基于 native ESM 的，所以如果开发者需要打包代码在 传统浏览器 or 兼容性差的浏览器版本， 就需要用到此插件。

注意：此插件暂时不支持动态返回html，如部分ssr插件。因为该插件的核心原理是打包后在 index.html 静态文件中注入 scripts， 而大部分ssr是动态返回html。

## 什么是polyfill

简单来说，polyfill就是兼容旧浏览器的代码块，磨平差异。比如说 有的浏览器不支持 globalThis, 那我们可以自己实现一个globalThis然后注入到script中。

注意：polyfill和代码编译(renderLegacyChunks)是两个概念， 前者是添加额外的代码来使得旧浏览器支持某些特性，后者是把浏览器不认识的语法转化为可以运行的语法。

vite的polyfill分为 modern polyfill(modernPolyfills属性)和 legacy polyfill(polyfills属性)，之所以区分开来，是为了尽量减少polyfills的大小

## 什么是传统浏览器

传统浏览器一般指不支持 native ESM 的浏览器，如chrome<60，Edge<15，Firefox<59 等等，如果使用vite打包而不做任何的处理的话，是无法在这些浏览器上面运行的，因为打包出来的代码是 很新的规范。

开发者则需要使用此插件配置相应的兼容处理，如：

```shell
npm i @vitejs/plugin-legacy
```

```ts
// vite.config.js
import legacy from '@vitejs/plugin-legacy'

export default {
  plugins: [
    legacy({
      targets: ['chrome < 60', 'edge < 15'],
      renderLegacyChunks: true,
    })
  ]
}
```
