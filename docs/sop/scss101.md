---
title: scss 快速入门
date: 2023-03-06 22:38:12
tags: CSS
categories: CSS
description: CSS预处理器用一种专门的编程语言，进行Web页面样式设计，然后再编译成正常的CSS文件，以供项目使用。CSS预处理器为CSS增加一些编程的特性，无需考虑浏览器的兼容性问题。
---

# scss 快速入门

## 简介

CSS预处理器用一种专门的编程语言，进行Web页面样式设计，然后再编译成正常的CSS文件，以供项目使用。CSS预处理器为CSS增加一些编程的特性，无需考虑浏览器的兼容性问题。 比如说：Sass（SCSS）、LESS、Stylus、Turbine、Swithch CSS、CSS Cacheer、DT CSS等。都属于css预处理器。其中比较优秀的： Sass、LESS，Stylus。

sass 和 scss 的区别？

Sass的缩排语法，对于写惯css前端的web开发者来说很不直观，也不能将css代码加入到Sass里面，因此sass语法进行了改良，Sass 3就变成了Scss(sassy css)。与原来的语法兼容，只是用{}取代了原来的缩进。

## 准备工作

scss需要经过编译为css才能被浏览器识别，我这里只做一个小demo所以不上vue-cli，直接使用webpack进行编译。

首先安装css-loader、style-loader、node-sass、sass-loader：

```bash
npm install css-loader style-loader --save-dev
npm install node-sass sass-loader --save-dev
```

然后在webpack.config.js配置文件中添加对应的loader,完整的配置图如下：

```js
const path = require("path");
const {VueLoaderPlugin} = require('vue-loader');
module.exports = {
    entry: './webapp/App.js',
    output: {
        filename: 'App.js',
        path: path.resolve(__dirname, './dist')
    },
	module: {
		rules: [
            {
                test: /\.scss/,
                use: ['style-loader', 'css-loader','sass-loader']
            },
			{
				test: /\.vue$/,
				use: 'vue-loader'
			}
		]
	},
	plugins: [
		new VueLoaderPlugin()
	],
	mode: "production"
}
```

创建一个App.scss文件，接着在入口文件中引入：

```js
import './App.scss';
```

后面我们将在App.scss中编写scss代码。

注意：vite提供了对 .scss, .sass, .less, .styl 和 .stylus 文件的内置支持。没有必要为它们安装特定的 Vite 插件，但必须安装相应的预处理器依赖。

```bash
# .scss and .sass
npm add -D sass
# .less
npm add -D less
# .styl and .stylus
npm add -D stylus
```

如果使用的是单文件组件，可以通过 `<style lang="sass">`（或其他预处理器）自动开启。


## 正式开始

### 使用变量

SCSS中的变量以$开头。

```scss
$border-color:#aaa; //声明变量
.container {
$border-width:1px;
  border:$border-width solid $border-color; //使用变量
}
```

上述例子中定义了两个变量，其中$border-color在大括号之外称为全局变量，顾名思义任何地方都可以使用，$border-width是在.container之内声明的，是一个局部变量，只有.container内部才能使用。

编译后的CSS

```css
.container {
  border:1px solid #aaa; //使用变量
}
```

我们可以把SCSS看做一个模板引擎，编译的过程中用变量的值去替代变量所占据的位置。

tips:SCSS中变量名使用中划线或下划线都是指向同一变量的，上文中定义了一个变量$border-color，这时再定义一个变量 `$border_color:#ccc`,他们指向同一个变量，.container的值会被第二次定义的变量覆盖。

```scss
$border-color:#aaa; //声明变量
$border_color:#ccc;
.container {
  $border-width:1px;
  border:$border-width solid $border-color; //使用变量
}
编译后的CSS
.container {
  border:1px solid #ccc; //使用变量
}
```

这个例子中我们要知道

1. 变量名使用中划线或下划线都是指向同一变量的。
2. 后定义的变量声明会被忽略，但赋值会被执行，这一点和ES5中var声明变量是一样的。

### 嵌套规则
我们先来看一个例子：

```css
/*css*/
.container ul {
  border:1px solid #aaa;
  list-style:none;
}
.container ul:after {
  display:block;
  content:"";
  clear:both;
}
.container ul li {
  float:left;
}
.container ul li>a {
  display:inline-block;
  padding:6px 12px;
}
```

这是一个让列表元素横向排列的例子，我们在这个例子中写了很多重复的代码，.container写了很多遍，下面我将用SCSS简写上面的例子：

```scss
.container ul {
  border:1px solid #aaa;
  list-style:none;
  
  li {
    float:left;
    
    >a {
      display:inline-block;
      padding:6px 12px;
    }
  }
  
  &:after {
    display:block;
    content:"";
    clear:both;
  }
}
```

当然也可以嵌套属性：

```css
/*css*/
li {
  border:1px solid #aaa;
  border-left:0;
  border-right:0;
}
```

scss识别一个属性以分号结尾时则判断为一个属性，以大括号结尾时则判断为一个嵌套属性，规则是将外部的属性以及内部的属性通过中划线连接起来形成一个新的属性。

```scss
/*scss*/
li {
  border:1px solid #aaa {
    left:0;
    right:0;
  }
}
```

### 导入SCSS文件
大型项目中css文件往往不止一个，css提供了@import命令在css内部引入另一个css文件，浏览器只有在执行到@import语句后才会去加载对应的css文件，导致页面性能变差，故基本不使用。

#### 导入变量的优先级问题-变量默认值

```scss
/*App1.scss*/
$border-color:#aaa; //声明变量
@import App2.scss;  //引入另一个SCSS文件
.container {
  border:1px solid $border-color; //使用变量
}
/*App2.scss*/
$border-color:#ccc; //声明变量

/*生成的css文件*/
.container {
  border:1px solid #ccc; //使用变量
}
```

这可能并不是我们想要的，有时候我们希望引入的某些样式不更改原有的样式，这时我们可以使用变量默认值。

```scss
/*App1.scss*/
$border-color:#aaa; //声明变量
@import App2.scss;  //引入另一个SCSS文件
.container {
  border:1px solid $border-color; //使用变量
}
/*App2.scss*/
$border-color:#ccc !default; //声明变量

/*生成的css文件*/
.container {
  border:1px solid #aaa; //使用变量
}
```

导入的文件App2.scss只在文件中不存在$border-color时起作用，若App1.scss中已经存在了$border-color变量，则App2.scss中的$border-color不生效。

!default只能使用与变量中。

#### 嵌套导入
上一个例子中我们是在全局中导入的App2.scss，现在我们在为App2.scss添加一些内容，并在局部中导入。

```scss
/*App1.scss*/
$border-color:#aaa; //声明变量
.container {
  @import App2.scss;  //引入另一个SCSS文件
  border:1px solid $border-color; //使用变量
}
/*App2.scss*/
$border-color:#ccc !default; //声明变量
p {
  margin:0;
}

/*生成的css文件*/
.container {
  border:1px solid #aaa; //使用变量
}
.container p {
  margin:0;
}
```

可以看得出来，就是将App2.scss中的所有内容直接写入到App1.scss的.container选择器中。

#### 使用原生@import
前面我们说到基本不使用原生@import，但某些情况下我们不得不使用原生@import时了，SCSS也为我们处理了这种情况，直接导入css文件即可。

```scss
@import 'App.css';
```

### 注释
SCSS中的注释有两种：

1. `/*注释*/` 这种注释会被保留到编译后的css文件中。
2. `//注释` 这种注释不会被保留到编译后生成的css文件中。


### 混合器（函数）

#### 声明一个函数

使用@mixin指令声明一个函数，看一下自己的css文件，有重复的代码片段都可以考虑使用混合器将他们提取出来复用。

```scss
@mixin border-radius{
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
  color:red;
}
```

混合器作用域内的属性都是return的值，除此之外，还可以为函数传参数。

```scss
@mixin get-border-radius($border-radius,$color){
  -moz-border-radius: $border-radius;
  -webkit-border-radius: $border-radius;
  border-radius: $border-radius;
  color:$color;
}
```

也可以设置混合器的默认值。

```scss
@mixin get-border-radius($border-radius:5px,$color:red){
  -moz-border-radius: $border-radius;
  -webkit-border-radius: $border-radius;
  border-radius: $border-radius;
  color:$color;
}
```

#### 使用函数

使用函数的关键字为@include

```scss
.container {
  border:1px solid #aaa;
  @include get-border-radius;         //不传参则为默认值5px
  @include get-border-radius(10px,blue);   //传参
}
/*多个参数时，传参指定参数的名字，可以不用考虑传入的顺序*/
.container {
  border:1px solid #aaa;
  @include get-border-radius;         //不传参则为默认值5px
  @include get-border-radius($color:blue,$border-radius:10px);   //传参
}
```

我们可能会想到，直接将混合器写成一个class不就行了，但是写成一个class的时候是需要在html文件中使用的，而使用混合器并不需要在html文件中使用class既可达到复用的效果。

tips:混合器中可以写一切scss代码。

### 继承

继承是面向对象语言的一大特点，可以大大降低代码量。

使用%定义一个被继承的样式，类似静态语言中的抽象类，他本身不起作用，只用于被其他人继承。

```scss
%border-style {
  border:1px solid #aaa;
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}
```

通过关键字@extend即可完成继承。

```scss
.container {
	@extend %border-style;
}
```

上述例子中看不出混合器与继承之间的区别，那么下一个例子可以看出继承与混合器之间的区别。

```scss
.container {
	@extend %border-style;
	color:red;
}
.container1 {   //继承另一个选择器
	@extend .container;
}
```

### 其他

#### 操作符
SCSS提供了标准的算术运算符，例如+、-、*、/、%。

```scss
/*SCSS*/
width: 600px / 960px * 100%;
/*编译后的CSS*/
width: 62.5%;
```