---
tag: CSS
---

# CSS @media print控制浏览器web打印样式

浏览器打印，可以通过 window.print() 、document.execCommand(‘print’) 调用浏览器打印。不同浏览器的区别：在Safari和Chrome都会弹起打印预览的窗口，FireFox和 IE 没有预览而是直接让你选择打印机，但是直接使用浏览器打印虽然省事，但是存在很多问题，无法满足我们的打印需求：

1、打印的是整个网页，不能打印局部内容；
2、打印不支持自定义分页行为，默认不支持批量打印；
3、打印的时候样式有问题，所见非所得；
4、打印可以准确识别的样式单位是绝对单位（如pt、mm、cm），对相对单位识别不同打印机可能会得到意想不到的结果；

`@media print`里面的内容只对打印出来的内容有效，之外的内容就是屏幕显示的样式。

```css
@media print {}{
  .class {
    color: black;
  }
}
```

打印默认是A4的尺寸，如果超过了会被隐藏，在css控制合适的缩放比例:

```css
html{
  zoom:56%;
}
```

用于伪类修改样式

:left、:right、:first。

通过:left 和 :right设置左右页面不同样式，并不代表用户代理会将页面双面打印

```css
/* 通过分别设置左页和右页不同的左右页面距，为装订边留出更多的空间 */
@page :left {
    margin-left: 2.5cm;
    margin-right: 2.7cm;
}

@page :right {
    margin-left: 2.7cm;
    margin-right: 2.5cm;
}
```

:first用于匹配到文档的第一页

```css
/* 首页上页边距设置为 10cm */
@page :first {
    margin-top: 10cm; 
}
```

分页：page-break-before

page-break-before用于设置元素前面的分页行为

```css
div {
   page-break-before: avoid;
}
```


