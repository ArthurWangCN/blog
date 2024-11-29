---
title: 前端打印方案
date: 2024-03-28 19:19:03
tags:
categories: 前端
description: 前端打印方案，调起页面的打印
---

## window.print

> Window.print 方法打开打印对话框，以打印当前文档。

print 方法最大的缺陷就是无法只打印特定的dom。

## vue3-print-nb

### 用法

安装：

```bash
npm install vue-print-nb --save
```

引入：

```js
// 全局引入
import { createApp } from 'vue'
import App from './App.vue'
import print from 'vue3-print-nb'
const app = createApp(App)
app.use(print)
app.mount('#app')

// 局部引入
import print from 'vue3-print-nb'

directives: {
    print   
}
```

使用：

```html
<div id="printTest"> 打印测试 </div> 
<el-button v-print="'#printTest'">打印</el-button>
```

### 踩坑解决

1. 打印时内容显示不全问题

   在实际页面中高度不够部分内容隐藏了，这时候需要将对应块的内容为隐藏滚动的内容显示为全显示，打印时隐藏的内容并不会打印，只会打印到页面直接展示出来的内容

   ```css
   // 打印媒体查询
   @media print {
     @page{
       size:  auto;
       margin: 3mm;
     }
     body{ 
       height:auto;  //在实际页面中高度不够部分内容隐藏了
     }    
   }
   ```



## Print.js

### 用法

安装：

```bash
npm install print-js --save
```

使用：

```html
<script>
import print from 'print-js'
  printSomething(){
    // 此处的style即为打印时的样式
    const style = '@page {  } ' +'@media print { .print-div{ padding:8px;background-color:#cccccc;line-height:12px } .red{ color:#f00} .green{color:green}' ;
    print({
        printable: 'print_area',
        type: 'html',
        style: style,// 亦可使用引入的外部css;
        scanStyles: false
    })
}
</script>

<div>
    <p>test print</p>
    <div class="print-div" id="print_area">
        <p class="red">世上本没有路，走的人多了，便有了路 ---- 鲁迅</p>
        <p class="green">世上本没有路，走的人多了，便有了路 ---- 鲁迅</p>
        <p>世上本没有路，走的人多了，便有了路 ---- 鲁迅</p>
        <p>世上本没有路，走的人多了，便有了路 ---- 鲁迅</p>
    </div>
    <button @click="printSomething">打印</button>
</div>
```

