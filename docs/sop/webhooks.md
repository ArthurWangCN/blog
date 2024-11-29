---
title: 宝塔利用 Git + WebHook 实现与码云同步【自动部署】
date: 2024-03-08 13:47:54
tags:
categories: 博客
description: WebHook 功能是帮助用户 push 代码后，自动回调一个您设定的 http 地址。
---

WebHook 功能是帮助用户 push 代码后，自动回调一个您设定的 http 地址。

这是一个通用的解决方案，用户可以自己根据不同的需求，来编写自己的脚本程序(比如发邮件，自动部署等)。



**一、安装宝塔webhook**

![](https://developer.qcloudimg.com/http-save/yehe-7130271/849a248a36425cc424c60a8ab1696cda.jpg)

安装后添加一个webhook，在webhook中添加想要执行的脚本：

![](https://developer.qcloudimg.com/http-save/yehe-7130271/69ab98109587b77953d5e71cc63ceddb.jpg)



**二、配置gitee WebHooks**

复制宝塔的 WebHook 提供的URL和密钥

![](https://developer.qcloudimg.com/http-save/yehe-7130271/51a5a81b053c5ba0ee85959ea8229e3e.png)

在 gitee 仓库的 WebHook 中添加 WebHook

![](https://developer.qcloudimg.com/http-save/yehe-7130271/ff5f3ad988ce8bc651387b10a959600f.jpg)



**三、测试同步**

码云仓库随意改个文件保存一下, 生成新的提交记录, 看云服务器上有没有同步更新