# sessionStorage 能在多个标签页之间共享数据吗？

> sessionStorage 不能在多个窗口或标签页之间共享数据，但是当通过 window.open 或链接打开新页面时(不能是新窗口)，新页面会复制前一页的 sessionStorage。

sessionStorage 就是会话级别的存储（关键在于会话）

**如何定义一个会话？**

- 在A页面点击超链接或者在控制台window.open打开页面B，都是属于当前页面的延续，属于一个会话。
- 在A页面已经打开的前提下，然后在新tab打开同域页面C，此时C和A页面无直接关系，不属于一个会话。

引用中已经说了，sessionSession并不是共享的，而是复制的。

也就是说，B页面打开的时候复制了A页面的sessionSession，仅仅是复制。此时，无论修改A页面的sessionStorage还是修改B页面的SessionStorage，都不会彼此影响。

也就是说两个页面存储的SessionStorage数据都不会同步变化（各自都是自己的存储，存储独立存在）。
