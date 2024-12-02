---
title: npm 使用淘宝镜像
date: 2022-05-14 14:17:58
tags: npm
categories: 工程化
description: npm 设置淘宝镜像源，来解决安装失败的问题。
---

# npm 使用淘宝镜像

**单次修改：**

```shell
npm install --registry=https://registry.npm.taobao.org
```



**永久使用：**

```shell
npm config set registry https://registry.npm.taobao.org
```



**检测是否成功：**

```shell
npm config get registry
```



**还原**

```shell
npm config set registry https://registry.npmjs.org/
```
