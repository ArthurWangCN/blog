---
title: Chrome多版本共存
date: 2024-06-17 20:23:40
tags:
categories: 工具
description: 有的项目要求兼容低版本Chrome，所以就需要多版本共存以方便调试
---

有的项目要求兼容低版本Chrome，所以就需要多版本共存以方便调试

下载地址：https://www.chromedownloads.net/chrome64win-stable/，下载 Chrome 离线安装安装包 (注意是离线安装包)。

右键属性查看安装包数字签名证书为 Google lnc 才是官方出品，否则是加料的恶意包。

1. 将安装包后缀exe改为 rar 或者 zip
2. 然后解压得到 chrome.7z 文件
3. 再次解压得到 Chrome-bin 文件夹

 最终得到想要的版本主体：

 ![](https://img2023.cnblogs.com/blog/1318601/202306/1318601-20230627163521546-292469590.png)

创建 user-data 空文件夹、chrome.exe 快捷方式

![](https://img2023.cnblogs.com/blog/1318601/202306/1318601-20230627163549737-927516151.png)

快捷方式右键选择属性, 在目标后面加入 `--user-data-dir` 属性：

- `--user-data-dir` 属性意思是：新建用户数据存放目录路径
- `--user-data-dir=D:/akong/app/Chrome/Chrome-64.0/Chrome-bin/user-data`

至此，就成功安装了一个Chrome版本，可通过快捷方式打开，想安装其他版本按以上流程走一遍即可。切记，不可同时打开多个不同版本的Chrome。