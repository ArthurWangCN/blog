# 解决 vite 在 Jenkins 上打包导致内存爆满问题

## 场景
采用Jenkins打包、因为项目打包放在打包服务器上，间歇性出现打包CPU直接爆了，服务器直接die了，需要重启服务器才能解决，从而也导致项目的其他服务出现异常。

## 解决方案

### build 配置采用”esbuild“进行打包：

```ts
// vite.config.ts
build: {
    minify: "esbuild"
}
```

默认值也是esbuild。有些项目设置了为”terser“，terser会更小，但是占用的资源会更多。

设置了这个之后，可以稍微下降第一个峰值。

### css 配置去除charset限制规则

```ts
// vite.config.ts
css: {
    postcss: {
        plugins: [
          {
            postcssPlugin: 'internal:charset-removal',
            AtRule: {
              charset: (atRule) => {
                if (atRule.name === 'charset') {
                  atRule.remove();
                }
              }
            }
          }
        ],
    }
}
```

参考：https://github.com/evanw/esbuild/issues/1862

有伙伴测试，charset在行内插入，会导致很多warning，从而导致打包系统不断处理，时长不断增长。

最简单的测试，就是先不加入此条规则，查看打包后的产物css是否含有charset。

### 分包、去除压缩报告

仔细观察打包过程，两段CPU暴涨的时间分别发生在

render chunck....
computed gizp size....

我们前面两步，通过处理，只能降低第一段CPU暴涨到100%。

第二段的暴涨其实是在打包过程中，打包系统会做出gizp的压缩报告。

其实我觉得是不需要报告的，如要压缩，直接用"vite-plugin-compression"压缩就好了。

去除gizp的报告可以直接抹杀第二段CPU峰值。

```ts
// vite.config.ts
build: {
    // 去除gizp的报告、可以直接抹杀第二段峰值
    reportCompressedSize: false
}
```

分包的话就按照自己的项目来了，可以采用"rollup-plugin-visualizer"进行产物分析。

直接贴上我的分包，我这里用了ezuikit-js这个插件，这个插件包特别大，所以我把它分开了。


```ts
// vite.config.ts
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        ['ezuikit-js']: ['ezuikit-js'],
        ['element-plus']: ['element-plus'],
      },
    },
}
```


---

## 分包

聊一下vite打包上的优化，这里我们有一个vite工程，我们对它进行打包，打包完成后我们会发现，他的结果里面是把所有的东西全部合并到一个js里面。这个工程里面用到了vue、loadsh，像这些内容也会被合并到我们的打包结果里面去；

也就是说现在我们会遇到一个问题，就是我们打包出来的结果里面包含了loadsh、vue，还有我们自己的代码。


### 对象属性分包

你可能会问，这跟我有什么关系嘛？你可以想象得到，将来项目越来越大，变动比较大的是我们自己的业务代码，而这些第三方库，变动比较小，是相对比较稳定的；如果你不加处理的话，就意味着将来我们哪怕只改了我们自己的代码，整个打包出来的文件，它的文件指纹（就是 index-  后面这一块）会导致用户端每次更新必须要重新下载整个js

我们一般会怎么做呢？会把相对稳定的包给它踢出去，在打包结果里面，它们会形成独立的文件，这个文件可以是多个第三方库合并到一起的，也可以是单独的，这样一来由于你们比较稳定，所以里面的文件指纹也会相对比较稳定，不太容易发生变化；而将来这种不稳定的代码更新之后，用户只需要去下载业务代码就可以了，简称 分包。

在Vite里面有一个自动分包，使用动态导入 import('loadsh') ，在Vite中，开发环境用的是esBuild，打包用的是rollup，想要影响打包结果，就得去vite里面配置rollup的选项，因为rollup里面是可以进行分包的，通过配置 manualChunks ，这个chunks 跟webpack的含义是一样的，块 就是一个包，理清楚概念之后，就可以非常轻松的配置了；

在 vue.config.js 文件里面，使用build表示影响打包结果的配置，rollupOptios表示针对rollup的配置，剩下的里面就是吧rollup的配置给写进去就完事，使用 manulChunks，键表示包的名字，值表示包含那些模块；

```ts
rollupOptions: {
  output: {
    manualChunks: {
      a: ['vue', 'lodash']
    }
  }
}
```

npm run build 重新打包结果中出现了两个js文件，其中以a-开头的里面包含的内容就是loadsh和Vue;

### 函数分包

项目中用到的第三方库特别多怎么办？难道也打包成一个js文件嘛？显然不合理，如果其中一个库更新了，我能不能单独升级这个包？答案是可以的；

```ts
rollupOptions: {
  output: {
    manualChunks: {
      ['vue']: ['vue'],
      ['lodash']: ['lodash']
    }
  }
}
```

那么这样就完美了嘛？细心的小伙伴会发现，第三方包特别的情况，这样写下去，人都要麻了，那有没有更好的方案，有的！不把它配置为对象了，而是配置成函数 manualChunks(id) ，rollup在打包的时候，它会调用这个函数，把当前依赖模块的id传过来，我们可以打印id看一下有些啥！


打印结果里面主要包含 node_modules 和我们自己文件的路径，那我们能不能把 node_modules 合并成一个chunk 单独打包？

```ts
manualChunks(id) {
    if (id.includes('node_modules')) {
      return 'vendor'
    }
}
```
