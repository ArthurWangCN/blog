---
title: react-devtools 简单安装教程
date: 2022-05-14 14:12:55
tags: react
categories: JavaScript
description: 在进行React开发的过程中，React提供了官方的浏览器扩展插件React Devtools调试工具。
---

1. 首先打开官网：https://github.com/facebook/react-devtools；
2. 进去v3分支，地址：https://github.com/facebook/react-devtools/tree/v3，直接download ZIP格式；
3. 知道下载位置，解压到自己可以找见的目录下，进入到react-devtools-3目录，cnpm i一下安装一下依赖；
4. 再进入到react-devtools-3\shells\chrome切换到chrome目录下，运行node build.js，当前目录下会生成build目录 这个build目录下的unpacked目录就是chrome中所需react-devtools的工具扩展程序包；
5. 打开谷歌浏览器，网址输入chrome://extensions/， 选择react-detools-3目录下的shells->chrome中build目录中的unpacked即可。

到此 react-devtools安装成功!

