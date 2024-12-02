---
date: 2022.11.22 11:11:11
---

# promise.all then里返回的一定是有序的吗？

>问：promise.all then里返回的一定是有序的吗？

答：

在 JavaScript 中，Promise.all() 方法会在所有传入的 Promise 都解决（fulfilled）时才会被解决，且返回的 Promise 的解决值是一个数组，包含了传入的 Promise 解决时的值，数组中的顺序与传入的 Promise 顺序相同。

但是，需要注意的是，Promise.all() 并不会保证 Promise 的执行顺序。这意味着即使传入的 Promise 在数组中的顺序不同，它们的执行顺序也可能是不同的。这是因为 JavaScript 是单线程的，Promise 的执行是异步的，它们可能会在不同的时间点开始执行。

下面是一个示例，演示了 Promise.all() 返回的数组顺序与传入 Promise 顺序相同，但实际执行顺序可能是不同的：

```js
const promise1 = new Promise(resolve => setTimeout(() => resolve(1), 1000));
const promise2 = new Promise(resolve => setTimeout(() => resolve(2), 500));
const promise3 = new Promise(resolve => setTimeout(() => resolve(3), 1500));

Promise.all([promise1, promise2, promise3])
  .then(results => {
    console.log(results); // 输出 [1, 2, 3]
  });
```

在这个示例中，尽管 promise2 比 promise1 先解决，但是 Promise.all() 返回的结果数组 [1, 2, 3] 顺序仍然是传入的 Promise 顺序。

总结来说，**Promise.all() 保证了返回的数组顺序与传入的 Promise 顺序相同，但并不保证传入的 Promise 的执行顺序**。
