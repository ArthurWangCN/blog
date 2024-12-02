---
title: Pinia 快速入门
date: 2023-03-06 22:34:53
tags: Vue.js
categories: JavaScript
description: Pinia（发音为/piːnjʌ/，如英语中的“peenya”）是最接近 piña（西班牙语中的菠萝）的词。 Pinia本质上依然是一个全局状态管理的库，用于跨组件、页面进行状态共享（这点和Vuex、Redux、Mobx一样）。
---

# Pinia 快速入门

## 概念

Pinia（发音为/piːnjʌ/，如英语中的“peenya”）是最接近 piña（西班牙语中的菠萝）的词。 Pinia本质上依然是一个全局状态管理的库，用于跨组件、页面进行状态共享（这点和Vuex、Redux、Mobx一样）。

**什么是store？**

一个 Store （如 Pinia）是一个实体，它持有未绑定到您的组件树的状态和业务逻辑。换句话说，它托管全局状态。它有点像一个始终存在并且每个人都可以读取和写入的组件。它有三个概念，state、getters 和 actions 并且可以安全地假设这些概念等同于组件中的“数据”、“计算”和“方法”。

**我什么时候应该使用store？**

存储应该包含可以在整个应用程序中访问的数据。这包括在许多地方使用的数据，例如导航栏中显示的用户信息，以及需要通过页面保留的数据，例如一个非常复杂的多步骤表格。

另一方面，您应该避免在存储中包含可以托管在组件中的本地数据，例如页面本地元素的可见性。

并非所有应用程序都需要访问全局状态，但如果您需要一个，Pania 将使您的生活更轻松。

## 与 Vuex 3.x/4.x 的比较
> Vuex 3.x 是 Vuex 的 Vue 2 而 Vuex 4.x 是 Vue 3

Pinia API 与 Vuex ≤4 有很大不同，即：

- mutations 不再存在。他们经常被认为是 非常 冗长。他们最初带来了 devtools 集成，但这不再是问题。
- 无需创建自定义复杂包装器来支持 TypeScript，所有内容都是类型化的，并且 API 的设计方式尽可能利用 TS 类型推断。
- 不再需要注入、导入函数、调用函数、享受自动完成功能！
- 无需动态添加 Store，默认情况下它们都是动态的，您甚至都不会注意到。请注意，您仍然可以随时手动使用 Store 进行注册，但因为它是自动的，您无需担心。
- 不再有 modules 的嵌套结构。您仍然可以通过在另一个 Store 中导入和 使用 来隐式嵌套 Store，但 Pinia 通过设计提供平面结构，同时仍然支持 Store 之间的交叉组合方式。 您甚至可以拥有 Store 的循环依赖关系。
- 没有 命名空间模块。鉴于 Store 的扁平架构，“命名空间” Store 是其定义方式所固有的，您可以说所有 Store 都是命名空间的。


## 开始使用

用你最喜欢的包管理器安装 pinia：

```sh
yarn add pinia
# 或者使用 npm
npm install pinia
```

如果您的应用使用 Vue 2，您还需要安装组合 API：@vue/composition-api。 本文将基于 vue3 示范，vue2 具体用法参考 [pinia官网](https://pinia.web3doc.top/getting-started.html#%E5%AE%89%E8%A3%85)。

创建一个 pinia（根存储）并将其传递给应用程序：

```js
import { createPinia } from 'pinia'
app.use(createPinia())
```


## 核心概念

### 定义一个store

在深入了解核心概念之前，我们需要知道 Store 是使用 defineStore() 定义的，并且它需要一个唯一名称，作为第一个参数传递：

```js
import { defineStore } from 'pinia'

// useStore 可以是 useUser、useCart 之类的任何东西
// 第一个参数是应用程序中 store 的唯一 id
export const useStore = defineStore('main', {
  // other options...
})
```

这个 name，也称为 id，是必要的，Pinia 使用它来将 store 连接到 devtools。 将返回的函数命名为 use... 是跨可组合项的约定，以使其符合你的使用习惯。

### 使用store

我们正在 定义 一个 store，因为在 setup() 中调用 useStore() 之前不会创建 store：

```js
import { useStore } from '@/stores/counter'

export default {
  setup() {
    const store = useStore()

    return {
      // 您可以返回整个 store 实例以在模板中使用它
      store,
    }
  },
}
```

您可以根据需要定义任意数量的 store ，并且**您应该在不同的文件中定义每个 store **以充分利用 pinia（例如自动允许您的包进行代码拆分和 TypeScript 推理）。

一旦 store 被实例化，你就可以直接在 store 上访问 state、getters 和 actions 中定义的任何属性。 我们将在接下来的页面中详细介绍这些内容，但自动补全会对您有所帮助。

请注意，store 是一个用reactive 包裹的对象，这意味着不需要在getter 之后写.value，但是，就像setup 中的props 一样，我们不能对其进行解构：

```js
export default defineComponent({
  setup() {
    const store = useStore()
    // ❌ 这不起作用，因为它会破坏响应式
    // 这和从 props 解构是一样的
    const { name, doubleCount } = store

    name // "eduardo"
    doubleCount // 2

    return {
      // 一直会是 "eduardo"
      name,
      // 一直会是 2
      doubleCount,
      // 这将是响应式的
      doubleValue: computed(() => store.doubleCount),
      }
  },
})
```

为了从 Store 中提取属性同时保持其响应式，您需要使用 `storeToRefs()`。 它将为任何响应式属性创建 refs。 当您仅使用 store 中的状态但不调用任何操作时，这很有用：

```js
import { storeToRefs } from 'pinia'

export default defineComponent({
  setup() {
    const store = useStore()
    // `name` 和 `doubleCount` 是响应式引用
    // 这也会为插件添加的属性创建引用
    // 但跳过任何 action 或 非响应式（不是 ref/reactive）的属性
    const { name, doubleCount } = storeToRefs(store)

    return {
      name,
      doubleCount
    }
  },
})
```

### state

大多数时候，state 是 store 的核心部分。 我们通常从定义应用程序的状态开始。 在 Pinia 中，状态被定义为返回初始状态的函数。 Pinia 在服务器端和客户端都可以工作。

```js
import { defineStore } from 'pinia'

const useStore = defineStore('storeId', {
  // 推荐使用 完整类型推断的箭头函数
  state: () => {
    return {
      // 所有这些属性都将自动推断其类型
      counter: 0,
      name: 'Eduardo',
      isAdmin: true,
    }
  },
})
```

#### 访问state

默认情况下，您可以通过 store 实例访问状态来直接读取和写入状态：

```js
const store = useStore()
store.counter++
```

#### 重置状态

您可以通过调用 store 上的 `$reset()` 方法将状态 重置 到其初始值：

```js
const store = useStore()
store.$reset()
```

#### 改变状态

除了直接用 store.counter++ 修改 store，你还可以调用 $patch 方法。 它允许您使用部分“state”对象同时应用多个更改：

```js
store.$patch({
  counter: store.counter + 1,
  name: 'Abalam',
})
```

但是，使用这种语法应用某些突变非常困难或代价高昂：任何集合修改（例如，从数组中推送、删除、拼接元素）都需要您创建一个新集合。 正因为如此，$patch 方法也接受一个函数来批量修改集合内部分对象的情况：

```js
cartStore.$patch((state) => {
  state.items.push({ name: 'shoes', quantity: 1 })
  state.hasChanged = true
})
```

这里的主要区别是$patch() 允许您将批量更改的日志写入开发工具中的一个条目中。 注意两者，state 和 $patch() 的直接更改都出现在 devtools 中，并且可以进行 time travelled（在 Vue 3 中还没有）。

#### 替换state

您可以通过将其 $state 属性设置为新对象来替换 Store 的整个状态：

```js
store.$state = { counter: 666, name: 'Paimon' }
```

您还可以通过更改 pinia 实例的 state 来替换应用程序的整个状态。 这在 SSR for hydration 期间使用。

```js
pinia.state.value = {}
```

#### 订阅状态

可以通过 store 的 $subscribe() 方法查看状态及其变化，类似于 Vuex 的 subscribe 方法。 与常规的 watch() 相比，使用 $subscribe() 的优点是 subscriptions 只会在 patches 之后触发一次（例如，当使用上面的函数版本时）。

```js
cartStore.$subscribe((mutation, state) => {
  // import { MutationType } from 'pinia'
  mutation.type // 'direct' | 'patch object' | 'patch function'
  // 与 cartStore.$id 相同
  mutation.storeId // 'cart'
  // 仅适用于 mutation.type === 'patch object'
  mutation.payload // 补丁对象传递给 to cartStore.$patch()

  // 每当它发生变化时，将整个状态持久化到本地存储
  localStorage.setItem('cart', JSON.stringify(state))
})
```

默认情况下，state subscriptions 绑定到添加它们的组件（如果 store 位于组件的 setup() 中）。 意思是，当组件被卸载时，它们将被自动删除。 如果要在卸载组件后保留它们，请将 { detached: true } 作为第二个参数传递给 detach 当前组件的 state subscription：

```js
export default {
  setup() {
    const someStore = useSomeStore()

    // 此订阅将在组件卸载后保留
    someStore.$subscribe(callback, { detached: true })

    // ...
  },
}
```

您可以在 pinia 实例上查看整个状态：

```js
watch(
  pinia.state,
  (state) => {
    // 每当它发生变化时，将整个状态持久化到本地存储
    localStorage.setItem('piniaState', JSON.stringify(state))
  },
  { deep: true }
)
```

### Getters

Getter 完全等同于 Store 状态的 计算值。 它们可以用 defineStore() 中的 getters 属性定义。 他们接收“状态”作为第一个参数以鼓励箭头函数的使用：

```js
export const useStore = defineStore('main', {
  state: () => ({
    counter: 0,
  }),
  getters: {
    doubleCount: (state) => state.counter * 2,
  },
})
```

大多数时候，getter 只会依赖状态，但是，他们可能需要使用其他 getter。 正因为如此，我们可以在定义常规函数时通过 this 访问到 整个 store 的实例， 但是需要定义返回类型（在 TypeScript 中）。 这是由于 TypeScript 中的一个已知限制，并且不会影响使用箭头函数定义的 getter，也不会影响不使用 this 的 getter：

```js
export const useStore = defineStore('main', {
  state: () => ({
    counter: 0,
  }),
  getters: {
    // 自动将返回类型推断为数字
    doubleCount(state) {
      return state.counter * 2
    },
    // 返回类型必须明确设置
    doublePlusOne(): number {
      return this.counter * 2 + 1
    },
  },
})
```

然后你可以直接在 store 实例上访问 getter。

#### 访问其他 getter

与计算属性一样，您可以组合多个 getter。 通过 this 访问任何其他 getter。 即使您不使用 TypeScript，您也可以使用 JSDoc 提示您的 IDE 类型：

```js
export const useStore = defineStore('main', {
  state: () => ({
    counter: 0,
  }),
  getters: {
    // 类型是自动推断的，因为我们没有使用 `this`
    doubleCount: (state) => state.counter * 2,
    // 这里需要我们自己添加类型（在 JS 中使用 JSDoc）。 我们还可以
    // 使用它来记录 getter
    /**
     * 返回计数器值乘以二加一。
     *
     * @returns {number}
     */
    doubleCountPlusOne() {
      // 自动完成 ✨
      return this.doubleCount + 1
    },
  },
})
```

#### 将参数传递给 getter

Getters 只是幕后的 computed 属性，因此无法向它们传递任何参数。 但是，您可以从 getter 返回一个函数以接受任何参数：

```js
export const useStore = defineStore('main', {
  getters: {
    getUserById: (state) => {
      return (userId) => state.users.find((user) => user.id === userId)
    },
  },
})
```

并在组件中使用：

```html
<script>
export default {
  setup() {
    const store = useStore()

    return { getUserById: store.getUserById }
  },
}
</script>

<template>
  <p>User 2: {{ getUserById(2) }}</p>
</template>
```

请注意，在执行此操作时，getter 不再缓存，它们只是您调用的函数。 但是，您可以在 getter 本身内部缓存一些结果，这并不常见，但应该证明性能更高：

```js
export const useStore = defineStore('main', {
  getters: {
    getActiveUserById(state) {
      const activeUsers = state.users.filter((user) => user.active)
      return (userId) => activeUsers.find((user) => user.id === userId)
    },
  },
})
```

#### 访问其他 Store 的getter

要使用其他存储 getter，您可以直接在 better 内部使用它：

```js
import { useOtherStore } from './other-store'

export const useStore = defineStore('main', {
  state: () => ({
    // ...
  }),
  getters: {
    otherGetter(state) {
      const otherStore = useOtherStore()
      return state.localData + otherStore.data
    },
  },
})
```

### Actions

Actions 相当于组件中的 methods。 它们可以使用 defineStore() 中的 actions 属性定义，并且它们非常适合定义业务逻辑：

```js
export const useStore = defineStore('main', {
  state: () => ({
    counter: 0,
  }),
  actions: {
    increment() {
      this.counter++
    },
    randomizeCounter() {
      this.counter = Math.round(100 * Math.random())
    },
  },
})
```

与 getters 一样，操作可以通过 this 访问 whole store instance 并提供完整类型（和自动完成✨）支持。 与它们不同，actions 可以是异步的，您可以在其中await 任何 API 调用甚至其他操作！ 这是使用 Mande 的示例。 请注意，只要您获得“Promise”，您使用的库并不重要，您甚至可以使用浏览器的“fetch”函数：

```js
import { mande } from 'mande'

const api = mande('/api/users')

export const useUsers = defineStore('users', {
  state: () => ({
    userData: null,
    // ...
  }),

  actions: {
    async registerUser(login, password) {
      try {
        this.userData = await api.post({ login, password })
        showTooltip(`Welcome back ${this.userData.name}!`)
      } catch (error) {
        showTooltip(error)
        // 让表单组件显示错误
        return error
      }
    },
  },
})
```

你也可以完全自由地设置你想要的任何参数并返回任何东西。 调用 Action 时，一切都会自动推断！

Actions 像 methods 一样被调用：

```js
export default defineComponent({
  setup() {
    const main = useMainStore()
    // Actions 像 methods 一样被调用：
    main.randomizeCounter()

    return {}
  },
})
```

#### 访问其他 store 操作

要使用另一个 store ，您可以直接在操作内部使用它：

```js
import { useAuthStore } from './auth-store'

export const useSettingsStore = defineStore('settings', {
  state: () => ({
    // ...
  }),
  actions: {
    async fetchUserPreferences(preferences) {
      const auth = useAuthStore()
      if (auth.isAuthenticated) {
        this.preferences = await fetchPreferences()
      } else {
        throw new Error('User must be authenticated')
      }
    },
  },
})
```

#### 订阅 Actions

可以使用 store.$onAction() 订阅 action 及其结果。 传递给它的回调在 action 之前执行。 after 处理 Promise 并允许您在 action 完成后执行函数。 以类似的方式，onError 允许您在处理中抛出错误。 这些对于在运行时跟踪错误很有用，类似于 Vue 文档中的这个提示。

这是一个在运行 action 之前和它们 resolve/reject 之后记录的示例。

```js
const unsubscribe = someStore.$onAction(
  ({
    name, // action 的名字
    store, // store 实例
    args, // 调用这个 action 的参数
    after, // 在这个 action 执行完毕之后，执行这个函数
    onError, // 在这个 action 抛出异常的时候，执行这个函数
  }) => {
    // 记录开始的时间变量
    const startTime = Date.now()
    // 这将在 `store` 上的操作执行之前触发
    console.log(`Start "${name}" with params [${args.join(', ')}].`)

    // 如果 action 成功并且完全运行后，after 将触发。
    // 它将等待任何返回的 promise
    after((result) => {
      console.log(
        `Finished "${name}" after ${
          Date.now() - startTime
        }ms.\nResult: ${result}.`
      )
    })

    // 如果 action 抛出或返回 Promise.reject ，onError 将触发
    onError((error) => {
      console.warn(
        `Failed "${name}" after ${Date.now() - startTime}ms.\nError: ${error}.`
      )
    })
  }
)

// 手动移除订阅
unsubscribe()
```

默认情况下，action subscriptions 绑定到添加它们的组件（如果 store 位于组件的 setup() 内）。 意思是，当组件被卸载时，它们将被自动删除。 如果要在卸载组件后保留它们，请将 true 作为第二个参数传递给当前组件的 detach action subscription：

```js
export default {
  setup() {
    const someStore = useSomeStore()

    // 此订阅将在组件卸载后保留
    someStore.$onAction(callback, true)

    // ...
  },
}
```

## 最后

Pinia 相较于 Vuex 来说，更简单的API，更少的写法，并且对于TypeScript 支持非常好，可以自动推断类型，可以随意调用，不用局限于 mutations，大大提高了开发效率，并且降低了学习成本。