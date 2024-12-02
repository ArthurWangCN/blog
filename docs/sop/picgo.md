---
title: PicGo + github 搭建图床
date: 2022-05-08 22:14:00
tags: 
categories: 博客
description: Markdown 文档编写时可使用本地图片，但是无法在网络上使用。图床的作用可以理解为将文档中的图片放到网络上，直接引用网络地址，这样可以做到无论在那个平台都可以使用统一的图片地址。
---

# PicGo + github 搭建图床

**1. 在github上创建一个仓库。**

**2. 下载并安装 PicGo：**

官网：https://molunerfinn.com/PicGo/

下载：https://github.com/Molunerfinn/picgo/releases

**3. 配置PicGo：**

+ GitHub中创建一个token：依次打开 `Settings -> Developer settings -> Personal access tokens`，最后点击 `generate new token`。勾选相关信息（expiration 改为 no expiration、select scope 勾选repo），然后点击生成token；
+ 打开 PicGo，`图床设置 -> Github 图床`
+ 填写相关设置：

![](https://cdn.jsdelivr.net/gh/ArthurWangCN/PictureBed/使用Github搭建图床_imgs_47.png)

**4. 设置完成，可以上传图片测试了。**

**5. 设置cdn加速：**设置自定义域名为 `https://cdn.jsdelivr.net/gh/用户名/图床仓库名`。

![](https://cdn.jsdelivr.net/gh/ArthurWangCN/PictureBed/picGo设置自定义域名.png)
