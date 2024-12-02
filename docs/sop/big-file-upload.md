---
title: 大文件上传实现
date: 2023-03-18 22:23:15
tags: JavaScript
categories: JavaScript
description: 本文从零搭建前端和服务端，实现一个大文件上传的 demo
---

# 大文件上传实现

本文从零搭建前端和服务端，实现一个大文件上传的 demo

前端：Vue@2 + Element-ui

服务端：Nodejs@14 + multiparty

## 整体思路

### 前端

前端大文件上传网上的大部分文章已经给出了解决方案，核心是利用 `Blob.prototype.slice` 方法，和数组的 slice 方法相似，文件的 slice 方法可以返回原文件的某个切片。

预先定义好单个切片大小，将文件切分为一个个切片，然后借助 http 的可并发性，同时上传多个切片。这样从原本传一个大文件，变成了并发传多个小的文件切片，可以大大减少上传时间。

另外由于是并发，传输到服务端的顺序可能会发生变化，因此我们还需要给每个切片记录顺序。

### 服务端

服务端负责接受前端传输的切片，并在接收到所有切片后合并所有切片

这里又引伸出两个问题

1. 何时合并切片，即切片什么时候传输完成
2. 如何合并切片

第一个问题需要前端配合，前端在每个切片中都携带切片最大数量的信息，当服务端接受到这个数量的切片时自动合并。或者也可以额外发一个请求，主动通知服务端进行切片的合并。

第二个问题，具体如何合并切片呢？这里可以使用 Nodejs 的 读写流（readStream/writeStream），将所有切片的流传输到最终文件的流里。

## 前端部分
前端使用 Vue 作为开发框架，对界面没有太大要求，原生也可以，考虑到美观使用 Element-ui 作为 UI 框架

### 上传控件

首先创建选择文件的控件并监听 change 事件，另外就是上传按钮

```html
<template>
   <div>
    <input type="file" @change="handleFileChange" />
    <el-button @click="handleUpload">upload</el-button>
  </div>
</template>

<script>
export default {
  data: () => ({
    container: {
      file: null
    }
  }),
  methods: {
     handleFileChange(e) {
      const [file] = e.target.files;
      if (!file) return;
      Object.assign(this.$data, this.$options.data());
      this.container.file = file;
    },
    async handleUpload() {}
  }
};
</script>
```

### 请求逻辑

考虑到通用性，这里没有用第三方的请求库，而是用原生 XMLHttpRequest 做一层简单的封装来发请求

```js
request({
      url,
      method = "post",
      data,
      headers = {},
      requestList
    }) {
      return new Promise(resolve => {
        const xhr = new XMLHttpRequest();
        xhr.open(method, url);
        Object.keys(headers).forEach(key =>
          xhr.setRequestHeader(key, headers[key])
        );
        xhr.send(data);
        xhr.onload = e => {
          resolve({
            data: e.target.response
          });
        };
      });
    }
```

### 上传切片

接着实现比较重要的上传功能，上传需要做两件事

1. 对文件进行切片
2. 将切片传输给服务端

```diff
<template>
  <div>
    <input type="file" @change="handleFileChange" />
    <el-button @click="handleUpload">上传</el-button>
  </div>
</template>

<script>
+ // 切片大小
+ // the chunk size
+ const SIZE = 10 * 1024 * 1024; 

export default {
  data: () => ({
    container: {
      file: null
    }，
+   data: []
  }),
  methods: {
    request() {},
    handleFileChange() {},
+    // 生成文件切片
+    createFileChunk(file, size = SIZE) {
+     const fileChunkList = [];
+      let cur = 0;
+      while (cur < file.size) {
+        fileChunkList.push({ file: file.slice(cur, cur + size) });
+        cur += size;
+      }
+      return fileChunkList;
+    },
+   // 上传切片
+    async uploadChunks() {
+      const requestList = this.data
+        .map(({ chunk，hash }) => {
+          const formData = new FormData();
+          formData.append("chunk", chunk);
+          formData.append("hash", hash);
+          formData.append("filename", this.container.file.name);
+          return { formData };
+        })
+        .map(({ formData }) =>
+          this.request({
+            url: "http://localhost:3000",
+            data: formData
+          })
+        );
+      // 并发请求
+      await Promise.all(requestList); 
+    },
+    async handleUpload() {
+      if (!this.container.file) return;
+      const fileChunkList = this.createFileChunk(this.container.file);
+      this.data = fileChunkList.map(({ file }，index) => ({
+        chunk: file,
+        // 文件名 + 数组下标
+        hash: this.container.file.name + "-" + index
+      }));
+      await this.uploadChunks();
+    }
  }
};
</script>
```

当点击上传按钮时，调用 createFileChunk 将文件切片，切片数量通过文件大小控制，这里设置 10MB，也就是说一个 100 MB 的文件会被分成 10 个 10MB 的切片。

createFileChunk 内使用 while 循环和 slice 方法将切片放入 fileChunkList 数组中返回。

在生成文件切片时，需要给每个切片一个标识作为 hash，这里暂时使用文件名 + 下标，这样后端可以知道当前切片是第几个切片，用于之后的合并切片。

随后调用 uploadChunks 上传所有的文件切片，将文件切片，切片 hash，以及文件名放入 formData 中，再调用上一步的 request 函数返回一个 proimise，最后调用 Promise.all 并发上传所有的切片。

### 发送合并请求

使用整体思路中提到的第二种合并切片的方式，即前端主动通知服务端进行合并

前端发送额外的合并请求，服务端接受到请求时合并切片

```diff
<template>
  <div>
    <input type="file" @change="handleFileChange" />
    <el-button @click="handleUpload">upload</el-button>
  </div>
</template>

<script>
export default {
  data: () => ({
    container: {
      file: null
    },
    data: []
  }),
  methods: {
    request() {},
    handleFileChange() {},
    createFileChunk() {},
    async uploadChunks() {
      const requestList = this.data
        .map(({ chunk，hash }) => {
          const formData = new FormData();
          formData.append("chunk", chunk);
          formData.append("hash", hash);
          formData.append("filename", this.container.file.name);
          return { formData };
        })
        .map(({ formData }) =>
          this.request({
            url: "http://localhost:3000",
            data: formData
          })
        );
      await Promise.all(requestList);
+     // 合并切片
+     await this.mergeRequest();
    },
+    async mergeRequest() {
+      await this.request({
+        url: "http://localhost:3000/merge",
+        headers: {
+          "content-type": "application/json"
+        },
+        data: JSON.stringify({
+          filename: this.container.file.name
+        })
+      });
+    },    
    async handleUpload() {}
  }
};
</script>
```

## 服务端部分

使用 http 模块搭建一个简单服务端

```js
const http = require("http");
const server = http.createServer();

server.on("request", async (req, res) => {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Headers", "*");
  if (req.method === "OPTIONS") {
    res.status = 200;
    res.end();
    return;
  }
});

server.listen(3000, () => console.log("listening port 3000"));
```

### 接受切片

使用 multiparty 处理前端传来的 formData

在 multiparty.parse 的回调中，files 参数保存了 formData 中文件，fields 参数保存了 formData 中非文件的字段

```diff
const http = require("http");
const path = require("path");
+ const fse = require("fs-extra");
+ const multiparty = require("multiparty");

const server = http.createServer();
+ // 大文件存储目录
+ const UPLOAD_DIR = path.resolve(__dirname, "..", "target");

server.on("request", async (req, res) => {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Headers", "*");
  if (req.method === "OPTIONS") {
    res.status = 200;
    res.end();
    return;
  }

+  const multipart = new multiparty.Form();

+  multipart.parse(req, async (err, fields, files) => {
+    if (err) {
+      return;
+    }
+    const [chunk] = files.chunk;
+    const [hash] = fields.hash;
+    const [filename] = fields.filename;
+    // 创建临时文件夹用于临时存储 chunk
+    // 添加 chunkDir 前缀与文件名做区分
+    const chunkDir = path.resolve(UPLOAD_DIR, 'chunkDir' + filename);

+    if (!fse.existsSync(chunkDir)) {
+      await fse.mkdirs(chunkDir);
+    }

+    // fs-extra 的 rename 方法 windows 平台会有权限问题
+    // @see https://github.com/meteor/meteor/issues/7852#issuecomment-255767835
+    await fse.move(chunk.path, `${chunkDir}/${hash}`);
+    res.end("received file chunk");
+  });
});

server.listen(3000, () => console.log("listening port 3000"));
```

查看 multiparty 处理后的 chunk 对象，path 是存储临时文件的路径，size 是临时文件大小，在 multiparty 文档中提到可以使用 fs.rename（这里换成了 fs.remove, 因为 fs-extra 的 rename 方法在 windows 平台存在权限问题）

在接受文件切片时，需要先创建临时存储切片的文件夹，以 chunkDir 作为前缀，文件名作为后缀。

由于前端在发送每个切片时额外携带了唯一值 hash，所以以 hash 作为文件名，将切片从临时路径移动切片文件夹中。

### 合并切片

在接收到前端发送的合并请求后，服务端将文件夹下的所有切片进行合并

```diff
const http = require("http");
const path = require("path");
const fse = require("fs-extra");

const server = http.createServer();
const UPLOAD_DIR = path.resolve(__dirname, "..", "target");

+ const resolvePost = req =>
+   new Promise(resolve => {
+     let chunk = "";
+     req.on("data", data => {
+       chunk += data;
+     });
+     req.on("end", () => {
+       resolve(JSON.parse(chunk));
+     });
+   });

+ // 写入文件流
+ const pipeStream = (path, writeStream) =>
+  new Promise(resolve => {
+    const readStream = fse.createReadStream(path);
+    readStream.on("end", () => {
+      fse.unlinkSync(path);
+      resolve();
+    });
+    readStream.pipe(writeStream);
+  });

// 合并切片
+ const mergeFileChunk = async (filePath, filename, size) => {
+   const chunkDir = path.resolve(UPLOAD_DIR, 'chunkDir' + filename);
+   const chunkPaths = await fse.readdir(chunkDir);
+   // 根据切片下标进行排序
+   // 否则直接读取目录的获得的顺序会错乱
+   chunkPaths.sort((a, b) => a.split("-")[1] - b.split("-")[1]);
+   // 并发写入文件
+   await Promise.all(
+     chunkPaths.map((chunkPath, index) =>
+       pipeStream(
+         path.resolve(chunkDir, chunkPath),
+         // 根据 size 在指定位置创建可写流
+         fse.createWriteStream(filePath, {
+           start: index * size,
+         })
+       )
+     )
+  );
+  // 合并后删除保存切片的目录
+  fse.rmdirSync(chunkDir);
+};

server.on("request", async (req, res) => {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Headers", "*");
  if (req.method === "OPTIONS") {
    res.status = 200;
    res.end();
    return;
  }

+   if (req.url === "/merge") {
+     const data = await resolvePost(req);
+     const { filename,size } = data;
+     const filePath = path.resolve(UPLOAD_DIR, `${filename}`);
+     await mergeFileChunk(filePath, filename);
+     res.end(
+       JSON.stringify({
+         code: 0,
+         message: "file merged success"
+       })
+     );
+   }

});

server.listen(3000, () => console.log("listening port 3000"));
```

由于前端在发送合并请求时会携带文件名，服务端根据文件名可以找到上一步创建的切片文件夹

接着使用 fs.createWriteStream 创建一个可写流，可写流文件名就是上传时的文件名

随后遍历整个切片文件夹，将切片通过 fs.createReadStream 创建可读流，传输合并到目标文件中

值得注意的是每次可读流都会传输到可写流的指定位置，这是通过 createWriteStream 的第二个参数 start 控制的，目的是能够并发合并多个可读流至可写流中，这样即使并发时流的顺序不同，也能传输到正确的位置

所以还需要让前端在请求的时候提供之前设定好的 size 给服务端，服务端根据 size 指定可读流的起始位置

```diff
  async mergeRequest() {
     await this.request({
       url: "http://localhost:3000/merge",
       headers: {
         "content-type": "application/json"
       },
       data: JSON.stringify({
+        size: SIZE,
         filename: this.container.file.name
       })
     });
   },
```

其实也可以等上一个切片合并完后再合并下个切片，这样就不需要指定位置，但传输速度会降低，所以使用了并发合并的手段。接着只要保证每次合并完成后删除这个切片，等所有切片都合并完毕后最后删除切片文件夹即可

至此一个简单的大文件上传就完成了，接下来我们再此基础上扩展一些额外的功能。


## 显示上传进度条
上传进度分两种，一个是每个切片的上传进度，另一个是整个文件的上传进度，而整个文件的上传进度是基于每个切片上传进度计算而来，所以我们先实现单个切片的进度条。

### 单个切片进度条

XMLHttpRequest 原生支持上传进度的监听，只需要监听 upload.onprogress 即可，我们在原来的 request 基础上传入 onProgress 参数，给 XMLHttpRequest 注册监听事件

```diff
 // xhr
    request({
      url,
      method = "post",
      data,
      headers = {},
+     onProgress = e => e,
      requestList
    }) {
      return new Promise(resolve => {
        const xhr = new XMLHttpRequest();
+       xhr.upload.onprogress = onProgress;
        xhr.open(method, url);
        Object.keys(headers).forEach(key =>
          xhr.setRequestHeader(key, headers[key])
        );
        xhr.send(data);
        xhr.onload = e => {
          resolve({
            data: e.target.response
          });
        };
      });
    }
```

由于每个切片都需要触发独立的监听事件，所以需要一个工厂函数，根据传入的切片返回不同的监听函数

在原先的前端上传逻辑中新增监听函数部分

```diff
    // 上传切片，同时过滤已上传的切片
    async uploadChunks(uploadedList = []) {
      const requestList = this.data
+       .map(({ chunk,hash,index }) => {
          const formData = new FormData();
          formData.append("chunk", chunk);
          formData.append("hash", hash);
          formData.append("filename", this.container.file.name);
+         return { formData,index };
        })
+       .map(({ formData,index }) =>
          this.request({
            url: "http://localhost:3000",
            data: formData，
+           onProgress: this.createProgressHandler(this.data[index]),
          })
        );
      await Promise.all(requestList);
      await this.mergeRequest();
    },
    async handleUpload() {
      if (!this.container.file) return;
      const fileChunkList = this.createFileChunk(this.container.file);
      this.data = fileChunkList.map(({ file }，index) => ({
        chunk: file,
+       index,
        hash: this.container.file.name + "-" + index
+       percentage:0
      }));
      await this.uploadChunks();
    }    
+   createProgressHandler(item) {
+      return e => {
+        item.percentage = parseInt(String((e.loaded / e.total) * 100));
+      };
+   }
```

每个切片在上传时都会通过监听函数更新 data 数组对应元素的 percentage 属性，之后把将 data 数组放到视图中展示即可。

### 总进度条

将每个切片已上传的部分累加，除以整个文件的大小，就能得出当前文件的上传进度，所以这里使用 Vue 的计算属性

```js
 computed: {
      uploadPercentage() {
         if (!this.container.file || !this.data.length) return 0;
         const loaded = this.data
           .map(item => item.size * item.percentage)
           .reduce((acc, cur) => acc + cur);
         return parseInt((loaded / this.container.file.size).toFixed(2));
       }
}
```

## 总结

大文件上传：

- 前端上传大文件时使用 Blob.prototype.slice 将文件切片，并发上传多个切片，最后发送一个合并的请求通知服务端合并切片
- 服务端接收切片并存储，收到合并请求后使用流将切片合并到最终文件
- 原生 XMLHttpRequest 的 upload.onprogress 对切片上传进度的监听
- 使用 Vue 计算属性根据每个切片的进度算出整个文件的上传进度


参考 https://juejin.cn/post/6844904046436843527#heading-24