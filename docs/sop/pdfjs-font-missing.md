---
title: pdfjs-dist 加载发票文字丢失
date: 2023-08-09 23:16:14
tags: pdfs
categories: JavaScript
description: pdfjs-dist 加载发票文字丢失
---

# pdfjs-dist 加载发票文字丢失

控制台报错 缺少字体包
`Warning: Error during font loading: The CMap "baseUrl" parameter must be specified, ensure that the "cMapUrl" and "cMapPacked" API parameters are provided.`

原因：打开部分pdf文件，pdfjs-dist需要引入特殊字体。

看到网上解决方案，加载线上地址文件字体，直接改 node_modules 里的 pdf.js ：

```js
  params.rangeChunkSize = params.rangeChunkSize || DEFAULT_RANGE_CHUNK_SIZE;
  params.CMapReaderFactory = params.CMapReaderFactory || _display_utils.DOMCMapReaderFactory;
  params.ignoreErrors = params.stopAtErrors !== true;
  params.fontExtraProperties = params.fontExtraProperties === true;
  params.pdfBug = params.pdfBug === true;
  
  //解决文字缺少问题
  params.cMapPacked = true
  params.cMapUrl = 'https://cdn.jsdelivr.net/npm/pdfjs-dist@2.2.228/cmaps/'
```

比较扯，又找了另一种方法：

1. 把下面 node_modules/pdfjs-dist/cmaps/ 的 cmaps 目录直接拷贝到public目录下；
2. 把cMapUrl设置为 '/cmaps/'，从项目跟路径加载字体文件：
   ```js
    const DEFAULT_URL = pdf文件url
    const CMAP_URL = './cmaps/';
    const CMAP_PACKED = true;
    // ....
    // Loading document.
    const loadingTask = pdfjsLib.getDocument({
        url: DEFAULT_URL,
        cMapUrl: CMAP_URL,
        cMapPacked: CMAP_PACKED,
    });
    // ...
   ```

解决。