# vite构建打包性能优化

## 一、清除console和debugger

安装 terser插件

```shell
npm install  terser  -D
```

build里添加terserOptions配置

```ts
// 打包环境移除console.log，debugger
terserOptions: {
  compress: {
    drop_console: true,
    drop_debugger: true
  }
},
```

## 二、gzip静态资源压缩

**第一步：客户端打包开启**

首先下载 vite-plugin-compression

```shell
npm i vite-plugin-compression -D
```

在vite.config.ts中引入

```ts
// 引入
import viteCompression from 'vite-plugin-compression'

//在plugins配置数组里添加gzip插件
viteCompression({
  verbose: true, // 默认即可
  disable: false, // 开启压缩(不禁用)，默认即可
  deleteOriginFile: false, // 删除源文件
  threshold: 5120, // 压缩前最小文件大小
  algorithm: 'gzip', // 压缩算法
  ext: '.gz' // 文件类型
})
```

**第二步：部署服务端开启**

nginx中添加压缩配置

```conf
# 开启或者关闭gzip模块(on|off)
gzip on;
# 允许压缩的页面最小字节数, 默认值是0，不管页面多大都压缩。建议设置成大于1k的字节数，小于1k可能会越压越大
gzip_min_length 1k; 
# 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。
gzip_buffers 4 16k; 
# 识别http的协议版本。由于早期的一些浏览器或者http客户端，可能不支持gzip自解压，用户就会看到乱码，所以做一些判断还是有必要的。
#gzip_http_version 1.0;
# gzip压缩比，1 压缩比最小处理速度最快，9 压缩比最大但处理最慢（传输快但比较消耗cpu）。
gzip_comp_level 2;
# 匹配MIME类型进行压缩，（无论是否指定）"text/html"类型总是会被压缩的。
gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
# 和http头有关系，加个vary头，给代理服务器用的
gzip_vary off;
# 表示IE6及以下禁止压缩
gzip_disable "MSIE [1-6]\.";
```


## 三、 静态文件按类型分包

build中添加如下代码：

```ts
build: {
    rollupOptions: {
      output: {
        chunkFileNames: 'static/js/[name]-[hash].js',
        entryFileNames: 'static/js/[name]-[hash].js',
        assetFileNames: 'static/[ext]/[name]-[hash].[ext]',
      }
    }
  }
```

## 四、超大静态资源拆分(代码分割)

第一种：提高静态资源的容量大小

```ts
build: {
 chunkSizeWarningLimit: 1500,
}
```

第二种：合并路由打包

build里的output设置内,添加以下代码

```ts
manualChunks: {
  // vue vue-router合并打包
  vue: ['vue', 'vue-router'],
  lodash: ['lodash'],
  // 两个文件合并成一个a文件
  helloWorld: ['src/components/a.vue','src/components/b.vue'],
}
```

第三种：最小拆分打包

```ts
// 插件打包做处理
manualChunks(id) {
  if (id.includes('node_modules')) {
    return id.toString().split('node_modules/')[1].split('/')[0].toString();
  }
  return;
}
```


## 五、打包分析插件

Rollup Plugin Visualizer,这是一个依赖分析插件，它提供了多种模式的依赖分析，包括直观的视图分析，sunburst（循环层次图，像光谱）、treemap（矩形层次图，看起来比较直观，也是默认参数）、network（网格图，查看包含关系）、raw-data（原数据模式，json格式）, list(列表模式),你可以选择任意一种你喜欢的观察模式。

安装rollup-plugin-visualizer插件

```shell
npm i rollup-plugin-visualizer
```

在vite.config.ts中引入并配置

```ts
// 引入
import { visualizer } from 'rollup-plugin-visualizer';

// 在plugins配置数组里添加
visualizer({
  open:true, // 注意这里要设置为true，否则无效，如果存在本地服务端口，将在打包后自动展示
  gzipSize:true,
  file: "stats.html", //分析图生成的文件名
  brotliSize:true
}),
```

将在打包后自动展示


## 六、组件按需导入

安装unplugin-vue-components插件

```shell
npm i unplugin-vue-components -D
```

在vite.config.ts中引入并配置

```ts
import Components from 'unplugin-vue-components/vite'
// ui库解析器，也可以自定义，需要安装相关UI库，unplugin-vue-components/resolvers
// 提供了以下集中解析器，使用的时候，需要安装对应的UI库，这里以element为示例
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    Components({
      dirs: ['src/components'], // 目标文件夹
      extensions: ['vue','jsx'], // 文件类型
      dts: 'src/components.d.ts', // 输出文件，里面都是一些import的组件键值对
      // ui库解析器，也可以自定义，需要安装相关UI库
      resolvers: [
        ElementPlusResolver(),
      ],
    })
  ]
})
```

## 七、 图片资源压缩

安装 vite-plugin-imagemin插件

```shell
npm i vite-plugin-imagemin -D
```

 在vite.config.js里面配置

```ts
// 引入
import viteImagemin from 'vite-plugin-imagemin'

// 在plugins配置数组里添加
plugin: [
    viteImagemin({
      gifsicle: {
        optimizationLevel: 7,
        interlaced: false
      },
      optipng: {
        optimizationLevel: 7
      },
      mozjpeg: {
        quality: 20
      },
      pngquant: {
        quality: [0.8, 0.9],
        speed: 4
      },
      svgo: {
        plugins: [
          {
            name: 'removeViewBox'
          },
          {
            name: 'removeEmptyAttrs',
            active: false
          }
        ]
      }
    })
]
```


## 八、CDN加速 

内容分发网络（Content Delivery Network，简称 CDN）就是让用户从最近的服务器请求资源，提升网络请求的响应速度。通常我们请求依赖模块使用 CDN ，而请求项目代码依然使用自己的服务器。还是以 lodash 为例：

安装插件 vite-plugin-cdn-import

```shell
npm install vite-plugin-cdn-import --save-dev
```

在vite.config.js里面配置

```ts
import { autoComplete, Plugin as importToCDN } from 'vite-plugin-cdn-import';

export default defineConfig({
  plugins: [
     importToCDN({
      // 需要 CDN 加速的模块
      modules: [
        // 支持自动导入
        autoComplete('axios'),
        {
          name: 'lodash',
          var: '_',
          path: `https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js`
        }
      ]
    })
  ]
})
```

配置完成后，打包自动会在index.html中引入script资源

自动导入CDN的功能（autoComplete），有很多常用的依赖不需要自己再找（如vue、reac、antd、axios等），autoComplete功能可以在文档中查找使用。

注意：自动完成记得在main.ts中引入模块

在main.ts引入对应模块然后挂载，注意名称要和你CDN加速的name名称相同，不然引不进来，一般不用怎么修改，只有遇到问题后再检查一下自己是不是忘记引入或者引入错误了。

CDN网站查看： https://cdnjs.com/

注意：build打包时候如果遇到报错importToCDN is not a function，这个错误应该和版本有关，将import importToCDN from ‘vite-plugin-cdn-import’引入改成import { Plugin as importToCDN } from ‘vite-plugin-cdn-import’即可


完整的vite.config.ts 

```ts
import path from 'path';
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import vueJsx from '@vitejs/plugin-vue-jsx';
import Components from 'unplugin-vue-components/vite';
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers';
import { viteCommonjs } from '@originjs/vite-plugin-commonjs';
import AutoImport from 'unplugin-auto-import/vite';
import WindiCSS from 'vite-plugin-windicss';
import viteCompression from 'vite-plugin-compression';
import { visualizer } from 'rollup-plugin-visualizer';
import { autoComplete, Plugin as importToCDN } from 'vite-plugin-cdn-import';
// 引入
import viteImagemin from 'vite-plugin-imagemin';
import Components from 'unplugin-vue-components/vite';
// ui库解析器，也可以自定义，需要安装相关UI库，unplugin-vue-components/resolvers
// 提供了以下集中解析器，使用的时候，需要安装对应的UI库，这里以element为示例
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers';

const pathSrc = path.resolve(__dirname, 'src');
export default defineConfig({
  resolve: {
    alias: {
      '@/': `${pathSrc}/`
    }
  },
  plugins: [
    vueJsx(),
    WindiCSS(),
    vue(),
    visualizer(),
    viteCommonjs(),
    Components({
      dirs: ['src/components'], // 目标文件夹
      extensions: ['vue', 'jsx'], // 文件类型
      dts: 'src/components.d.ts', // 输出文件，里面都是一些import的组件键值对
      // ui库解析器，也可以自定义，需要安装相关UI库
      resolvers: [
        ElementPlusResolver()
      ]
    }),
    // 自动导入vue3的hooks
    AutoImport({
      resolvers: [ElementPlusResolver()],
      imports: ['vue', 'vue-router', 'vuex', '@vueuse/core'],
      // 生成路径
      dts: path.resolve(pathSrc, 'auto-imports.d.ts')
    }),
    // 自动导入组件
    Components({
      resolvers: [
        ElementPlusResolver({
          importStyle: 'sass'
        })
      ],
      dts: path.resolve(pathSrc, 'components.d.ts')
    }),
    viteCompression({
      // gzip静态资源压缩配置
      verbose: true,
      disable: false,
      threshold: 10240,
      algorithm: 'gzip',
      ext: '.gz'
    }),
    importToCDN({
      // 需要 CDN 加速的模块
      modules: [
        autoComplete('axios'),
        {
          name: 'lodash',
          var: '_',
          path: `https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js`
        }
      ]
    }),
    viteImagemin({
      gifsicle: {
        optimizationLevel: 7,
        interlaced: false
      },
      optipng: {
        optimizationLevel: 7
      },
      mozjpeg: {
        quality: 20
      },
      pngquant: {
        quality: [0.8, 0.9],
        speed: 4
      },
      svgo: {
        plugins: [
          {
            name: 'removeViewBox'
          },
          {
            name: 'removeEmptyAttrs',
            active: false
          }
        ]
      }
    })
  ],
  server: {
    watch: { usePolling: true },
    open: false,
    hmr: true,
    /* 设置为0.0.0.0则所有的地址均能访问 */
    host: '0.0.0.0',
    port: 8080,
    https: false,
    proxy: {
      '/api': {
        /* 目标代理服务器地址 */
        target: 'https://code-nav.top/',
        /* 允许跨域 */
        changeOrigin: true
      }
    }
  },
  build: {
    outDir: 'JcCloudWeb',
    minify: 'terser',
    cssCodeSplit: true,
    chunkSizeWarningLimit: 1500,
    rollupOptions: {
      input: 'index.html',
      output: {
        // 静态资源打包做处理
        chunkFileNames: 'assets/js/[name]-[hash].js',
        entryFileNames: 'assets/js/[name]-[hash].js',
        assetFileNames: 'assets/[ext]/[name]-[hash].[ext]',
        manualChunks(id) {
          if (id.includes('node_modules')) {
            return id.toString().split('node_modules/')[1].split('/')[0].toString();
          }
        }
      }
    },
    terserOptions: {
      // 清除console和debugger
      compress: {
        drop_console: true,
        drop_debugger: true
      }
    }
  }
});
```
    

