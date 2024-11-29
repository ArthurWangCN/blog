---
tag: vue
---

# vue3 监听元素大小变化

在 Vue 3 中，你可以使用 ResizeObserver 来监听元素大小的变化。ResizeObserver 是浏览器提供的 API，用于监测元素的尺寸改变。

以下是一个示例，演示如何在 Vue 3 中监听元素大小变化：

```vue
<template>
  <div ref="element" class="resize-element">
    ...
  </div>
</template>

<script>
import { onMounted, onUnmounted, ref } from 'vue';

export default {
  mounted() {
    const element = ref(null);

    const resizeObserver = new ResizeObserver(entries => {
      // 处理大小变化的回调函数
      for (const entry of entries) {
        console.log('元素大小已变化', entry.target);
        // 在这里执行你想要的操作，比如更新数据或触发其他事件
      }
    });

    onMounted(() => {
      resizeObserver.observe(element.value);
    });

    onUnmounted(() => {
      resizeObserver.unobserve(element.value);
    });

    return {
      element
    };
  }
}
</script>

<style scoped>
.resize-element {
  width: 200px;
  height: 200px;
  background-color: lightgray;
}
</style>
```

在上述示例中，我们使用了 ref 来引用被监听的元素，并在 mounted 钩子函数中创建了一个 ResizeObserver 实例。然后，在 onMounted 钩子函数中使用 observe 方法来开始监听元素的大小变化。当元素大小发生变化时，ResizeObserver 的回调函数将被触发。
在回调函数中，你可以执行你想要的操作，例如更新相关的数据或触发其他事件。

最后，在 onUnmounted 钩子函数中使用 unobserve 方法停止监听，以避免内存泄漏。

请确保在支持 ResizeObserver 的浏览器环境中进行测试，并在必要时进行兼容性处理。

