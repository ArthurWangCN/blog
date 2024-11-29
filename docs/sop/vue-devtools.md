---
title: vue-devtools 安装教程
date: 2023-03-07 22:51:55
tags: Vue.js
categories: 工具
description: vue-devtools是一款基于chrome浏览器的插件，用于调试vue应用，这可以极大地提高我们的调试效率。
---

## 安装

> 省流版：https://crxdl.com/ 搜索vue就能下载 .crx 文件，之后打开谷歌扩展程序，拖进去即可。

**1. 克隆 vue-devtools 代码到本地**

```sh
git clone https://github.com/vuejs/vue-devtools.git
```

**2. 进入代码目录，安装依赖**

```sh
cd vue-devtools
npm install
```

如果你的 npm 是 npm7.x，那么很可能出现以下问题：

```js
npm ERR! code ERESOLVE
npm ERR! ERESOLVE unable to resolve dependency tree
npm ERR! 
npm ERR! While resolving: vue-devtools@6.0.0-beta.15
npm ERR! Found: eslint@7.32.0
npm ERR! node_modules/eslint
npm ERR!   dev eslint@"^7.26.0" from the root project
npm ERR!   peer eslint@"^7.12.1" from @vue/eslint-config-standard@6.1.0
npm ERR!   node_modules/@vue/eslint-config-standard
npm ERR!     dev @vue/eslint-config-standard@"^6.0.0" from the root project
npm ERR!   3 more (eslint-plugin-import, eslint-plugin-node, eslint-plugin-promise)
npm ERR! 
npm ERR! Could not resolve dependency:
npm ERR! peer eslint@"^5.0.0 || ^6.0.0" from eslint-plugin-vue@6.2.2
npm ERR! node_modules/eslint-plugin-vue
npm ERR!   dev eslint-plugin-vue@"^6.0.0" from the root project
npm ERR! 
npm ERR! Fix the upstream dependency conflict, or retry
npm ERR! this command with --force, or --legacy-peer-deps
npm ERR! to accept an incorrect (and potentially broken) dependency resolution.
npm ERR! 
npm ERR! See /Users/ykx/.npm/eresolve-report.txt for a full report.
 
npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/xxx/.npm/_logs/2021-09-01T06_06_25_956Z-debug.log
```

因为npm7.x对某些事情比npm6.x更严格

解决办法：

```sh
npm i --legacy-peer-deps
```

**3. 修改 manifest.json 文件 persistent 为 true**

vue-devtools>packages>shell-chrome>manifest.josn

**4. 在 vue-devtools 目录下编译代码**

```sh
npm run build
```

这个过程若报错如下：

```sh
lerna notice cli v4.0.0
lerna info Executing command in 9 packages: "yarn run build"
lerna ERR! yarn run build exited 2 in '@vue/devtools-api'
lerna ERR! yarn run build exited 2 in '@vue/devtools-api'
```

解决办法：

1. 安装 yarn 工具 `npm install yarn -g`
2. 用 yarn 安装依赖 `yarn install`
3. 用 yarn 构建 `yarn run build`

**5. 安装 Chrome 扩展插件**

打开 Chrome 浏览器，点击 三个点-更多工具-扩展程序-开发者模式-加载已解压的扩展程序- vue-devtools>packages>shell-chrome 放入。

启动 Vue 项目后，f12 在控制台中即会出现 vue 一栏，选中 vue 就可使用了。


 **安装后控制台不显示vue选项的可能原因：** 

1. 找到插件 -> 点击“详情” -> 把“允许访问文件网址”开启；
2. 入口文件（main.js）中禁用了 vue 调试工具：Vue.config.devtools = false 把它设置为 true 即可；
3. 因为使用了 vue 的压缩版本，所以浏览器控制台默认不显示 vue 调试工具，可以在 index.html 中加个判断条件，开发环境时不使用压缩版本。
    ```html
    <% if (process.env.NODE_ENV === 'production') { %>
    <script src="xxx/vue.min.js"></script>
    <% } else { %>
    <script src="xxx/vue.js"></script>
    <% } %>
    ```