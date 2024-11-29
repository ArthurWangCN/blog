---
title: 百度地图 JS API
date: 2024-04-17 21:57:29
tags:
categories: 前端
description: 百度地图JavaScript API是一套由JavaScript语言编写的应用程序接口，可帮助您在网站中构建功能丰富、交互性强的地图应用，支持PC端和移动端基于浏览器的地图应用开发，且支持HTML5特性的地图开发。
---

百度地图JavaScript API是一套由JavaScript语言编写的应用程序接口，可帮助您在网站中构建功能丰富、交互性强的地图应用，支持PC端和移动端基于浏览器的地图应用开发，且支持HTML5特性的地图开发。

百度地图JavaScript API支持HTTP和HTTPS，免费对外开放，可直接使用。接口使用无次数限制。在使用前，您需先申请密钥（ak）才可使用。

## 准备工作

第一步、注册开发者账号：

https://lbsyun.baidu.com/apiconsole/key?application=key#/home


第二步、创建应用，获取AK

进入控制台--应用管理--我的应用下面，创建应用。

注册成功之后，会生成一个App Key, 这个AK贯穿你使用百度地图Api的始终。

另外, 百度地图对个人认证用户, 绝大多数Api每天调用的限制次数是5000次, 每秒钟只能查询30次。用于学习的话，够用了。


## 快速上手

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8">
    <title>地图展示</title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge">
    <style>
    body,
    html,
    #container {
        overflow: hidden;
        width: 100%;
        height: 100%;
        margin: 0;
        font-family: "微软雅黑";
    }
    .info {
        z-index: 999;
        width: auto;
        min-width: 22rem;
        padding: .75rem 1.25rem;
        margin-left: 1.25rem;
        position: fixed;
        top: 1rem;
        background-color: #fff;
        border-radius: .25rem;
        font-size: 14px;
        color: #666;
        box-shadow: 0 2px 6px 0 rgba(27, 142, 236, 0.5);
    }
    </style>
    <script src="//api.map.baidu.com/api?type=webgl&v=1.0&ak=您的密钥"></script>
</head>
<body>
    <div class = "info">最新版GL地图命名空间为BMapGL, 可按住鼠标右键控制地图旋转、修改倾斜角度。</div>
    <div id="container"></div>
</body>
</html>
<script>
    var map = new BMapGL.Map('container'); // 创建Map实例
    map.centerAndZoom(new BMapGL.Point(116.404, 39.915), 12); // 初始化地图,设置中心点坐标和地图级别
    map.enableScrollWheelZoom(true); // 开启鼠标滚轮缩放
</script>
```

## 在ts中使用百度地图

方式一、创建一个全局的ts声明文件global.d.ts,在里面加入BMapGL全局类型声明

```ts
declare const BMapGL: any;
```

方式二

```shell
yarn add -D @types/bmapgl
```

在tsconfig.lib.json中引入bmapgl的定义

```ts
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "../../dist/out-tsc",
    "types": ["webpack-env", "node", "bmapgl"]
  },
  "exclude": ["**/*.spec.ts", "**/*.spec.tsx"],
  "include": ["../../types", "**/*.ts", "**/*.tsx", "**/*.vue"]
}
```

## 调用百度地图功能
### 根据经纬度获取地址

```ts
// 根据经纬度获取地址
export const getGeoSurroundingPois = (point) => {
  return new Promise((resolve, reject) => {
    try {
      const geoc = new win.BMapGL.Geocoder()
      geoc.getLocation(point, function (geocInfo) {
        if (geocInfo.surroundingPois) resolve(geocInfo.surroundingPois)
      })
    } catch (error) {
      reject(error)
    }
  })
}

baidumap.addEventListener('click', async function (e) {
    console.log('点击位置经纬度：' + e.latlng.lng + ',' + e.latlng.lat)
    baidumap.clearOverlays()
    addMarker(
      {
        BMap: baidumap,
        pointMarker: '',
        lng: e.latlng.lng,
        lat: e.latlng.lat,
        icon: ''
      },
      false
    )
    const res: any = await getGeoSurroundingPois(e.latlng)
    locations.value = res
    if (res.length > 0 && res[0].title) address.value = res[0].title
  })
```
