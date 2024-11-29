---
title: 前端小组代码审查相关
date: 2022-02-20 21:18:25
tags: 前端
categories: 前端
description: 代码审查（Code review）是指对计算机源代码系统化地审查，常用软件同行评审的方式进行，其目的是在找出及修正在软件开发初期未发现的错误，提升软件质量及开发者的技术。
---

## 审查原则
1. 保证代码可读性和健壮性
2. 团队知识共享
3. 尽早定位和解决bug

## 审查内容
### 一般项
- 是否实现预期的功能 ✅
- 逻辑是否正确完备 ✅
- 是否简单易懂（KISS） ✅
- 是否有更好的实现方式 ✅
- 是否符合规范（见审查清单） ✅
- 是否存在多余的或重复的代码（DRY） ✅
- 是否尽可能地模块化 ✅
- 是否有可以删除的日志或调试代码 ✅

### 注释/文档
- 函数是否有注释（公用的函数强制注释） ✅
- 是否对边界情况或特殊操作有说明（如 hack）（必须注释） ✅
- 是否对未完成的内容做说明 TODO：XXX ✅

### 安全
- 是否处理边界情况 ✅
- 是否对输入做校验 ✅
- 是否对异常做处理 ✅

## 审查方式
### 小片段式
小片段式的代码审查，又称事前审查，是在代码合并到主分支之前就做的一种审查形式。如果发现了问题，在代码合并之前就会进行需改。

### 大规模式
大规模式的代码审查，又称事后审查，大模块式的Code Review会议，进行大规模的代码检视，提出问题并记录，之后可以以重构的方式来处理在会议上发现的问题。

## 审查清单
### 命名
**拼写正确**

VSCode 可安装 Code Spell Checker

**语义化**

- can：判断是否可执行某个动作
- has：判断是否含有某个值
- is：判断是否为某个值
- get：获取某个值
- set：设置某个值

常量均需命名且大写，使用 _ 连接 const MAX_VALUE = 9999

变量使用小驼峰（Camel Case）

```js
// 推荐：
let formInstance = 'foo'
// 不推荐：
let form_instance = 'foo'
```

### 函数
函数参数超过3个时，尽量使用 ES6 解构语法， 避免参数引入顺序问题

```js
function setContact({ name, phone, email }) {
        // ...
}
```

使用参数默认值，使用参数默认值代替使用条件语句进行赋值

```js
// 推荐
function foo(bar = "baz") {
  // ...
}
// 不推荐
function foo(bar) {
  const mybar = bar || "baz";
  // ...
}
```

尽量使用纯函数，避免副作用

```js
// 纯函数
function checkAge(age) {
  let mini = 18
  return age >= mini
}
// 非纯函数
let mini= 18
function checkAe(age) {
  return age >= mini
}
```

尽早 return：通过用 if/return 替换 if/else 来减少缩进（代码可读性）

```js
function foo() {
  if (error) return
  
  // doStuff...
}
```

### 性能相关
1. 避免全局变量
2. 清除无用的定时器、事件监听、event bus、闭包内变量
3. 优化消耗性能的操作，如递归、cloneDeep 等
4. 缓存复杂计算结果
5. 合理使用节流、防抖
6. 减少页面回流、重绘
7. 图片 CDN 


## Vue相关

### 组件（文件）名称
1. 所有组件使用 SFC + TSX 形式
2. SFC 中的标签顺序统一为 `<template>、<script>、<style>`
3. 凡是跟组件名相关的（组件名、文件名及在模板中使用），都必须采用大驼峰 PascalCase 形式
4. 一般化描述放前，特殊化描述放后，例：

```js
SearchButtonClear.vue
SearchButtonRun.vue
SearchInput.vue
SearchInputQuery.vue
SettingsCheckbox.vue
SettingsTerms.vue 
```

### 属性相关
1. defineProps、defineEmits 要用类型声明的方式，而非运行时声明
2. 始终使用指令缩写，即 : @ #

### 性能
1. 避免非必要更新，合理使用 v-once、v-memo、computed（Vue: When a computed property can be the wrong tool）
例如 props 使用 isActive 而不是 activeId，导致每次 id 变化使所有子组件都更新
2. 代码复用，stateful logic 使用自定义 hooks，以 use 开头
3. setup await？hook 放前


## 开发流程相关
### 分支
👉 master
- 主分支，稳定版本，用于部署生产环境
- 不允许直接 push 代码，只能 Gitlab 上提交 merge request，且只允许 hotfix、release 分支代码合入
- 每次发版成功后打 tag，加上日期
- 常驻分支

👉 release
- 预发布分支
- 验证过程中，bug 在 dev 分支自测，合入 test 分支提交给测试人员测试，测试通过后合入 release 分支验证
- 验证通过后，合入 master 分支，master 分支发布版本
- 常驻分支

👉 test
- 提测分支
- 常驻分支

👉 dev
- 开发分支，包括新特性和 bug 修复，始终保持最新代码
- 常驻分支

👉 feature
- 新功能，以 master 为基础创建
- 命名以 feat/ 开头
- ❗️需 push 到远程
- 临时分支，合并后删除

fix
- 修复非生产 bug
- 命名以 fix/ 开头
- 修复完成后合入 dev 验证
- 临时分支，合并后删除

👉 hotfix
- 紧急 bug 修复分支，修复生产 bug
- 命名以 hotfix/ 开头
- 修复完成后合入 master 验证
- 临时分支，合并后删除

### Commit 提交规范
1. 提交格式（考虑 commitizen）
`<type>(<scope>): <subject>`

type
- feat: 新功能/特性
- fix: 修复 bug
- perf: 修改代码，以提高性能
- refactor: 代码重构
- docs: 文档修改
- style: 代码格式修改, 注意不是 css 修改（例如分号修改）
- test: 测试用例新增、修改
- build: 影响项目构建或依赖项修改
- revert: 恢复上一次提交
- ci: 持续集成相关文件修改
- chore: 其他修改（不在上述类型中的修改）
- release: 发布新版本

scope
commit 影响的功能或文件范围，比如: route、component、utils、build...

subject
commit message 的概述

2. husky pre-commit


## 环境说明
- 开发环境 Develop
用于开发人员开发新特性，修复缺陷，自测使用
- 测试环境 Test
用于测试人员测试，提出缺陷
- 预发布环境 Release
用于上线前的真实环境验证
- 生产环境 Production
用于发布稳定版本，打标签