---
date: 2022.11.22 11:11:11
---

# CSS 实现多行文本“展开收起”

实现这一类布局和交互难点主要有以下几点

位于多行文本右下角的“展开收起”按钮
“展开”和“收起”两种状态的切换
当文本不超过指定行数时，不显示“展开收起”按钮

说实话，之前单独看这个布局，即使借助 JavaScript 也不是一件容易的事啊（需要计算文字宽度动态截取文本，vue-clamp就是这么做的），更别说下面的交互和判断逻辑了，不过经过我的一番琢磨，其实纯 CSS 也能完美实现的，下面就一步一步来看看如何实现吧~

## 一、位于右下角的“展开收起”按钮

很多设计同学都喜欢这样的设计，把按钮放在右下角，和文本 混合 在一起，而不是单独一行，视觉上可能更加舒适美观。先看看多行文本截断吧，这个比较简单

**1. 多行文本截断**

假设有这样一个 html 结构

```html
<div class="text">
  浮动元素是如何定位的
正如我们前面提到的那样，当一个元素浮动之后，它会被移出正常的文档流，然后向左或者向右平移，一直平移直到碰到了所处的容器的边框，或者碰到另外一个浮动的元素。
</div>
```

多行文本超出省略大家应该很熟悉这个了吧，主要用到 **line-clamp**，关键样式如下

```css
.text {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

**2. 右下角环绕效果**

提到文本环绕效果，一般能想到 浮动 float，没错，千万不要以为浮动已经是过去式了，具体的场景还是很有用的。比如下面放一个按钮，然后设置浮动

```html
<div class="text">
  <button class="btn">展开</button>
  浮动元素是如何定位的
正如我们前面提到的那样，当一个元素浮动之后，它会被移出正常的文档流，然后向左或者向右平移，一直平移直到碰到了所处的容器的边框，或者碰到另外一个浮动的元素。
</div>
```

```css
.btn {
  float: left;
  /*其他装饰样式*/
}
```

设置右浮动

```css
.btn {
  float: right;
  /*其他装饰样式*/
}
```

这时已经有了 环绕 的效果了，只是位于右上角，如何将按钮移到右下角呢？先尝试一下 margin

```css
.btn {
  float: right;
  margin-top: 50px;
  /*其他装饰样式*/
}
```

可以看到，虽然按钮到了右下角，但是文本却没有环绕按钮上方的空间，空出了一大截，无能为力了吗？

虽然 margin 不能解决问题，但是整个文本还是受到了浮动按钮的影响，如果有多个浮动元素会怎么样呢？这里用伪元素来 ::before 代替

```css
.text::before{
  content: '';
  float: right;
  width: 10px;
  height: 50px;/*先随便设置一个高度*/
  background: red
}
```

现在按钮到了伪元素的左侧，如何移到下面呢？很简单，清除一下浮动 clear: both; 就可以了

```css
.btn {
  float: right;
  clear: both;
  /*其他装饰样式*/
}
```

可以看到，现在文本是完全环绕在右侧的两个浮动元素了，只要把红色背景的伪元素宽度设置为0（或者不设置宽度，默认就是 0），就实现了右下角环绕的效果

```css
.text::before{
  content: '';
  float: right;
  width: 0; /*设置为0，或者不设置宽度*/
  height: 50px;/*先随便设置一个高度*/
}
```

**3. 动态高度**

上面虽然完成了右下加环绕，但是高度是固定的，如何动态设置呢？这里可以用到 calc 计算，用整个容器高度减去按钮的高度即可，如下

```css
.text::before{
  content: '';
  float: right;
  width: 0;
  height: calc(100% - 24px);
}
```

很可惜，好像并没有什么效果，打开控制台看看，结果发现 calc(100% - 24px) 计算高度为 0

原因其实很容易想到，就是高度 100% 失效的问题，关于这类问题网上的分析有很多，通常的解决方式是给父级指定一个高度，但是这里的高度是动态变化的，而且还有展开状态，高度更是不可预知，所以设置高度不可取。
除此之外，其实还有另一种方式，那就是利用 flex 布局。大概的方法就是在 flex 布局 的子项中，可以通过百分比来计算变化高度，具体可参考 w3.org 中关于 css-flexbox 的描述

> If the flex item hasalign-self: stretch, redo layout for its contents, treating this used size as its definite cross size so that percentage-sized children can be resolved.

因此，这里需要给 .text 包裹一层，然后设置 display: flex

```html
<div class="wrap">
  <div class="text">
    <button class="btn">展开</button>
    浮动元素是如何定位的
  正如我们前面提到的那样，当一个元素浮动之后，它会被移出正常的文档流，然后向左或者向右平移，一直平移直到碰到了所处的容器的边框，或者碰到另外一个浮动的元素。
  </div>
</div>
```

```css
.wrap{
  display: flex;
}
```

实践下来，display: grid 和 display: -webkit-box 同样有效，原理类似 这样下来，刚才的计算高度就生效了，改变文本的行数，同样位于右下角~

除此之外，动态高度也可以采用负的 margin 来实现（性能会比 calc 略好一点）

```css
.text::before{
  content: '';
  float: right;
  width: 0;
  /*height: calc(100% - 24px);*/
  height: 100%;
  margin-bottom: -24px;
}
```

到这里，右下角环绕的效果就基本完成，省略号也是位于展开按钮之前的。

**4.其他浏览器的兼容处理**

上面的实现是最完美的处理方式。原本以为兼容性没什么大问题的，毕竟只用到了文本截断和浮动，-webkit-line-clamp 虽然是 -webkit- 前缀，不过 firefox 也是支持的，打开一看傻了眼，safari 和 firefox 居然全乱了！

这就有点难受了，前面那么多努力都白费了吗？不可能不管这两个，不然就只能是 demo 了，无法用于生产环境。

赶紧打开控制台看看是什么原因。一番查找，结果发现是**display: -webkit-box**！设置该属性后，原本的文本好像变成了一整块，浮动元素也无法产生环绕效果，去掉之后浮动就正常了

那么问题来了：没有 display: -webkit-box 怎么实现多行截断呢 ？

其实上面的努力已经实现了右下角环绕的效果，如果在知道行数的情况下设置一个最大高度，是不是也完成了多行截断呢？为了便于设置高度，可以添加一个行高 line-height，如果需要设置成 3 行，那高度就设置成line-height * 3

```css
.text {
  /*
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  */
  line-height: 1.5;
  max-height: 4.5em;
  overflow: hidden;
}
```

为了方便更好的控制行数，这里可以把常用的行数通过属性选择器独立出来（通常不会太多），如下

```css
[line-clamp="1"] {
  max-height: 1.5em;
}
[line-clamp="2"] {
  max-height: 3em;
}
[line-clamp="3"] {
  max-height: 4.5em;
}
...
```

```html
<!--3行-->
<div class="text" line-clamp="3">
...
</div>
<!--5行-->
<div class="text" line-clamp="5">
 ...
</div>
```

可以看到基本上正常了，除了没有省略号，现在加上省略号吧，跟在展开按钮之前就可以了，可以用伪元素实现

```css
.btn::before{
  content: '...';
  position: absolute;
  left: -10px;
  color: #333;
  transform: translateX(-100%)
}
```

这样，Safari 和 Firefox 的兼容布局基本上就完成了。

## 二、“展开”和“收起”两种状态

提到 CSS 状态切换，大家都能想到 input type="checkbox" 吧。这里我们也需要用到这个特性，首先加一个 input，然后把之前的 button 换成 label ，并且通过 for 属性关联起来

```html
<div class="wrap">
  <input type="checkbox" id="exp">
  <div class="text">
    <label class="btn" for="exp">展开</label>
    浮动元素是如何定位的
  正如我们前面提到的那样，当一个元素浮动之后，它会被移出正常的文档流，然后向左或者向右平移，一直平移直到碰到了所处的容器的边框，或者碰到另外一个浮动的元素。
  </div>
</div>
```

这样，在点击 label 的时候，实际上是点击了 input 元素，现在来添加两种状态，分别是只显示 3 行和不做行数限制

```css
.exp:checked+.text{
  -webkit-line-clamp: 999; /*设置一个足够大的行数就可以了*/
}
```

兼容版本可以直接设置最大高度 max-height 为一个较大的值，或者直接设置为 none

```css
.exp:checked+.text{
  max-height: none;
}
```

这里还有一个小问题，“展开”按钮在点击后应该变成“收起”，如何修改呢？

有一个技巧，凡是碰到需要动态修改内容的，都可以使用伪类 content 生成技术，具体做法就是去除或者隐藏按钮里面的文字，采用伪元素生成

```html
<label class="btn" for="exp"></label><!--去除按钮文字-->
```

```css
.btn::after{
  content:'展开' /*采用content生成*/
}
```

添加 :checked 状态

```css
.exp:checked+.text .btn::after{
  content:'收起'
}
```

兼容版本由于前面的省略号是模拟出来的，不能自动隐藏，所以需要额外来处理

```css
.exp:checked+.text .btn::before {
    visibility: hidden; /*在展开状态下隐藏省略号*/
}
```

还有一点，如果给max-height设置一个合适的值，注意，是合适的值，具体原理可以参考 [CSS 奇技淫巧：动态高度过渡动画](https://github.com/chokcoco/iCSS/issues/91?fileGuid=XtpJhGpvWxj6qcTr)，还能加上过渡动画

```css
.text{
  transition: .3s max-height;
}
.exp:checked+.text{
  max-height: 200px; /*超出最大行高度就可以了*/
}
```

## 三、文本行数的判断

上面的交互已经基本满足要求了，但是还是会有问题。比如当文本较少时，此时是没有发生截断，也就是没有省略号的，但是“展开”按钮却仍然位于右下角，如何隐藏呢？

通常 js 的解决方式很容易，比较一下元素的 scrollHeight 和 clientHeight 即可，然后添加相对应的类名。下面是伪代码

```js
if (el.scrollHeight > el.clientHeight) {
  // 文本超出了
  el.classList.add('trunk')
} 
```

那么，CSS 如何实现这类判断呢？

可以肯定的是，CSS 是没有这类逻辑判断，大多数我们都需要从别的角度，采用 “障眼法” 来实现。比如在这个场景，当没有发生截断的时候，表示文本完全可见了，这时，如果在文本末尾添加一个元素（红色小方块），为了不影响原有布局，这里设置了绝对定位

```css
.text::after {
    content: '';
    width: 10px;
    height: 10px;
    position: absolute;
    background: red;
}
```

可以看到，这里的红色小方块是完全跟随省略号的。当省略号出现时，红色小方块必定消失，因为已经被挤下去了，这里把父级 overflow: hidden 暂时隐藏就能看到是什么原理了

然后，可以把刚才这个红色的小方块设置一个足够大的尺寸，比如 100% * 100%

```css
.text::after {
    content: '';
    width: 100%;
    height: 100%;
    position: absolute;
    background: red;
}
```

可以看到，红色的块块把右下角的都覆盖了，现在把背景改为白色（和父级同底色），父级 overflow: hidden重新加上

```css
.text::after {
    content: '';
    width: 100%;
    height: 100%;
    position: absolute;
    background: #fff;
}
```

现在展开以后，发现按钮不见（被刚才那个伪元素所覆盖，并且也点击不了），如果希望点击以后仍然可见呢？添加一下 :checked 状态即可，在展开时隐藏覆盖层

```css
.exp:checked+.text::after{
    visibility: hidden;
}
```

这样，就实现了在文字较少的情况下隐藏展开按钮的功能

需要注意的是，兼容版本可以支持到 IE 10+

## 四、总结和说明

总的来说，重点还是在布局方面，交互其实相对容易，整体实现的成本其实是很低的，也没有比较生僻的属性，除了布局方面 -webkit-box 貌似有点 bug （毕竟是 -webkit-内核 ， 火狐只是借鉴了过来，难免有些问题），幸运的是可以通过另一种方式实现多行文本截断效果，兼容性相当不错，基本上全兼容（IE10+），这里整理一下实现重点

- 文本环绕效果首先考虑浮动 float
- flex 布局子元素可以通过百分比计算高度
- 多行文本截断还可以结合文本环绕效果用max-height模拟实现
- 状态切换可以借助 checkbox
- CSS 改变文本可以采用伪元素生成
- 多利用 CSS 遮挡 “障眼法”

作者：阅文前端团队
链接：https://juejin.cn/post/6963904955262435336
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


## 五、完整代码

```html
<div class="wrapper">
  <input id="exp1" class="exp"  type="checkbox">
        <div class="text">
            <label class="btn" for="exp1"></label>
            浮动元素是如何定位的
正如我们前面提到的那样，当一个元素浮动之后，它会被移出正常的文档流，然后向左或者向右平移，一直平移直到碰到了所处的容器的边框，或者碰到另外一个浮动的元素。
在下面的图片中，有三个红色的正方形。其中有两个向左浮动，一个向右浮动。要注意到第二个向左浮动的正方形被放在第一个向左浮动的正方形的右边。如果还有更多的正方形这样浮动，它们会继续向右堆放，直到填满容器一整行，之后换行至下一行。
        </div>
    </div>

<div class="wrapper">
  <input id="exp2"  class="exp"  type="checkbox">
        <div class="text">
            
            <label class="btn" for="exp2"></label>
            浮动元素是如何定位的
正如我们前面提到的那样，当一个元素浮动之后，它会被移出正常的文档流，然后向左或者向右平移，一直平移直到碰到了所处的容器的边框，或者碰到另外一个浮动的元素。
        </div>
    </div>
```

```css
.wrapper {
  display: flex;
  margin: 50px auto;
  width: 800px;
  overflow: hidden;
  border-radius: 8px;
  padding: 15px ;
  box-shadow: 20px 20px 60px #bebebe,
    -20px -20px 60px #ffffff;
}
.text {
  font-size: 20px;
  overflow: hidden;
  text-overflow: ellipsis;
  text-align: justify;
  /* display: flex; */
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  position: relative;
}
.text::before {
  content: '';
  height: calc(100% - 24px);
  float: right;
}
.text::after {
  content: '';
  width: 999vw;
  height: 999vw;
  position: absolute;
  box-shadow: inset calc(100px - 999vw) calc(30px - 999vw) 0 0 #fff;
    margin-left: -100px;
}
.btn{
  float: right;
  clear: both;
  margin-left: 10px;
  font-size: 16px;
  padding: 0 8px;
  background: #3F51B5;
  line-height: 24px;
  border-radius: 4px;
  color:  #fff;
  cursor: pointer;
  /* margin-top: -30px; */
}
.btn::before{
  content:'展开'
}
.exp{
  display: none;
}
.exp:checked+.text{
  -webkit-line-clamp: 999;
}
.exp:checked+.text::after{
  visibility: hidden;
}
.exp:checked+.text .btn::before{
  content:'收起'
}
```

封装vue3组件：

```vue
<script setup lang="ts">
withDefaults(
  defineProps<{
    text?: string;
  }>(),
  {
    text: "",
  },
);

const random = Math.random();
</script>

<template>
  <div class="container" :title="text">
    <div class="line-clamp">
      <input
        :id="`checkbox${random}`"
        type="checkbox"
        class="line-clamp-checkbox"
      />
      <div class="line-clamp-content">
        <label
          :for="`checkbox${random}`"
          class="line-clamp-btn line-clamp-btn-open"
        >
          <span>展开</span>
        </label>
        {{ text }}
        <label
          :for="`checkbox${random}`"
          class="line-clamp-btn line-clamp-btn-close"
        >
          <span>收起</span>
        </label>
      </div>
    </div>
  </div>
</template>

<style scoped lang="less">
.container {
  .line-clamp {
    display: flex;
  }

  .line-clamp-content {
    max-height: 3em;
    line-height: 1.5;
    overflow: hidden;
    text-overflow: ellipsis;
    text-align: justify;
    position: relative;
  }

  .line-clamp-content::before {
    content: "";
    width: 0;
    float: right;
    height: calc(100% - 1.5em + 1px);
  }

  .line-clamp-content::after {
    content: "";
    position: absolute;
    width: 200vw;
    height: 100vh;
    box-shadow: inset -100vw calc(1.5em - 100vh) 0 0 #fff;
    margin-left: -100vw;
  }

  .line-clamp-btn {
    float: right;
    border-radius: 0.2em;
    padding: 0 0.2em;
    clear: both;
  }

  .line-clamp-btn > span {
    color: #f47c32;
    cursor: pointer;
  }

  .line-clamp-btn-open {
    margin-left: 1.3em;
    transform: translate(0, -1px);
    position: relative;
    cursor: pointer;
  }

  .line-clamp-btn-open::before {
    content: "...";
    transform: translate(-1.3em, 0);
    position: absolute;
  }

  .line-clamp-btn-close,
  .line-clamp-checkbox,
  .line-clamp-checkbox:checked + .line-clamp-content::before,
  .line-clamp-checkbox:checked + .line-clamp-content::after,
  .line-clamp-checkbox:checked + .line-clamp-content .line-clamp-btn-open {
    display: none;
  }

  .line-clamp-checkbox:checked + .line-clamp-content {
    max-height: inherit;
  }

  .line-clamp-checkbox:checked + .line-clamp-content .line-clamp-btn-close {
    display: inherit;
  }
}
</style>
```



