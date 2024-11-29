---
title: NexT添加友情链接
date: 2022-05-25 00:10:37
tags:
categories: 博客
description: 最简单的友情链接页面定制
---

## 新建一个post

在 source 文件夹下建一个文件夹 links，里面放一个文件 index.md，即友链页面：

```html
---
title: 友情链接
date: 2021-06-16 00:34:27
---

<div class="post-body">
   <div id="links">
      <style>
         .links-content{
         margin-top:1rem;
         }
         .link-navigation::after {
         content: " ";
         display: block;
         clear: both;
         }
         .card {
         width: 45%;
         font-size: 1rem;
         padding: 10px 20px;
         border-radius: 4px;
         transition-duration: 0.15s;
         margin-bottom: 1rem;
         display:flex;
         }
         .card:nth-child(odd) {
         float: left;
         }
         .card:nth-child(even) {
         float: right;
         }
         .card:hover {
         transform: scale(1.1);
         box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.12), 0 0 6px 0 rgba(0, 0, 0, 0.04);
         }
         .card a {
         border:none;
         }
         .card .ava {
         width: 3rem!important;
         height: 3rem!important;
         margin:0!important;
         margin-right: 1em!important;
         border-radius:4px;
         }
         .card .card-header {
         font-style: italic;
         overflow: hidden;
         width: 100%;
         }
         .card .card-header a {
         font-style: normal;
         color: #2bbc8a;
         font-weight: bold;
         text-decoration: none;
         }
         .card .card-header a:hover {
         color: #d480aa;
         text-decoration: none;
         }
         .card .card-header .info {
         font-style:normal;
         color:#a3a3a3;
         font-size:14px;
         min-width: 0;
         overflow: hidden;
         white-space: nowrap;
         }
      </style>
      <div class="links-content">
         <div class="link-navigation">
            <div class="card">
               <img class="ava" src="https://cdn.jsdelivr.net/gh/hvnobug/assets/common/avatar.png" />
               <div class="card-header">
                  <div>
                     <a href="https://blog.hvnobug.com/">Emil’s blog</a>
                  </div>
                  <div class="info">这是一个分享IT技术的小站。</div>
               </div>
            </div>
            <div class="card">
               <img class="ava" src="https://yingwiki.top/avatar" />
               <div class="card-header">
                  <div>
                     <a href="https://yingwiki.top">越行勤's Blog</a>
                  </div>
                  <div class="info">努力学习的小菜鸟</div>
               </div>
            </div>
         </div>
      </div>
   </div>
</div>
```

其实就是一段css再接上几个div，以后每加一个友链，只需要复制如下一段即可：

```html
<div class="card">
   <img class="ava" src="{avatarurl}" />
   <div class="card-header">
      <div>
         <a href="{link}">{name}</a>
      </div>
      <div class="info">{description}</div>
   </div>
</div>
```

之后在每次 `hexo g` 时，就会生成一个 `links/index.html` 文件。同时由于它不在  `source/_posts` 文件中，hexo不会把它当成一篇新博客加入首页。

## 增加菜单项

在NexT主题的配置文件 `_config.yml` 中找到 `menu` 部分，加一行即可：

```none
...
menu:
  tags: /tags/ || tags
  categories: /categories/ || th
...
  Links: /links/ || link
...
```

