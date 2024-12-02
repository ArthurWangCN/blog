---
title: Node.js 模块查找规则
date: 2023-03-08 12:14:27
tags: Node.js
categories: Node.js
description: Node.js 模块查找规则
---

# Node.js 模块查找规则

## 当前模块拥有路径但没有后缀时

```javascript
require('./find.js')	// 1
require('./find')		// 2
```

1. require 方法根据模块查找路径查找模块，如果是完整路径，直接引入模块
2. 如果模块后缀省略，先找同名 JS 文件，再找同名文件夹
3. 如果找到了同名文件夹，找文件夹中的 `index.js`
4. 如果文件夹中没有 index.js 就会去当前文件夹中的 `package.json` 文件中查找 `main` 选项中的入口文件
5. 如果找指定的入口文件不存在或者没有指定入口文件就会报错，模块没有找到

## 当模块没有路径且没有后缀时

```javascript
require('find')
```

1. Node.js 会假设它是系统模块
2. Node.js 会去 `node_modules` 文件夹中
3. 首先看是否有该名字的 JS 文件
4. 再看是否有该名字的文件夹
5. 如果是文件夹看里面是否有 `index.js`
6. 如果没有 index.js 查看该文件夹中的 `package.json` 中的 `main` 选项确定模块入口文件
7. 否则找不到报错