---
tag: plugins
---

# 3D 地图下钻功能实现

## 一、前言

在做可视化大屏项目中，地图下钻功能是常被用户提出来要做的功能，其主要功能是指在地图上点击某个区域后，该区域会进一步细化展示其下一级别的地图信息，

例如：从国家地图，点击某省份下钻到省级地图，再点击市区下钻到市级地图，再到区县级地图，主要目的通过点击进入不同的省市区进而联动不同省市区界别，展示相关的数据在大屏上，这种逐级细化的过程有助于更细致地分析和理解地区数据。

地图下钻是和用户进行交互的功能，可以使用户能够深入地图的不同层级，查看更加详细的地图层级数据，帮助用户逐层深入地查看不同区域或层级的数据分布情况，增强了数据可视化的深度和用户体验。

## 二. 实现 3D 地图
本文所实现的 3D 地图主要使用 ECharts 和 ECharts-GL 构建，ECharts-GL 相当于是 ECharts 的扩展插件，主要用于创建 3D 可视化应用程序，因此使用 ECharts 搭配 ECharts—GL 插件库就可以构建出 3D 地图。
在这里，我们也简单总结一下如何使用 ECharts + ECharts-GL 实现一款还不错的 3D 地图：

1. 引入 ECharts 和 ECharts-GL 开源库
2. 准备 geoJson 层级数据
3. 初始化 3D 地图，根据所需的 geoJson 数据渲染地图
4. 丰富地图 3D 效果：包括地图颜色、标签、鼠标指向、提示等基础配置，环境贴图、地面、光照、纹理、视角控制等丰富配置

### 1. 引入 ECharts 和 ECharts-GL

下载合适版本的 ECharts 和 ECharts-GL，在项目中进行引入:

```shell
npm install echarts
npm install echarts-gl
```

### 2. 准备 geoJson 数据

在测试阶段，我们可以使用一些第三方网站已有的最新 geoJson 数据进行调试，比如：通过阿里云 DataV 数据可视化地图小工具，可以请求最新的 json 数据文件进行调试：

[阿里云 DataV.GeoAtlas 地理小工具](https://datav.aliyun.com/portal/school/atlas/area_selector)

它提供了一些 JSON API 可以通过远程调用，比如，请求中国地图的 geoJson 数据

```js
axios
  .get("https://geo.datav.aliyun.com/areas_v3/bound/geojson?code=100000_full")
  .then((res) => {
    // 请求完成的中国地图 geoJson 数据
  });
```

### 3. 开发基础 3D 地图

要实现一款效果还算不错的 3D 地图，只需要以下配置即可：

```js
const geoJson = "获取的的中国地图json";
const myChart = echarts.init(document.getElementById("main"));
// 注册地图名字(china)和数据(geoJson)
echarts.registerMap("china", geoJson);
// 图表配置项
const option = {
  tooltip: {}, // 提示框组件，指向地图该如何提示
  series: [
    {
      type: "map3D",
      map: "china",
      itemStyle: {}, // 地图的区域颜色
      label: {}, // 标签的相关设置
      emphasis: {}, // 鼠标 hover 高亮时图形和标签的样式
      environment: "auto", // 环境贴图
      groundPlane: {}, // 设置地面，地面可以使整个场景看起来更真实，更有模型感。
      light: {}, // 光照相关的设置
      viewControl: {}, // 设置视角控制
      shading: "realistic", // 三维图形的着色效果
      realisticMaterial: {}, // 真实感材质相关的配置项
      postEffect: {}, // 后处理特效
      data: [], // 数据，重要
    },
  ],
};
// 设置图表实例的配置项以及数据
myChart.setOption(option);
```

## 三. 实现 3D 地图下钻

有了以上的效果，接下来我们从零开始实现 3D 地图的下钻功能

1. 准备层级数据（包括中国、各省市区的 geoJson 数据）
2. 设置地图点击事件，当点击地图区域时，获取到该区域对应的 geoJson 数据
3. 根据区域 geoJson 数据渲染区域地图

### 1. 设置地图点击事件

首先需要给地图添加点击事件监听器，当用户点击地图时，获取到用户点击该区域对应的编码。

```js
myChart.on("click", function (params) {
  if (params.data) {
    const { adcode, name, level } = params.data;
  }
});
```

在这里有一个前提，我们在点击地图某区域时，能获取到该区域 adcode 的前提是：在渲染地图时，提前赋值了数据（在 series.data 中赋值），例如如下代码：

```js
const option = {
  series: [
    {
      data: geoJson.features.map((item) => {
        return item.properties;
      }),
    },
  ],
};
```

有了以上代码的设置，即可正常获取到 adcode, name, level 等数据。

当我们点击地图区域时，以点击山东省为例，会输出如下图所示的数据，则证明我们的数据正常。

### 2. 获取区域地图 JSON

根据上面的点击，获取到点击区域的地理编码 adcode 和名称，接下来需要通过 adcode 来获取子区域的 geoJson，以下的代码是通过调用阿里云 DataV 在线的 JSON API 接口获取到的数据。

```js
// 根据 adcode 获取原始地图json数据
function getJSON(adcode) {
  return axios.get(
    `https://geo.datav.aliyun.com/areas_v3/bound/geojson?code=${adcode}_full`
  );
}
// 例如：根据 adcode 获取省市区的 json 数据
// 获取山东省下的所有市的数据
getJSON("370000").then((res) => {
  console.log("山东省子区域地图数据：", res);
});

// 获取青岛市下的所有区级数据
getJSON("370200").then((res) => {
  console.log("山东省子区域地图数据：", res);
});
```

### 3. 渲染区域地图

通过上述操作，我们成功的获取到了点击区域的子区域地图 JSON 数据，通过 JSON 数据，我们就可以动态的渲染区域地图了，如下代码所示：

```js
// 渲染地图
function renderChart(name, data) {
  // 注册地图
  echarts.registerMap(name, data);
  // 根据 json 数据拼装 mapdata， 用于地图点击下钻时传递数据，主要有adcode、name
  const mapdata = data.features.map((item) => {
    return item.properties;
  });
  // 配置option
  setOption(name, mapdata);
}

// 根据adcode区域编码获取地图数据，例如：根据山东省的区域编码：370000，获取市区geoJson数据
getJSON("370000", function (data) {
  console.log("山东省子区域地图数据：", data);
  renderChart("山东省", data);
});
```

## 四. 体验优化

所谓体验优化，就是让用户使用舒服，在这里我仅提供一些简单优化思路，包括但不限于以下几种：

- 判断是否为最底层地图，也就是末级地图，当我们点击地图进行下钻到无法再继续时，可以提示用户或返回首层地图，是具体业务而定
- 可以记录点击地图的路径历史，类似于浏览器的 history，可以回退，避免用户点击下钻错误时返回上一层地图

### 1. 判断末级地图

在事件处理函数中，根据用户点击的区域信息，判断是否需要进行下钻操作。如果需要下钻，才可以展示子区域的地图信息。如果已经到了地图最末端，无法再进行下钻，应该相应提示用户。

例如，当用户点击到区县级地图且无法再下钻时，可以考虑弹窗显示详细信息或者返回至上一级别。

```js
// 设置地图点击事件
myChart.on("click", function (params) {
  if (params.data) {
    const { adcode, name, level } = params.data;
    // 判断如果是 district 层级，则提示用户已经为最底级了
    if (level === "district") {
      alert("无此区域地图显示！");
      initChinaMap(); // 重新渲染中国地图或其他逻辑处理
      return;
    }
    // 继续进行下钻
    getJSON({ name, adcode }, function (data) {
      renderChart(name, data);
    });
  }
});
```

### 2. 返回上一级地图

在用户点击地图进行下钻的过程中，可以返回上一层进行重新点击地图进行下探！

实现这个功能主要需要定义一个变量 mapList 数组，记录地图的层级数据，主要用于返回时可以根据 adcode 获取上一级的地图数据。

当用户点击地图进行下钻时，记录地图的名称、级别和编码。当用户返回上一级的时候，需要删除记录数组中对应的数据。

核心代码如下：

```js
// 点击地图下钻时记录name和adcode
mapList.push({ name, adcode });

// 点击返回时删除对应的name和adcode
mapList.splice(-2, 2);

// 点击返回，地图返回上一级
function goBack() {
  if (mapList.length >= 2) {
    const { name, adcode } = mapList[mapList.length - 2];
    mapList.splice(-2, 2);
    getJSON({ name, adcode }, function (data) {
      renderChart(name, data);
    });
  }
}
```


本文来自https://juejin.cn/post/7398352956712534054
