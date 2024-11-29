---
title: 一个基于GO的局域网文件互传工具
date: 2022-05-14 14:31:27
tags: golang
categories: 项目
description: 一个基于GO语言的「局域网PC与手机互传文件，且不想借助微信/QQ等骚扰软件」的软件
---

## 前言

最近在学习 Go 语言，语法看得差不多了，想着找点什么项目做做，正好我一直想做一个「局域网PC与手机互传文件，且不想借助微信/QQ等骚扰软件」的软件，于是就用 Go 来做了，最终截图如下：

![闪电传](http://blogpic.at15cm.com/flash_files_1.png)

![闪电传](http://blogpic.at15cm.com/flash_files_1.png)

![闪电传](http://blogpic.at15cm.com/flash_files_3.png)

功能很简单：

1. PC 上传文字或文件后创建二维码，提供给手机浏览器扫码下载
2. 手机在浏览器上传文字或文件后自动用 websocket 通知给 PC 端，弹出下载提示

[源码在此](https://github.com/ArthurWangCN/flash-files)，这里主要说一下实现思路。



## 工具

+ VSCode或者GoLand

+ zserge/lorca：一个制作桌面应用的库

+ React

+ gin-gonic/gin：提供服务器接口

+ gorilla/websocket：实现websocket通知

+ skip2/go-qrcode：生成二维码

+ go

  版本：1.17或以上。

  需要做的配置：

  ```shell
  go env -w GO111MODULE=on
  go env -w GOPROXY=https://goproxy.cn,direct
  ```

  安装gowatch

  ```shell
  go get github.com/silenceper/gowatch
  ```



## 整体思路

![闪电传](http://blogpic.at15cm.com/flash_files_4.png)

1. 用 Lorca 搞出一个窗口
2. 用 HTML 制作界面，用 JS 调用后台接口
3. 用 Gin 实现后台接口
4. 上传的文件都放到 uploads 文件夹中



## 用 Lorca 创建窗口

我了解到 Go 的如下库可以实现窗口：

1. [lorca](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fzserge%2Florca) - 调用系统现有的 Chrome/Edge 实现简单的窗口，UI 通过 HTML/CSS/JS 实现
2. [webview](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fwebview%2Fwebview) - 比 lorca 功能更强，实现 UI 的思路差不多
3. [fyne](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ffyne-io%2Ffyne) - 使用 Canvas 绘制的 UI 框架，性能不错
4. [qt](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ftherecipe%2Fqt) - 更复杂更强大的 UI 框架

我随便挑了个最简单的 Lorca 就开始了。

示例代码：

```go
var ui lorca.UI
ui, _ = lorca.New("http://www.baidu.com", "", 800, 600, "--disable-sync", "--disable-translate")
```



## 前端制作 UI

我用 React + ReactRouter 来实现页面结构，文件上传和对话框是自己用原生 JS 写的，UI 细节用 CSS3 来做，没有依赖其他 UI 组件库。

Lorca 的主要功能就是展示我写的 index.html。



## 用 gin 实现后台接口

前端项目用到了五个接口，我使用 gin 来实现：

```go
router.GET("/uploads/:path", controllers.UploadsController)              
router.GET("/api/v1/addresses", controllers.AddressesController) 
router.GET("/api/v1/qrcodes", controllers.QrcodesController)   
router.POST("/api/v1/files", controllers.FilesController)      
router.POST("/api/v1/texts", controllers.TextsController)
```

其中的二维码生成我用的是 [go-qrcode](https://github.com/skip2/go-qrcode)。



## 用 [gorilla/websocket](https://github.com/gorilla/websocket) 实现手机通知 PC



## 给exe文件添加图标

正常 `go build` 后exe文件是默认图标，但可以设置自己想要的图标。

网上找了一些文章，花里胡哨的，这里简单粗暴。需要一个名为 **rsrc** 工具：

1. 克隆rsrc源代码，并打包生成exe文件

   ```bash
   git clone https://github.com/akavel/rsrc.git
   ```

   ```bash
   cd rsrc
   go build
   ```

   生成 **rsrc.exe**，并拷贝到当前项目根目录。

2. 根目录创建配置文件main.manifest：

   ```xml
   <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
   <assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
   <assemblyIdentity
       version="1.0.0.0"
       processorArchitecture="x86"
       name="controls"
       type="win32"
   ></assemblyIdentity>
   <dependency>
       <dependentAssembly>
           <assemblyIdentity
               type="win32"
               name="Microsoft.Windows.Common-Controls"
               version="6.0.0.0"
               processorArchitecture="*"
               publicKeyToken="6595b64144ccf1df"
               language="*"
           ></assemblyIdentity>
       </dependentAssembly>
   </dependency>
   </assembly>
   ```

3. 制作你想要的图标favicon.ico，并放到根目录；

4. 生成syso文件（我们最终只需要这个文件）：

   ```bash
   ./rsrc.exe -manifest main.manifest -ico favicon.ico -o main.syso
   ```

5. `go build -ldflags="-H windowsgui"` 得到exe文件，此时生成的exe文件图标就是自己设置的。
