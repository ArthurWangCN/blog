---
title: 在Mac上轻松使用SVN
date: 2024-03-25 17:21:59
tags:
categories: 工程化
description: 大家都知道，在Mac或Linux环境下使用git比较方便，但有时候根剧项目要求又不得不使用SVN，在windows系统上面有我们最为熟悉的小乌龟（TortoiseSVN,下载链接：https://tortoisesvn.net/downloads.zh.html）在mac系统上面则很少svn的工具，本文就带大家对比Git，介绍如何在Mac上轻松使用命令行进行操作SVN，同时提升开发人员的格调。
---

# 在Mac上轻松使用SVN

**1、安装svn**

mac:

```bash
brew install svn
```

centos:

```bash
yum -y install subversion
```

**2、验证是否安装成功**

```bash
 svn --version
```

**3、拉取仓库文件**

通过 `svn checkout` 命令检出资源，svn checkout 可以使用缩写 svn co

```bash
svn checkout svn://xxxxxx
例子：
[root@s145 tmp]# svn checkout svn://192.168.0.146:18080/repos /tmp/svntest --username=testuser

#等同于
git clone    git@gitlab.*.com:gituser/*.com.git (fetch)
```

格式：

```bash
svn checkout http://路径(目录或文件的全路径)　[本地目录全路径] --username 用户名 --password 密码
```

**4、添加文件**

使用svn add命令添加前要求文件已存在，添加新文件只是告诉SVN，并没有真实提交，需要使用commit提交。

```bash
svn add file

#等同于

git add file
```

**5、提交文件到svn**

```bash
svn commit -m "LogMessage" [-N] [--no-unlock] PATH(如果选择了保持锁，就使用--no-unlock开关)

#等同于

git commit -m 'init提示信息' filepath
```

- -m参数为必选，可以为空，用于备注说明
- commit前必须先svn add添加文件到版本控制库。
- svn commit可以缩写为svn ci

