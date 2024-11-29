---
title: 可视化大屏绘制地图相关知识点
date: 2023-09-06 14:57:58
tags: 可视化
categories: 项目
description: 接触了可视化大屏开发的小伙伴们或多或少都会地图模块的绘制与生成，不论是通过 echarts 还是别的可视化绘图工具，相关 api 都会要求我们提供一份目标地图对应的 GeoJSON 文件，可以是本地 JSON文件，可以是异步的链接。
---

接触了可视化大屏开发的小伙伴们或多或少都会地图模块的绘制与生成，不论是通过 echarts 还是别的可视化绘图工具，相关 api 都会要求我们提供一份目标地图对应的 GeoJSON 文件，可以是本地 JSON文件，可以是异步的链接。

那么没有 GIS 相关经验的前端小伙伴肯定就想知道了，GeoJSON 究竟是个啥？为啥地图模块绘制必须依赖它？



## 1、GeoJSON 是什么？

GeoJSON 是一种对各种地理数据结构进行编码的格式，基于 JavaScript 对象表示法( JavaScript Object Notation）, 简称 JSON 的地理空间信息数据交换格式。GeoJSON 对象可以表示几何、特征或者特征集合。GeoJSON 支持下面这几种几何类型：`点（Point）、线（LineString）、面（Polygon）、多点（MultiPoint）、多线（MultiLineString）、多面（MultiPolygon）和几何集合（GeometryCollection）`。GeoJSON 里的`特征（Feature）`包含一个几何对象和其他属性，`特征集合（FeatureCollection）`表示一系列特征。



## 2、GeoJSON 该如何获取？

1. 在线生成 GeoJSON 文件：[geojson.io](https://link.juejin.cn?target=https%3A%2F%2Fgeojson.io)

这里可以操作地图添加点线面来生成自己需要展示的 GeoJSON，也可以导入现成的地图块进行预览查看和编辑。

1. 在线获取目标行政区域的 GeoJSON 文件：[datav.aliyun.com/portal/scho…](https://link.juejin.cn?target=http%3A%2F%2Fdatav.aliyun.com%2Fportal%2Fschool%2Fatlas%2Farea_selector)

这里主要依靠 Ali Datav 产品提供的增值服务，可以让我们选择我们需要渲染的目标行政区域对应的 GeoJSON，可以支持全国、省份、城市、区县

> 之前有过一段时间出现过暂停服务的情况，所以个人建议将对应文件下载到自己的服务器或 CDN 进行保存，防止获取时出现异常。

例如：

杭州市 GeoJSON：[geo.datav.aliyun.com/areas_v3/bo…](https://link.juejin.cn?target=https%3A%2F%2Fgeo.datav.aliyun.com%2Fareas_v3%2Fbound%2F330100%5C_full.json)

我们打开它，可以发现行政区域描述主要是由一个**特征集合**以及多个区域的**特征**组成的，里面包含每个区域的**名称、中心点坐标、重心坐标以及构成这个区域的经纬度坐标**，这些我们在后面绘制地图时都需要用到。



## 3、常规地图板块的生成？

这里先看一个生成好的例子： [ztstory.github.io/vue-datav/#…](https://link.juejin.cn?target=https%3A%2F%2Fztstory.github.io%2Fvue-datav%2F%23%2F)

中间的镂空地图就是借助 echarts 通过浙江省的 GeoJSON 生成的浙江板块图，后面还会在此基础上增加一些流向线和特效。

具体实现是利用 options.geo 来完成的

```js
// 获取到行政区域的 geoJson 后，需要先全局注册地图名称，方便后续配置匹配
echarts.registerMap('registerMapName', geoJson);

geo: {
    map: 'registerMapName', // 这里名称需要对应
    // ...相关样式配置这里就不赘述了，参见文档即可
    // 如果对具体某些板块有自定义需求，可以用regions，下面列出常见的配置
    regions: [{
        name: '区域板块的名称',
        selected: true,
        select: {
            // 设置选中样式
            ...
        },
        tooltip: {
            // 一些区域板块绑定的提示框组件
        }
    }],
    // 这里对于一些有 i18n 需求的小伙伴可以在这里处理，自定义每个板块显示名称
    nameMap: {
        '浙江': 'Zhe Jiang'
    }
}
```

这些基本配置做好我们已经可以通过 `echarts` 生成一个基本的地图模块了。

> PS：https://www.isqqw.com/?t=map3D 中有很多示例可以参考

本文来自：https://juejin.cn/post/7226271371349082173

