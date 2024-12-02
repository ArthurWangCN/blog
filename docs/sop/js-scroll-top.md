---
date: 2022.11.22 11:11:11
---

# JavaScript 实现滚动到顶部

在 JavaScript 中，你可以使用以下几种方法将页面滚动到顶部：

1. **使用 `window.scrollTo()` 方法：**

   使用 `window.scrollTo()` 方法，你可以将页面的滚动位置设置为顶部坐标 `(0, 0)`。这是最基本的方法，适用于大多数情况。

   ```javascript
   window.scrollTo(0, 0);
   ```

2. **使用 `window.scroll()` 方法：**

   `window.scroll()` 方法可以实现平滑的滚动效果。你可以指定滚动到顶部的持续时间和滚动的方式（如平滑滚动）。

   ```javascript
   window.scroll({
     top: 0,
     left: 0,
     behavior: 'smooth' // 平滑滚动
   });
   ```

3. **使用 `window.requestAnimationFrame()` 实现平滑滚动：**

   如果你想要更高级的平滑滚动效果，可以使用 `window.requestAnimationFrame()` 来控制动画帧。

   ```javascript
   function scrollToTop() {
     const currentPosition = window.scrollY;
     if (currentPosition > 0) {
       window.requestAnimationFrame(scrollToTop);
       window.scrollTo(0, currentPosition - currentPosition / 8);
     }
   }
   
   scrollToTop();
   ```

   这个示例会平滑滚动页面，直到滚动到顶部为止。

4. **使用 jQuery（如果你在项目中使用 jQuery）：**

   如果你在项目中使用了 jQuery，可以使用 `animate()` 方法来实现平滑滚动到顶部。

   ```javascript
   $('html, body').animate({ scrollTop: 0 }, 'slow');
   ```

无论你选择哪种方法，都可以将页面滚动到顶部。根据你的需求和项目中的环境，选择最合适的方法。
