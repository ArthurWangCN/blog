---
title: 使用 vuepress 生成静态网站
date: 2022-05-14 14:04:39
tags: 博客
categories: 博客
description: vuepress 是一个 Vue 驱动的静态网站生成器，它可以让你快速搭建一个文档/博客，后期只需要修改markdown内容和导航参数，即可一键打包生成页面。
---

# 使用 vuepress 生成静态网站

## vuepress 是什么

vuepress 是一个 Vue 驱动的**静态网站生成器**，它可以让你：

+ **快速搭建**一个文档/博客，后期只需要修改markdown内容和导航参数，即可一键打包生成页面。

+ 页面具有非常好的加载性能和搜索引擎优化（SEO），其他的页面则会只在用户浏览到的时候才按需加载。

+ 自动生成全局搜索、记录上次修改时间等功能。

+ 可嵌入vue组件或跳转至其他页面，可定制自己的样式模板便于扩展。

+ 一键免费部署到 github.pages，**无需服务器即可拥有自己的在线文档/博客**。



## 开始

### 初始化

1、全局安装vuepress

```bash
npm install -g vuepress
```

2、初始化项目

```bash
npm init -y
```

3、按 [官方](https://vuepress.vuejs.org/zh/guide/directory-structure.html) 规定创建文件和文件夹

```bash
# 下面没有文件类型后缀的都是文件夹
# 部分内容并不是必须的，想自己定制的话可以参考官方文档。这里是按照我的思路写的。
├── docs
│   ├── .vuepress  //存放核心内容的文件夹
│   │   ├── components  //存放你需要添加的vue组件
│   │   ├── public  //存放静态文件，如图片等
│   │   ├── styles  //存放需要定制的样式
│   │   │   └── palette.styl  //配置页面主题颜色的文件
│   │   └── config.js   //设定顶部导航栏、侧边导航栏等项目配置的核心文件
│   ├── pages   //存放markdown文件，用于设置其他页面内容
│   ├── README.md   //首页展示用的markdown文件
├── deploy.sh     //之后用于编写上传、发布脚本的文件
└── package.json  //之前创建的项目描述文件
```

4、添加启动命令

```json
"scripts": {
    "dev": "vuepress dev docs",         //用于实时预览
    "build": "vuepress build docs"      //用于打包项目
}
```



### 页面配置

#### 修改页面整体设置

首先我们先设定网页与浏览器标签栏相关的一些设置，直接在我们的 `config.js` 文件增添下面的代码。

```js
module.exports = {
    title: '这里是标题名', // 显示在左上角的网页名称以及首页在浏览器标签显示的title名称
    description: '这里是描述', // meta 中的描述文字，用于SEO
    // 注入到当前页面的 HTML <head> 中的标签
    head: [
        ['link', 
            { rel: 'icon', href: '/icon.png' }
            //浏览器的标签栏的网页图标，第一个'/'会遍历public文件夹的文件
        ],  
    ],
}
```

修改完成后，在 `docs/.vuepress/public` 文件夹里面放置我们的logo图片。

然后运行我们的 `npm run dev`，打开 `http://localhost:8080/` 就可以看到效果。

#### 设置首页内容

接下来我们来设置首页的内容，按照官网给的格式修改`README.md`文件，填写如下内容：

```markdown
---
home: true
heroImage: /icon.png
heroText: xxx
tagline: xxx
actionText: 马上进入 →
actionLink: /xxx
features:
- title: 简洁至上
  details: 以 Markdown 为中心的项目结构，以最少的配置帮助你专注于写作。
- title: Vue驱动
  details: 享受 Vue + webpack 的开发体验，在 Markdown 中使用 Vue 组件，同时可以使用 Vue 来开发自定义主题。
- title: 高性能
  details: VuePress 为每个页面预渲染生成静态的 HTML，同时在页面被加载的时候，将作为 SPA 运行。
---
```

> 注意：：heroImage的地址配置第一个'/'默认指向的是 docs/.vuepress/public，你需要在此文件夹放置你的首页图片。 actionLink地址配置第一个'/'默认指向的是 docs/，若未路径文件不存在点击进去会跳转至404。

#### 修改页面导航栏、侧边导航栏

导航栏修改和侧边栏修改还是在我们的 `config.js` 文件进行修改，在之前添加的moudule.exports里添加如下代码

```js
module.exports = {
    //...省略部分代码
    
    //下面涉及到的md文件和其他文件的路径下一步再详细解释
    themeConfig: {
        logo: '/egg.png',  //网页顶端导航栏左上角的图标
        
        //顶部导航栏
        nav: [           
            //格式一：直接跳转，'/'为不添加路由，跳转至首页
            { text: '首页', link: '/' },    
            
            //格式二：添加下拉菜单，link指向的文件路径
            {
                text: '分类',  //默认显示        
                ariaLabel: '分类',   //用于识别的label
                items: [
                    { text: '文章', link: '/pages/folder1/test1.md' },  
                    //点击标签会跳转至link的markdown文件生成的页面
                    { text: '琐碎', link: '/pages/folder2/test4.md' },
                ]
            },
            { text: '功能演示', link: '/pages/folder1/test3.md' },
            
            //格式三：跳转至外部网页，需http/https前缀
            { text: 'Github', link: 'https://github.com/dwanda' },
        ],
        
        //侧边导航栏：会根据当前的文件路径是否匹配侧边栏数据，自动显示/隐藏
        sidebar: {
            '/pages/folder1/':[         
                {
                    title: '测试菜单1',   // 一级菜单名称
                    collapsable: false, // false为默认展开菜单, 默认值true是折叠,
                    sidebarDepth: 1,    //  设置侧边导航自动提取markdown文件标题的层级，默认1为h2层级
                    children: [
                        ['test1.md', '子菜单1'],  //菜单名称为'子菜单1'，跳转至/pages/folder1/test1.md
                        ['test3.md', '子菜单2']
                    ]
                },
                {
                    title: '测试菜单2',
                    collapsable: false, 
                    children: [
                        ['test2.md', '子菜单1']
                    ]
                }
            ],
            
            //...可添加多个不同的侧边栏，不同页面会根据路径显示不同的侧边栏
        }
    }
}
```



## 一键部署至github page

在你的项目中，创建一个如下的 `deploy.sh` 文件：

```shell
#!/usr/bin/env sh

# 确保脚本抛出遇到的错误
set -e

# 生成静态文件
npm run docs:build

# 进入生成的文件夹
cd docs/.vuepress/dist

# 如果是发布到自定义域名
# echo 'www.example.com' > CNAME

git init
git add -A
git commit -m 'deploy'

# 如果发布到 https://<USERNAME>.github.io
# git push -f git@github.com:<USERNAME>/<USERNAME>.github.io.git master

# 如果发布到 https://<USERNAME>.github.io/<REPO>
# git push -f git@github.com:<USERNAME>/<REPO>.git master:gh-pages

cd -
```

`package.json` 中配置命令执行此shell文件：

```json
"scripts": {
  "deploy": "bash deploy.sh"
}
```



## 其他

### 热编译浏览器不自动更新

本地开发模式运行Vuepress 1.x 时，浏览器不能自动更新:

在 package.json 中将运行命令 由

```json
"dev": "vuepress dev docs"
```

改为

```json
"dev": "vuepress dev docs --temp .temp"
```

即可解决。

> 注意：运行 vuepress 会生成一个临时文件夹 .temp，可以在 .gitignore 中忽略掉该文件夹



## 参考

1. [VuePress](https://vuepress.vuejs.org/)
2. [1小时搞定vuepress](https://juejin.cn/post/6844903999129436174#heading-9)
