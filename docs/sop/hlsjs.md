---
tag: plugins
---

# 使用 hls.js 播放 m3u8 格式视频

关于课程播放、活动直播之类的流媒体资源都是m3u8的格式。使用的阿里云直播和点播服务，但是video标签是无法直接播放m3u8格式的，所以采用阿里云提供的播放器播放。说实话，那时候只知道是m3u8格式，对该格式其实并不了解。其实我们学习还是需要深入源头，对不清楚的东西多加以了解。

## HLS 文件视频介绍

HLS （HTTP Live Streaming）是由苹果公司率先提出的一种协议标准，可用于直播。m3u8是一种基于 HLS 文件视频格式，它主要是存放整个视频的基本信息和分片(Segment)组成。不同于 mp4 大文件，m3u8 是由一系列的 ts 文件组成，一般一个 ts 文件大概 5-10 秒，这些 ts 文件通过一个 .m3u8 文件做索引。用户播放视频时，可随意拖动视频进度，会读取相应进度的 ts 文件继续观看视频，不必等到下载完整的视频。因此在播放 m3u8 的时候很少有卡顿的现象。现在越来越多的视频应用使用了 m3u8 格式的视频，包括点播流/直播流。相比 mp4 等视频源，m3u8 可以减轻服务器压力（按需加载）。

由于 HLS 是由 Apple 公司提出的，所以在 iOS 电脑或手机上，你可以直接使用 Safari 浏览器的 <video> 播放 m3u8 格式视频文件。而其他浏览器则需要借助 hls.js 来兼容m3u8。

使用 hls.js，不需要任何定制的播放器，只需要 `<video>` 元素就能播放 m3u8

## hls.js 如何使用

加载js和播放元素：在需要放置视频的页面位置上加入video元素和hls.js文件。

```html
// 提供播放元素
<video id="video" controls width="100%"></video>
// 加载 hls.js 文件
<script src="hls.js"></script>
```

调用 hls.js：演示页面查看源代码就可以了解

首先判断浏览器是否支持hls，如果支持就实例化 new Hls()，加载m3u8源，然后播放。如果不支持hls，而支持苹果原生应用，则播放另一个m3u8源。

```js
var video = document.getElementById('video');
  if(Hls.isSupported()) {
    var hls = new Hls();
    hls.loadSource('https://yunqivedio.alicdn.com/2017yq/v2/0x0/96d79d3f5400514a6883869399708e11/96d79d3f5400514a6883869399708e11.m3u8');
    hls.attachMedia(video);
    hls.on(Hls.Events.MANIFEST_PARSED,function() {
      video.play();
  });
 } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
    video.src = 'https://video-dev.github.io/streams/x36xhzz/x36xhzz.m3u8';
    video.addEventListener('loadedmetadata',function() {
      video.play();
    });
  }
```

运行，试着用PC浏览器和手机访问，你会发现m3u8的播放很流畅

**在Vue中使用hls.js**

```js
// 1、使用Vue框架可以用npm先安装hls
npm install --save hls.js

// 2、然后添加组件
<video
  v-if="src"
  ref="videoRef"
  :key="src"
  style="width: 100%; height: calc(100% - 52px)"
  controls
></video>

// 3、最后挂载代码：
const src = ref('xxxx')
const videoRef = ref()
const hls = ref()

if (Hls.isSupported()) {
  hls.value = new Hls()
  hls.value.loadSource(src.value)
  hls.value.attachMedia(videoRef.value)
  hls.value.on(Hls.Events.MANIFEST_PARSED, function () {
    videoRef.value?.play()
  })
} else if (videoRef.value?.canPlayType('application/vnd.apple.mpegurl')) {
  videoRef.value.src = src.value
  videoRef.value?.addEventListener('canplay', function () {
    videoRef.value?.play()
  })
}

onbeforeUnmount(() => {
  hls.value?.destroy && hls.value?.destroy()
  hls.value = null
})
```


