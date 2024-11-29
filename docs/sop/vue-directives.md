---
tag: vue
---

# vue3 实现长按事件

## v-longpress
在我们熟悉的HTML事件 中并没有规定所谓的长按事件，网上大部分实现方式都是通过间接的方法去实现，大致过程就是：

- 当用户按下鼠标左键触发 mousedown() 事件，则我们启动一个计时器，设定一个时间阈值，开始计时。
- 如果在阈值内 mouseup() 事件被触发了，我们就认为只是个普通点击事件，如果超过阈值后才触发事件则我们认为是长按事件。

代码：

```ts
app.directive('longpress', {
  beforeMount(el, binding) {
    const cb = binding.value;
    el.$duration = binding.arg || 3000; // 获取长按时长, 默认3秒执行长按事件
    if(typeof cb !== 'function') return console.warn('v-longpress指令必须接收一个回调函数');
    let timer = null;
    const add = (e) => {
      // 排除点击与右键情况, event.button: 0-左键  2-右键
      if(e.type === 'click' && e.button !== 0) return;
      e.preventDefault();
      if(timer === null) {
        timer = setTimeout(() => {
          cb();
          timer = null;
        }, el.$duration)
      }
    }
    const cancel = () => {
      if (timer !== null) {
        clearTimeout(timer);
        timer = null;
      }
    }

    // 添加计时器
    el.addEventListener('mousedown', add);
    el.addEventListener('touchstart', add);
    // 取消计时器
    el.addEventListener('click', cancel);
    el.addEventListener('mouseout', cancel);
    el.addEventListener('touchend', cancel)
    el.addEventListener('touchcancel', cancel)
  },
  updated(el, binding) {
    // 可以实时更新时长
    el.$duration = binding.arg;
  },
  unmounted(el) {
    el.removeEventListener('mousedown', () => {});
    el.removeEventListener('touchstart', () => {});
    el.removeEventListener('click', () => {});
    el.removeEventListener('mouseout', () => {});
    el.removeEventListener('touchend', () => {});
    el.removeEventListener('touchcancel', () => {});
  }
})
```

使用：

```vue
<template>
  <button v-longpress:[1000]="longpress">按钮</button>
</template>
```


来源： https://vue-js.com/topic/60c05c6496b2cb0032c38f47
