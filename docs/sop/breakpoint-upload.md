---
title: 断点续传实现
date: 2023-03-20 20:25:12
tags: JavaScript
categories: JavaScript
description: 断点续传的原理在于前端/服务端需要记住已上传的切片，这样下次上传就可以跳过之前已上传的部分
---

断点续传的原理在于前端/服务端需要记住已上传的切片，这样下次上传就可以跳过之前已上传的部分，有两种方案实现记忆的功能

- 前端使用 localStorage 记录已上传的切片 hash
- 服务端保存已上传的切片 hash，前端每次上传前向服务端获取已上传的切片

第一种是前端的解决方案，第二种是服务端，而前端方案有一个缺陷，如果换了个浏览器就失去了记忆的效果，所以这里选后者

## 生成 hash

无论是前端还是服务端，都必须要生成文件和切片的 hash，之前我们使用文件名 + 切片下标作为切片 hash，这样做文件名一旦修改就失去了效果，而事实上只要文件内容不变，hash 就不应该变化，所以正确的做法是根据文件内容生成 hash，所以我们修改一下 hash 的生成规则

webpack 的产物 contenthash 也是基于这个思路实现的

这里用到另一个库 spark-md5，它可以根据文件内容计算出文件的 hash 值

另外考虑到如果上传一个超大文件，读取文件内容计算 hash 是非常耗费时间的，并且会引起 UI 的阻塞，导致页面假死状态，所以我们使用 web-worker 在 worker 线程计算 hash，这样用户仍可以在主界面正常的交互。

由于实例化 web-worker 时，参数是一个 js 文件路径且不能跨域，所以我们单独创建一个 hash.js 文件放在 public 目录下，另外在 worker 中也是不允许访问 dom 的，但它提供了importScripts 函数用于导入外部脚本，通过它导入 spark-md5

```js
// /public/hash.js

// 导入脚本
self.importScripts("/spark-md5.min.js");

// 生成文件 hash
self.onmessage = e => {
  const { fileChunkList } = e.data;
  const spark = new self.SparkMD5.ArrayBuffer();
  let percentage = 0;
  let count = 0;
  const loadNext = index => {
    const reader = new FileReader();
    reader.readAsArrayBuffer(fileChunkList[index].file);
    reader.onload = e => {
      count++;
      spark.append(e.target.result);
      if (count === fileChunkList.length) {
        self.postMessage({
          percentage: 100,
          hash: spark.end()
        });
        self.close();
      } else {
        percentage += 100 / fileChunkList.length;
        self.postMessage({
          percentage
        });
        // calculate recursively
        loadNext(count);
      }
    };
  };
  loadNext(0);
};
```

在 worker 线程中，接受文件切片 fileChunkList，利用 fileReader 读取每个切片的 ArrayBuffer 并不断传入 spark-md5 中，每计算完一个切片通过 postMessage 向主线程发送一个进度事件，全部完成后将最终的 hash 发送给主线程。

spark-md5 文档中要求传入所有切片并算出 hash 值，不能直接将整个文件放入计算，否则即使不同文件也会有相同的 hash。

接着编写主线程与 worker 线程通讯的逻辑

```diff
+    // 生成文件 hash（web-worker）
+    calculateHash(fileChunkList) {
+      return new Promise(resolve => {
+        // 添加 worker 属性
+        this.container.worker = new Worker("/hash.js");
+        this.container.worker.postMessage({ fileChunkList });
+        this.container.worker.onmessage = e => {
+          const { percentage, hash } = e.data;
+          this.hashPercentage = percentage;
+          if (hash) {
+            resolve(hash);
+          }
+        };
+      });
    },
    async handleUpload() {
      if (!this.container.file) return;
      const fileChunkList = this.createFileChunk(this.container.file);
+     this.container.hash = await this.calculateHash(fileChunkList);
      this.data = fileChunkList.map(({ file }，index) => ({
+       fileHash: this.container.hash,
        chunk: file,
        hash: this.container.file.name + "-" + index,
        percentage:0
      }));
      await this.uploadChunks();
    }   
```

主线程使用 postMessage 给 worker 线程传入所有切片 fileChunkList，并监听 worker 线程发出的 postMessage 事件拿到文件 hash，加上显示计算 hash 的进度条。

至此前端需要将之前用文件名作为 hash 的地方改写为 worker 返回的 hash。

服务端则使用固定前缀 + hash 作为切片文件夹名，hash + 下标作为切片名，hash + 扩展名作为文件名。

## 文件秒传

在实现断点续传前先简单介绍一下文件秒传

所谓的文件秒传，即在服务端已经存在了上传的资源，所以当用户再次上传时会直接提示上传成功

文件秒传需要依赖上一步生成的 hash，即在上传前，先计算出文件 hash，并把 hash 发送给服务端进行验证，由于 hash 的唯一性，所以一旦服务端能找到 hash 相同的文件，则直接返回上传成功的信息即可

```diff
+    async verifyUpload(filename, fileHash) {
+       const { data } = await this.request({
+         url: "http://localhost:3000/verify",
+         headers: {
+           "content-type": "application/json"
+         },
+         data: JSON.stringify({
+           filename,
+           fileHash
+         })
+       });
+       return JSON.parse(data);
+     },
   async handleUpload() {
      if (!this.container.file) return;
      const fileChunkList = this.createFileChunk(this.container.file);
      this.container.hash = await this.calculateHash(fileChunkList);
+     const { shouldUpload } = await this.verifyUpload(
+       this.container.file.name,
+       this.container.hash
+     );
+     if (!shouldUpload) {
+       this.$message.success("skip upload：file upload success");
+       return;
+    }
     this.data = fileChunkList.map(({ file }, index) => ({
        fileHash: this.container.hash,
        index,
        hash: this.container.hash + "-" + index,
        chunk: file,
        percentage: 0
      }));
      await this.uploadChunks();
    }   
```

秒传其实就是给用户看的障眼法，实质上根本没有上传

服务端的逻辑非常简单，新增一个验证接口，验证文件是否存在即可

```diff
+ // 提取后缀名
+ const extractExt = filename =>
+  filename.slice(filename.lastIndexOf("."), filename.length);
const UPLOAD_DIR = path.resolve(__dirname, "..", "target");

const resolvePost = req =>
  new Promise(resolve => {
    let chunk = "";
    req.on("data", data => {
      chunk += data;
    });
    req.on("end", () => {
      resolve(JSON.parse(chunk));
    });
  });

server.on("request", async (req, res) => {
  if (req.url === "/verify") {
+    const data = await resolvePost(req);
+    const { fileHash, filename } = data;
+    const ext = extractExt(filename);
+    const filePath = path.resolve(UPLOAD_DIR, `${fileHash}${ext}`);
+    if (fse.existsSync(filePath)) {
+      res.end(
+        JSON.stringify({
+          shouldUpload: false
+        })
+      );
+    } else {
+      res.end(
+        JSON.stringify({
+          shouldUpload: true
+        })
+      );
+    }
  }
});

server.listen(3000, () => console.log("listening port 3000"));
```

## 暂停上传

讲完了生成 hash 和文件秒传，回到断点续传

断点续传顾名思义即断点 + 续传，所以我们第一步先实现“断点”，也就是暂停上传

原理是使用 XMLHttpRequest 的 abort 方法，可以取消一个 xhr 请求的发送，为此我们需要将上传每个切片的 xhr 对象保存起来，我们再改造一下 request 方法

```diff
   request({
      url,
      method = "post",
      data,
      headers = {},
      onProgress = e => e,
+     requestList
    }) {
      return new Promise(resolve => {
        const xhr = new XMLHttpRequest();
        xhr.upload.onprogress = onProgress;
        xhr.open(method, url);
        Object.keys(headers).forEach(key =>
          xhr.setRequestHeader(key, headers[key])
        );
        xhr.send(data);
        xhr.onload = e => {
+          // 将请求成功的 xhr 从列表中删除
+          if (requestList) {
+            const xhrIndex = requestList.findIndex(item => item === xhr);
+            requestList.splice(xhrIndex, 1);
+          }
          resolve({
            data: e.target.response
          });
        };
+        // 暴露当前 xhr 给外部
+        requestList?.push(xhr);
      });
    },
```

这样在上传切片时传入 requestList 数组作为参数，request 方法就会将所有的 xhr 保存在数组中了。

每当一个切片上传成功时，将对应的 xhr 从 requestList 中删除，所以 requestList 中只保存正在上传切片的 xhr

之后新建一个暂停按钮，当点击按钮时，调用保存在 requestList 中 xhr 的 abort 方法，即取消并清空所有正在上传的切片

```js
 handlePause() {
    this.requestList.forEach(xhr => xhr?.abort());
    this.requestList = [];
}
```

点击暂停按钮可以看到 xhr 都被取消了。

## 恢复上传

之前在介绍断点续传的时提到使用第二种服务端存储的方式实现续传

由于当文件切片上传后，服务端会建立一个文件夹存储所有上传的切片，所以每次前端上传前可以调用一个接口，服务端将已上传的切片的切片名返回，前端再跳过这些已经上传切片，这样就实现了“续传”的效果。

而这个接口可以和之前秒传的验证接口合并，前端每次上传前发送一个验证的请求，返回两种结果

- 服务端已存在该文件，不需要再次上传
- 服务端不存在该文件或者已上传部分文件切片，通知前端进行上传，并把已上传的文件切片返回给前端

所以我们改造一下之前文件秒传的服务端验证接口

```diff
const extractExt = filename =>
  filename.slice(filename.lastIndexOf("."), filename.length);
const UPLOAD_DIR = path.resolve(__dirname, "..", "target");

const resolvePost = req =>
  new Promise(resolve => {
    let chunk = "";
    req.on("data", data => {
      chunk += data;
    });
    req.on("end", () => {
      resolve(JSON.parse(chunk));
    });
  });
  
+ // 返回已上传的所有切片名
+ const createUploadedList = async fileHash =>
+   fse.existsSync(path.resolve(UPLOAD_DIR, fileHash))
+    ? await fse.readdir(path.resolve(UPLOAD_DIR, fileHash))
+    : [];

server.on("request", async (req, res) => {
  if (req.url === "/verify") {
    const data = await resolvePost(req);
    const { fileHash, filename } = data;
    const ext = extractExt(filename);
    const filePath = path.resolve(UPLOAD_DIR, `${fileHash}${ext}`);
    if (fse.existsSync(filePath)) {
      res.end(
        JSON.stringify({
          shouldUpload: false
        })
      );
    } else {
      res.end(
        JSON.stringify({
          shouldUpload: true，
+         uploadedList: await createUploadedList(fileHash)
        })
      );
    }
  }
});

server.listen(3000, () => console.log("listening port 3000"));
```

接着回到前端，前端有两个地方需要调用验证的接口

- 点击上传时，检查是否需要上传和已上传的切片
- 点击暂停后的恢复上传，返回已上传的切片

新增恢复按钮并改造原来上传切片的逻辑

```diff
<template>
  <div id="app">
      <input
        type="file"
        @change="handleFileChange"
      />
       <el-button @click="handleUpload">upload</el-button>
       <el-button @click="handlePause" v-if="isPaused">pause</el-button>
+      <el-button @click="handleResume" v-else>resume</el-button>
      //...
    </div>
</template>

+   async handleResume() {
+      const { uploadedList } = await this.verifyUpload(
+        this.container.file.name,
+        this.container.hash
+      );
+      await this.uploadChunks(uploadedList);
    },
    async handleUpload() {
      if (!this.container.file) return;
      const fileChunkList = this.createFileChunk(this.container.file);
      this.container.hash = await this.calculateHash(fileChunkList);
+     const { shouldUpload, uploadedList } = await this.verifyUpload(
+       this.container.file.name,
+       this.container.hash
+     );
+     if (!shouldUpload) {
+       this.$message.success("skip upload：file upload success");
+       return;
+     }
      this.data = fileChunkList.map(({ file }, index) => ({
        fileHash: this.container.hash,
        index,
        hash: this.container.hash + "-" + index,
        chunk: file，
        percentage: 0
      }));
+      await this.uploadChunks(uploadedList);
    },
    // 上传切片，同时过滤已上传的切片
+   async uploadChunks(uploadedList = []) {
      const requestList = this.data
+       .filter(({ hash }) => !uploadedList.includes(hash))
        .map(({ chunk, hash, index }) => {
          const formData = new FormData();
          formData.append("chunk", chunk);
          formData.append("hash", hash);
          formData.append("filename", this.container.file.name);
          formData.append("fileHash", this.container.hash);
          return { formData, index };
        })
        .map(({ formData, index }) =>
          this.request({
            url: "http://localhost:3000",
            data: formData,
            onProgress: this.createProgressHandler(this.data[index]),
            requestList: this.requestList
          })
        );
      await Promise.all(requestList);
+     // 之前上传的切片数量 + 本次上传的切片数量 = 所有切片数量时合并切片
+     if (uploadedList.length + requestList.length === this.data.length) {
         await this.mergeRequest();
+     }
    }
```

这里给原来上传切片的函数新增 uploadedList 参数，即上图中服务端返回的切片名列表，通过 filter 过滤掉已上传的切片，并且由于新增了已上传的部分，所以之前合并接口的触发条件做了一些改动。

到这里断点续传的功能基本完成了。

## 进度条改进

虽然实现了断点续传，但还需要修改一下进度条的显示规则，否则在暂停上传/接收到已上传切片时的进度条会出现偏差

### 单个切片进度条

由于在点击上传/恢复上传时，会调用验证接口返回已上传的切片，所以需要将已上传切片的进度变成 100%

```diff
   async handleUpload() {
      if (!this.container.file) return;
      const fileChunkList = this.createFileChunk(this.container.file);
      this.container.hash = await this.calculateHash(fileChunkList);
      const { shouldUpload, uploadedList } = await this.verifyUpload(
        this.container.file.name,
        this.container.hash
      );
      if (!shouldUpload) {
        this.$message.success("skip upload：file upload success");
        return;
      }
      this.data = fileChunkList.map(({ file }, index) => ({
        fileHash: this.container.hash,
        index,
        hash: this.container.hash + "-" + index,
        chunk: file,
+       percentage: uploadedList.includes(index) ? 100 : 0
      }));
      await this.uploadChunks(uploadedList);
    },
```

uploadedList 会返回已上传的切片，在遍历所有切片时判断当前切片是否在已上传列表里即可

### 总进度条

之前说到总进度条是一个计算属性，根据所有切片的上传进度计算而来，这就遇到了一个问题，点击暂停会取消并清空切片的 xhr 请求，此时如果已经上传了一部分，就会发现文件进度条有倒退的现象。

当点击恢复时，由于重新创建了 xhr 导致切片进度清零，所以总进度条就会倒退

解决方案是创建一个“假”的进度条，这个假进度条基于文件进度条，但只会停止和增加，然后给用户展示这个假的进度条

这里我们使用 Vue 的监听属性

```diff
  data: () => ({
+    fakeUploadPercentage: 0
  }),
  computed: {
    uploadPercentage() {
      if (!this.container.file || !this.data.length) return 0;
      const loaded = this.data
        .map(item => item.size * item.percentage)
        .reduce((acc, cur) => acc + cur);
      return parseInt((loaded / this.container.file.size).toFixed(2));
    }
  },  
  watch: {
+    uploadPercentage(now) {
+      if (now > this.fakeUploadPercentage) {
+        this.fakeUploadPercentage = now;
+      }
    }
  },
```

当 uploadPercentage 即真的文件进度条增加时，fakeUploadPercentage 也增加，一旦文件进度条后退，假的进度条只需停止即可。

至此一个大文件上传 + 断点续传的解决方案就完成了。


## 总结

断点续传

- 使用 spark-md5 根据文件内容算出文件 hash
- 通过 hash 可以判断服务端是否已经上传该文件，从而直接提示用户上传成功（秒传）
- 通过 XMLHttpRequest 的 abort 方法暂停切片的上传
- 上传前服务端返回已经上传的切片名，前端跳过这些切片的上传


参考 https://juejin.cn/post/6844904046436843527#heading-16