# vite 迁移至 Rspack 实践

## vite 的问题
大型项目开发期间页面刷新缓慢：

随着项目文件增多，Vite 页面加载速度会越来越慢，这个问题在开发时尤为明显，因为开发期间需要打开浏览器的 DevTools，使用 DevTools 时我们的很多页面需要加载 10 秒以上，这对于开发体验来说是很难接受的。

> 注意这里说的是开发期间 (vite dev) 的页面加载速度，而不是项目打包 (vite build) 后的页面加载速度

开发期间的页面加载速度是 Vite 的开发体验的瓶颈

注意这个瓶颈不是 HMR (页面热重载)，也不是 Vite 编译速度过慢，这 2 个速度 Vite 已经足够快了。

页面重载速度的瓶颈是来自于：每个页面的请求数太多，瓶颈其实发生在浏览器和 DevTools 上。而背后的原因是 Vite 直接把 ES 模块直接发送到浏览器的机制，这意味着即使在未来 Vite 实装了 Rolldown 这个问题也不会得到解决。

这个问题一直得不到重视，他们好像更关心的是启动速度、HRM 速度，但对于我们开发中的较大的项目来说，页面刷新速度（重载速度）是一个非常重要的问题，因为日常开发中常常需要刷新页面来进行业务调试，每一次代码更改可能对应着几十次页面刷新，而 Vite 的最大的瓶颈就在与此。

## Rspack 的优势

1. 没有瓶颈的页面加载速度
2. 开发环境与生产环境一致
3. 令人惊喜的打包速度
4. webpack 生态兼容性

## 迁移过程

### 1. 安装核心依赖

移除vite：

```shell
npm remove vite
```

安装 Rsbuild：

```shell
npm add @rsbuild/core -D
```

### 2. 更新 npm scripts

```json
{
  "scripts": {
    "dev": "rsbuild dev",
    "build": "rsbuild build"
  }
}
```

### 3. 创建配置文件

```shell
touch rsbuild.config.js
```

```js
// rsbuild.config.js
import { defineConfig } from '@rsbuild/core'

export default defineConfig({
  plugins: [],
})
```

在这里，配置文件类型可以是 rsbuild.config.js 或者 rsbuild.config.ts。我暂且选了 .js。

### 4. 构建入口

移除 index.html 中的 `<script type="module" src="/src/main.ts"></script>` 标签。

然后在 rsbuild.config.js 添加如下配置即可：

```js
export default {
  html: {
    template: './index.html',
  },
  source: {
    entry: {
      index: './src/main.js',
    },
  },
};
```

Rsbuild 会在构建时自动注入 `<script>` 标签到生成的 HTML 文件中。

假如有使用路径别名，可以在 rsbuild.config.js 中添加如下配置。

```js
export default {
  source: {
    alias: {
      '@': './src',
    },
  },
};
```

### 5. 安装框架插件

由于我的项目是 Vue 3 和 Sass，所以需要安装 `@rsbuild/plugin-vue` 和 `@rsbuild/plugin-sass`。虽然我也有少量写 JSX，但在安装了 `@rsbuild/plugin-vue-jsx` 之后，发现编译报错，所以暂时不安装，并将 JSX 写法改成了 Vue 的写法。

```shell
npm add @rsbuild/plugin-vue @rsbuild/plugin-sass -D
```

```js
import { defineConfig } from '@rsbuild/core'

import { pluginVue } from '@rsbuild/plugin-vue'
import { pluginSass } from '@rsbuild/plugin-sass'

export default defineConfig({
  plugins: [pluginVue(), pluginSass()],
})
```

### 6. 配置 UI 框架

我这里有用到 Vant 4 和 TDesign mobile，所以需要安装以下插件。

```shell
npm add unplugin-auto-import unplugin-vue-components @vant/auto-import-resolver -D
```

然后在 rsbuild.config.js 中添加如下配置。

```js
import { defineConfig } from '@rsbuild/core'

import { pluginVue } from '@rsbuild/plugin-vue'
import { pluginSass } from '@rsbuild/plugin-sass'
import AutoImport from 'unplugin-auto-import/rspack'
import Components from 'unplugin-vue-components/rspack'
import { VantResolver } from '@vant/auto-import-resolver'
import { TDesignResolver } from 'unplugin-vue-components/resolvers'
export default defineConfig({
  html: {
    template: './index.html',
  },
  source: {
    entry: {
      index: './src/main.js',
    },
    alias: {
      '@': './src',
    },
  },
  plugins: [pluginVue(), pluginSass()],
  tools: {
    rspack: {
      plugins: [
        AutoImport({
          resolvers: [VantResolver(), TDesignResolver({ library: 'mobile-vue' })],
        }),
        Components({
          resolvers: [VantResolver(), TDesignResolver({ library: 'mobile-vue' })],
        }),
      ],
    },
  },
})
```

### 7. 修改环境变量 Key

跟 Vite 类似，也可以通过 import.meta.env 来访问环境变量。但假如是 client 端能访问的变量，那么就需要将环境变量的 Key 从 VITE_APP 改成 PUBLIC。这样才能在 client 端访问到。不然就是 undefined。假如你不加 PUBLIC，那么在 client 端访问 import.meta.env 时，也只会是 undefined。

```js
console.log(import.meta.env.PUBLIC_NAME); // -> 'jack'
console.log(import.meta.env.PASSWORD); // -> undefined

console.log(process.env.PUBLIC_NAME); // -> 'jack'
console.log(process.env.PASSWORD); // -> undefined
```

### 8. 其他

我发现 pluginSass 对于 Sass 写法有点严苛检测，比如项目中存在一些同事的走心写法，在 Vite 中会正常编译无警告，但在 Rsbuild 中就会报错，需要将其修正。

至此，迁移完成。






