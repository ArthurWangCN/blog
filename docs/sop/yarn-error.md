---
title: yarn遇到的问题
date: 2024-01-30 11:56:13
tags: 工程化
categories: 工程化
description: yarn安装过程中遇到的问题以及解决方案
---

# yarn遇到的问题

## 报错 certificate has expired

在通过yarn包管理器安装 yarn install 时候

报错：

```shell
info No lockfile found.
[1/4] Resolving packages...
error Error: certificate has expired
    at TLSSocket.onConnectSecure (node:_tls_wrap:1539:34)
    at TLSSocket.emit (node:events:513:28)
    at TLSSocket._finishInit (node:_tls_wrap:953:8)
    at TLSWrap.ssl.onhandshakedone (node:_tls_wrap:734:12)
info Visit https://yarnpkg.com/en/docs/cli/install for documentation about this command.
```

出现这个问题的原因：

开始以为是下载源的问题，但是切换到淘宝源后依然无法解决问题，还是报这个问题。并且自己通过npm包管理器安装时，也会报同样的错误

各种查找了一圈，发现是【HTTPS 证书验证失败】导致的

解决方案：

将yarn配置中的 strict-ssl 设置为 flase , 在 info yarn config 信息中， 'strict-ssl' 为 true，表示需要验证 HTTPS 证书。我们可以将 'strict-ssl' 设置为 false，跳过 HTTPS 证书验证。

设置命令如下：

1. 首先通过 `yarn config list` 查看yarn的配置清单里的strict-ssl：
2. 使用命令 `yarn config set strict-ssl false` 将其改为 false 即可
3. 再次运行安装命令即可顺利安装



