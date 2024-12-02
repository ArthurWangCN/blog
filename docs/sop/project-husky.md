---
title: 使用 eslint + prettier + husky 打造规范化项目
date: 2023-03-12 00:58:24
tags: 工程化
categories: 工程化
description: 无论项目是个人开发还是多人开发，都应该有一个规范的代码格式。统一规范的代码风格在前端工程化中也是必不可少的一部分，项目初期如果没有定义好的代码规范，后期维护起来会非常难受，而 Husky 可以在项目中通过管理Git Hooks帮助我们检查代码规范。
---

# 使用 eslint + prettier + husky 打造规范化项目

## 准备
使用vue-ts模板创建项目，由于该篇文章的重点不是项目的搭建，所以这里就简单带过。

```sh
npm create vite my-vue-app --template vue-ts
```

## 为什么需要eslint和prettier

ESLint是一个用来识别 ECMAScript 并且按照规则给出报告的代码检测工具，使用它可以避免低级错误和统一代码的风格。如果每次在代码提交之前都进行一次eslint代码检查，就不会因为某个字段未定义为undefined或null这样的错误而导致服务崩溃，可以有效的控制项目代码的质量。

Prettier 是一个有主见的代码格式化程序。用来批量处理代码的统一，涉及引号，分号，换行，缩进。支持目前大部分语言处理，包括 JavaScript，Flow，TypeScript，CSS，SCSS，Less，JSX，Vue，GraphQL，JSON，Markdown。简而言之，这个工具能够使输出代码保持风格一致。

简单来说，就是eslint可以保证项目的质量，prettier可以保证项目的统一格式、风格。

## 配置eslint

执行安装命令：

```sh
pnpm add eslint -D
```

执行eslint初始化命令：

```sh
pnpm eslint --init
```

依次初始化选项（根据项目情况选择）：

```sh
(1) How would you like to use ESLint?
选择：To check syntax and find problems

(2) What type of modules does your project use?
选择：JavaScript modules (import/export)

(3) Which framework does your project use?
选择：Vue.js

(4) Does your project use TypeScript?
选择：Yes

(5) Where does your code run?
选择：Browser

(6) What format do you want your config file to be in?
选择：JavaScript

(7) Would you like to install them now?
选择：Yes

(8) Which package manager do you want to use?
选择：pnpm
```

依赖安装完成后，会生成.eslintrc.js配置文件：

```js
module.exports = {
    "env": {
        "browser": true,
        "es2021": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:vue/vue3-essential",
        "plugin:@typescript-eslint/recommended"
    ],
    "parserOptions": {
        "ecmaVersion": "latest",
        "parser": "@typescript-eslint/parser",
        "sourceType": "module"
    },
    "plugins": [
        "vue",
        "@typescript-eslint"
    ],
    "rules": {
    }
}
```

此时打开.eslintrc.js配置文件会出现一个报错，需要再env字段中增加node: true配置以解决eslint找不到module的报错

```diff
"env": {
    "browser": true,
    "es2021": true,
    // 新增
+        "node": true
},
```

在package.json文件中的script中添加lint命令：

```json
{
    "scripts": {
        // eslint . 为指定lint当前项目中的文件
        // --ext 为指定lint哪些后缀的文件
        // --fix 开启自动修复
        "lint": "eslint . --ext .vue,.js,.ts,.jsx,.tsx --fix"
    }
}
```

执行lint命令：

```sh
pnpm lint
```

这时候命令行中会显示出报错，意思就是在解析.vue后缀的文件时候出现解析错误parsing error。

查阅资料后发现，eslint默认是不会解析.vue后缀文件的。因此，需要一个额外的解析器来解析.vue后缀文件。

但是我们查看.eslintrc.js文件中的extends会发现已经有继承"plugin:vue/vue3-essential"的配置。然后在node_modules中可以找到eslint-plugin-vue/lib/cinfigs/essential，里面配置了extends是继承于同级目录下的base.js，在里面会发现parser: require.resolve('vue-eslint-parser')这个配置。因此，按道理来说应该是会解析.vue后缀文件的。

继续往下看.eslintrc.js文件中的extends会发现，extends中还有一个"plugin:@typescript-eslint/recommended"，它是来自于/node_modules/@typescript-eslint/eslint-plugin/dist/configs/recommended.js，查看该文件会发现最终继承于同级目录下的base.js文件。从该文件中可以发现parser: '@typescript-eslint/parser',配置。

按照.eslintrc.js文件中的extends配置的顺序可知，最终导致报错的原因就是@typescript-eslint/parser把vue-eslint-parser覆盖了。

```json
{
    "extends": [
        "eslint:recommended",
        "plugin:vue/vue3-essential",
        "plugin:@typescript-eslint/recommended"
    ],
}
```

查看eslint-plugin-vue官方文档可知。如果已经使用了另外的解析器（例如"parser": "@typescript-eslint/parser"），则需要将其移至parseOptions，这样才不会与vue-eslint-parser冲突。

修改.eslintrc.js文件：

```js
module.exports = {
    "env": {
        "browser": true,
        "es2021": true,
        "node": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:vue/vue3-essential",
        "plugin:@typescript-eslint/recommended"
    ],
    "parser": "vue-eslint-parser",
    "parserOptions": {
        "ecmaVersion": "latest",
        "parser": "@typescript-eslint/parser",
        "sourceType": "module"
    },
    "plugins": [
        "vue",
        "@typescript-eslint"
    ],
    "rules": {
    }
}
```

两个parser的区别在于，外面的parser用来解析.vue后缀文件，使得eslint能解析 `<template>` 标签中的内容，而parserOptions中的parser，即@typescript-eslint/parser用来解析vue文件中 `<script>` 标签中的代码。

此时，再执行pnpm lint，就会发现校验通过了。

**安装vscode插件ESLint**

如果写一行代码就要执行一遍lint命令，这效率就太低了。所以我们可以配合vscode的ESLint插件，实现每次保存代码时，自动执行lint命令来修复代码的错误。
在项目中新建.vscode/settings.json文件，然后在其中加入以下配置。

```json
{
    // 开启自动修复
    "editor.codeActionsOnSave": {
        "source.fixAll": false,
        "source.fixAll.eslint": true
    }
}
```

安装依赖说明：

- eslint： JavaScript 和 JSX 检查工具
- eslint-plugin-vue： 使用 ESLint 检查 .vue文件 的 `<template>` 和 `<script>`，以及.js文件中的Vue代码


## 配置prettier

执行安装命令：

```sh
pnpm add prettier -D
```

在根目录下新建.prettierrc.js

添加以下配置，更多配置可查看[官方文档](https://prettier.io/docs/en/options.html)

```js
module.exports = {
    // 一行的字符数，如果超过会进行换行，默认为80
    printWidth: 80, 
    // 一个tab代表几个空格数，默认为80
    tabWidth: 2, 
    // 是否使用tab进行缩进，默认为false，表示用空格进行缩减
    useTabs: false, 
    // 字符串是否使用单引号，默认为false，使用双引号
    singleQuote: true, 
    // 行位是否使用分号，默认为true
    semi: false, 
    // 是否使用尾逗号，有三个可选值"<none|es5|all>"
    trailingComma: "none", 
    // 对象大括号直接是否有空格，默认为true，效果：{ foo: bar }
    bracketSpacing: true
}
```

在package.json中的script中添加以下命令：

```json
{
    "scripts": {
        "format": "prettier --write \"./**/*.{html,vue,ts,js,json,md}\"",
    }
}
```

运行该命令，会将我们项目中的文件都格式化一遍，后续如果添加其他格式的文件，可在该命令中添加，例如：.less后缀的文件

**安装vscode的Prettier - Code formatter插件**

安装该插件的目的是，让该插件在我们保存的时候自动完成格式化

在.vscode/settings.json中添加一下规则

```json
{
    // 保存的时候自动格式化
    "editor.formatOnSave": true,
    // 默认格式化工具选择prettier
    "editor.defaultFormatter": "esbenp.prettier-vscode"
}
```

## 解决eslint与prettier的冲突

在理想的状态下，eslint与prettier应该各司其职。eslint负责我们的代码质量，prettier负责我们的代码格式。但是在使用的过程中会发现，由于我们开启了自动化的eslint修复与自动化的根据prettier来格式化代码。所以我们已保存代码，会出现屏幕闪一起后又恢复到了报错的状态。

这其中的根本原因就是eslint有部分规则与prettier冲突了，所以保存的时候显示运行了eslint的修复命令，然后再运行prettier格式化，所以就会出现屏幕闪一下然后又恢复到报错的现象。这时候你可以检查一下是否存在冲突的规则。

查阅资料会发现，社区已经为我们提供了一个非常成熟的方案，即eslint-config-prettier + eslint-plugin-prettier。

- eslint-plugin-prettier： 基于 prettier 代码风格的 eslint 规则，即eslint使用pretter规则来格式化代码。
- eslint-config-prettier： 禁用所有与格式相关的 eslint 规则，解决 prettier 与 eslint 规则冲突，确保将其放在 extends 队列最后，这样它将覆盖其他配置

安装依赖：

```sh
pnpm add eslint-config-prettier eslint-plugin-prettier -D
```

在 .eslintrc.json中extends的最后添加一个配置：

```diff
{ 
    extends: [
    'eslint:recommended',
    'plugin:vue/vue3-essential',
    'plugin:@typescript-eslint/recommended',
+    // 新增，必须放在最后面
+    'plugin:prettier/recommended' 
  ],
}
```

最后重启vscode，你会发现冲突消失了，eslint与prettier也按照我们预想的各司其职了。


## 配置styleling

stylelint为css的lint工具。可格式化css代码，检查css语法错误与不合理的写法，指定css书写顺序等...

**安装依赖**

由于我的项目使用的less预处理器，因此配置的为less相关的，项目中使用其他预处理器的可以按照该配置方法改一下就好

stylelint v13版本将css, parse CSS(如SCSS,SASS),html内的css(如*.vue中的style)等编译工具都包含在内。但是v14版本没有包含在内，所以需要安装需要的工具

```sh
pnpm add stylelint postcss postcss-less postcss-html stylelint-config-prettier stylelint-config-recommended-less stylelint-config-standard stylelint-config-standard-vue stylelint-less stylelint-order -D
```

依赖说明：

- stylelint: css样式lint工具
- postcss: 转换css代码工具
- postcss-less: 识别less语法
- postcss-html: 识别html/vue 中的<style></style>标签中的样式
- stylelint-config-standard: Stylelint的标准可共享配置规则，详细可查看官方文档
- stylelint-config-prettier: 关闭所有不必要或可能与Prettier冲突的规则
- stylelint-config-recommended-less: less的推荐可共享配置规则，详细可查看官方文档
- stylelint-config-standard-vue: lint.vue文件的样式配置
- stylelint-less: stylelint-config-recommended-less的依赖，less的stylelint规则集合
- stylelint-order: 指定样式书写的顺序，在.stylelintrc.js中order/properties-order指定顺序

增加.stylelintrc.js配置文件

```js
module.exports = {
  extends: [
    'stylelint-config-standard',
    'stylelint-config-prettier',
    'stylelint-config-recommended-less',
    'stylelint-config-standard-vue'
  ],
  plugins: ['stylelint-order'],
  // 不同格式的文件指定自定义语法
  overrides: [
    {
      files: ['**/*.(less|css|vue|html)'],
      customSyntax: 'postcss-less'
    },
    {
      files: ['**/*.(html|vue)'],
      customSyntax: 'postcss-html'
    }
  ],
  ignoreFiles: [
    '**/*.js',
    '**/*.jsx',
    '**/*.tsx',
    '**/*.ts',
    '**/*.json',
    '**/*.md',
    '**/*.yaml'
  ],
  rules: {
    'no-descending-specificity': null, // 禁止在具有较高优先级的选择器后出现被其覆盖的较低优先级的选择器
    'selector-pseudo-element-no-unknown': [
      true,
      {
        ignorePseudoElements: ['v-deep']
      }
    ],
    'selector-pseudo-class-no-unknown': [
      true,
      {
        ignorePseudoClasses: ['deep']
      }
    ],
    // 指定样式的排序
    'order/properties-order': [
      'position',
      'top',
      'right',
      'bottom',
      'left',
      'z-index',
      'display',
      'justify-content',
      'align-items',
      'float',
      'clear',
      'overflow',
      'overflow-x',
      'overflow-y',
      'padding',
      'padding-top',
      'padding-right',
      'padding-bottom',
      'padding-left',
      'margin',
      'margin-top',
      'margin-right',
      'margin-bottom',
      'margin-left',
      'width',
      'min-width',
      'max-width',
      'height',
      'min-height',
      'max-height',
      'font-size',
      'font-family',
      'text-align',
      'text-justify',
      'text-indent',
      'text-overflow',
      'text-decoration',
      'white-space',
      'color',
      'background',
      'background-position',
      'background-repeat',
      'background-size',
      'background-color',
      'background-clip',
      'border',
      'border-style',
      'border-width',
      'border-color',
      'border-top-style',
      'border-top-width',
      'border-top-color',
      'border-right-style',
      'border-right-width',
      'border-right-color',
      'border-bottom-style',
      'border-bottom-width',
      'border-bottom-color',
      'border-left-style',
      'border-left-width',
      'border-left-color',
      'border-radius',
      'opacity',
      'filter',
      'list-style',
      'outline',
      'visibility',
      'box-shadow',
      'text-shadow',
      'resize',
      'transition'
    ]
  }
}
```

安装vscode的Stylelint插件

安装该插件可在我们保存代码时自动执行stylelint

在.vscode/settings.json中添加一下规则

```diff
{
  // 开启自动修复
  "editor.codeActionsOnSave": {
    "source.fixAll": false,
    "source.fixAll.eslint": true,
+   "source.fixAll.stylelint": true
  },
  // 保存的时候自动格式化
  "editor.formatOnSave": true,
  // 默认格式化工具选择prettier
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  // 配置该项，新建文件时默认就是space：2
  "editor.tabSize": 2,
  // stylelint校验的文件格式
+ "stylelint.validate": ["css", "less", "vue", "html"]
}
```

## 配置husky

> You can use it to **lint your commit messages**, **run tests**, **lint code**, etc... when you commit or push. Husky supports all Git hooks.

虽然上面已经配置好了eslint、preitter与stylelint，但是还是存在以下问题。

对于不使用vscode的，或者没有安装eslint、preitter与stylelint插件的同学来说，就不能实现在保存的时候自动的去修复与和格式化代码。

这样提交到git仓库的代码还是不符合要求的。因此需要引入强制的手段来保证提交到git仓库的代码时符合我们的要求的。

husky是一个用来管理git hook的工具，git hook即在我们使用git提交代码的过程中会触发的钩子。


安装依赖

```sh
pnpm add husky -D
```

在package.json中的script中添加一条脚本命令

```json
{
    "scripts": {
        "prepare": "husky install"
    },
}
```

该命令会在pnpm install之后运行，这样其他克隆该项目的同学就在装包的时候就会自动执行该命令来安装husky。这里我们就不重新执行pnpm install了，直接执行pnpm prepare，这个时候你会发现多了一个.husky目录。

然后使用husky命令添加pre-commit钩子，运行

```sh
pnpm husky add .husky/pre-commit "pnpm lint && pnpm format && pnpm lint:style"
```

执行完上面的命令后，会在.husky目录下生成一个pre-commit文件

```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint && pnpm format
```

现在当我们执行git commit的时候就会执行pnpm lint与pnpm format，当这两条命令出现报错，就不会提交成功。以此来保证提交代码的质量和格式。


## 最后

本篇文章主要是以一个vue-ts-vite的项目的搭建为基础，在项目中引入eslint + prettier + husky来规范项目。完整的代码上传至github，[点击此处](https://github.com/yusongh/vue3-ts-vite-eslint-prettier)可以查看完整代码。