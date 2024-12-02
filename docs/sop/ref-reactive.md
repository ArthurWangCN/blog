---
tag: vue
sidebar: false
---

# 源码视角，Vue3为什么推荐使用ref而不是reactive

ref 和 reactive 是 Vue3 中实现响应式数据的核心 API。ref 用于包装基本数据类型，而 reactive 用于处理对象和数组。尽管 reactive 似乎更适合处理对象，但 Vue3 官方文档更推荐使用 ref。

我的想法，ref就是比reactive好用，官方也是这么说的，不服来踩！下面我们从源码的角度详细讨论这两个 API，以及 Vue3 为什么推荐使用ref而不是reactive？

## ref 的内部工作原理
ref 是一个函数，它接受一个内部值并返回一个响应式且可变的引用对象。这个引用对象有一个 .value 属性，该属性指向内部值。 

```js
// 深响应式
export function ref(value?: unknown) {
  return createRef(value, false)
}

// 浅响应式
export function shallowRef(value?: unknown) {
  return createRef(value, true)
}

function createRef(rawValue: unknown, shallow: boolean) {
  // 如果传入的值已经是一个 ref，则直接返回它
  if (isRef(rawValue)) {
    return rawValue
  }
  // 否则，创建一个新的 RefImpl 实例
  return new RefImpl(rawValue, shallow)
}

class RefImpl<T> {
  // 存储响应式的值。我们追踪和更新的就是_value。（这个是重点）
  private _value: T
  // 用于存储原始值，即未经任何响应式处理的值。（用于对比的，这块的内容可以不看）
  private _rawValue: T 

  // 用于依赖跟踪的 Dep 类实例
  public dep?: Dep = undefined
  // 一个标记，表示这是一个 ref 实例
  public readonly __v_isRef = true

  constructor(
    value: T,
    public readonly __v_isShallow: boolean,
  ) {
    // 如果是浅响应式，直接使用原始值，否则转换为非响应式原始值
    this._rawValue = __v_isShallow ? value : toRaw(value)
    // 如果是浅响应式，直接使用原始值，否则转换为响应式值
    this._value = __v_isShallow ? value : toReactive(value)
    
    // toRaw 用于将响应式引用转换回原始值
    // toReactive 函数用于将传入的值转换为响应式对象。对于基本数据类型，toReactive 直接返回原始值。
    // 对于对象和数组，toReactive 内部会调用 reactive 来创建一个响应式代理。
    // 因此，对于 ref 来说，基本数据类型的值会被 RefImpl 直接包装，而对象和数组
    // 会被 reactive 转换为响应式代理，最后也会被 RefImpl 包装。
    // 这样，无论是哪种类型的数据，ref 都可以提供响应式的 value 属性，
    // 使得数据变化可以被 Vue 正确追踪和更新。
    // export const toReactive = (value) => isObject(value) ? reactive(value) : value
  }

  get value() {
    // 追踪依赖，这样当 ref 的值发生变化时，依赖这个 ref 的组件或副作用函数可以重新运行。
    trackRefValue(this)
    // 返回存储的响应式值
    return this._value
  }

  set value(newVal) {
    // 判断是否应该使用新值的直接形式（浅响应式或只读）
    const useDirectValue =
      this.__v_isShallow || isShallow(newVal) || isReadonly(newVal)
    // 如果需要，将新值转换为非响应式原始值
    newVal = useDirectValue ? newVal : toRaw(newVal)
    // 如果新值与旧值不同，更新 _rawValue 和 _value
    if (hasChanged(newVal, this._rawValue)) {
      this._rawValue = newVal
      this._value = useDirectValue ? newVal : toReactive(newVal)
      // 触发依赖更新
      triggerRefValue(this, DirtyLevels.Dirty, newVal)
    }
  }
}
```

在上述代码中，ref 函数通过 new RefImpl(value) 创建了一个新的 RefImpl 实例。这个实例包含 getter 和 setter，分别用于追踪依赖和触发更新。使用 ref 可以声明任何数据类型的响应式状态，包括对象和数组。

```js
import { ref } from 'vue' 

let state = ref({ count: 0 })
state.value.count++
```

注意，ref核心是返回响应式且可变的引用对象，而reactive核心是返回的是响应式代理，这是两者本质上的核心区别，也就导致了ref优于reactive，我们接着看下reactive源码实现。

## reactive 的内部工作原理

reactive 是一个函数，它接受一个对象并返回该对象的响应式代理，也就是 Proxy。

```js
function reactive(target) {
  if (target && target.__v_isReactive) {
    return target
  }

  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers,
    reactiveMap
  )
}

function createReactiveObject(
  target,
  isReadonly,
  baseHandlers,
  collectionHandlers,
  proxyMap
) {
  if (!isObject(target)) {
    return target
  }
  
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }
  
  const proxy = new Proxy(target, baseHandlers)
  proxyMap.set(target, proxy)
  return proxy
}
```

reactive的源码相对就简单多了，reactive 通过 new Proxy(target, baseHandlers) 创建了一个代理。这个代理会拦截对目标对象的操作，从而实现响应式。

```js
import { reactive } from 'vue' 

let state = reactive({ count: 0 })
state.count++
```

到这里我们可以看出 ref 和 reactive 在声明数据的响应式状态上，底层原理是不一样的。ref 采用 RefImpl对象实例，reactive采用Proxy代理对象。

## ref 更深入的理解

当你使用 new RefImpl(value) 创建一个 RefImpl 实例时，这个实例大致上会包含以下几部分：

1. 内部值：实例存储了传递给构造函数的初始值。
2. 依赖收集：实例需要跟踪所有依赖于它的效果（effect），例如计算属性或者副作用函数。这通常通过一个依赖列表或者集合来实现。
3. 触发更新：当实例的值发生变化时，它需要通知所有依赖于它的效果，以便它们可以重新计算或执行。

RefImpl 类似于发布-订阅模式的设计，以下是一个简化的 RefImpl 类的伪代码实现，展示这个实现过程：

```js
class Dep {
  constructor() {
    this.subscribers = new Set();
  }

  depend() {
    if (activeEffect) {
      this.subscribers.add(activeEffect);
    }
  }

  notify() {
    this.subscribers.forEach(effect => effect());
  }
}

let activeEffect = null;

function watchEffect(effect) {
  activeEffect = effect;
  effect();
  activeEffect = null;
}

class RefImpl {
  constructor(value) {
    this._value = value;
    this.dep = new Dep();
  }

  get value() {
    // 当获取值时，进行依赖收集
    this.dep.depend();
    return this._value;
  }

  set value(newValue) {
    if (newValue !== this._value) {
      this._value = newValue;
      // 值改变时，触发更新
      this.dep.notify();
    }
  }
}

// 使用示例
let count = new RefImpl(0);

watchEffect(() => {
  console.log(`The count is: ${count.value}`); // 订阅变化
});

count.value++; // 修改值，触发通知，重新执行watchEffect中的函数
```

Dep 类负责管理一个依赖列表，并提供依赖收集和通知更新的功能。RefImpl 类包含一个内部值 _value 和一个 Dep 实例。当 value 被访问时，通过 get 方法进行依赖收集；当 value 被赋予新值时，通过 set 方法触发更新。

ref 和 reactive 尽管两者在内部实现上有所不同，但它们都能满足我们对于声明响应式变量的要求，但是 reactive 却存在一定的局限性。

## reactive 的局限性

在 Vue3 中，reactive API 通过 Proxy 实现了一种响应式数据的方法，尽管这种方法在性能上比 Vue2 有所提升，但 Proxy 的局限性也导致了 reactive 的局限性，这些局限性可能会影响开发者的使用体验。

**1、仅对引用数据类型有效**

reactive 主要适用于对象，包括数组和一些集合类型（如 Map 和 Set）。对于基础数据类型（如 string、number 和 boolean），reactive 是无效的。这意味着如果你尝试使用 reactive 来处理这些基础数据类型，将会得到一个非响应式的对象。

```js
import { reactive } from 'vue';
const state = reactive({ count: 0 });
```

**2、使用不当会失去响应**

- 直接赋值对象：如果直接将一个响应式对象赋值给另一个变量，将会失去响应性。这是因为 reactive 返回的是对象本身，而不仅仅是代理。

```js
import { reactive } from 'vue';

let state = reactive({ count: 0 });
state = { count: 1 }; // 失去响应性
```

- 直接替换响应式对象：同样，直接替换一个响应式对象也会导致失去响应性。

```js
import { reactive } from 'vue';

let state = reactive({ count: 0 });
state = reactive({ count: 1 }); // 失去响应性
```

- 直接解构对象：在解构响应式对象时，如果直接解构对象属性，将会得到一个非响应式的变量。

```js
const state = reactive({ count: 0 });

let { count } = state;
count++; // count 仍然是 0
```

解决这个问题，需要使用 toRefs 函数来将响应式对象转换为 ref 对象。

```js
import { toRefs } from 'vue';

const state = reactive({ count: 0 });
let { count } = toRefs(state);
count++; // count 现在是 1
```

- 将响应式对象的属性赋值给变量：如果将响应式对象的属性赋值给一个变量，这个变量的值将不会是响应式的。

```js
let state = reactive({ count: 0 })

let count = state.count
count++  // count 仍然是 0
console.log(state.count)
```

使用 reactive 声明响应式变量的确存在一些不便之处，尤其是对于喜欢使用解构赋值的开发者而言。这些局限性可能会导致意外的行为，因此在使用 reactive 时需要格外注意。相比之下，ref API 提供了一种更灵活和统一的方式来处理响应式数据。

## 为什么推荐使用 ref ？

ref()它为响应式编程提供了一种统一的解决方案，适用于所有类型的数据，包括基本数据类型和复杂对象。以下是推荐使用 ref 的几个关键原因：

1、统一性

ref 的核心优势之一是它的统一性。它提供了一种简单、一致的方式来处理所有类型的数据，无论是数字、字符串、对象还是数组。这种统一性极大地简化了开发者的代码，减少了在不同数据类型之间切换时的复杂性。

```js
import { ref } from 'vue';

let num = ref(0);
let str = ref('Hello');
let obj = ref({ count: 0 });

// 修改基本数据类型
num.value++;
str.value += ' World';

// 修改对象
obj.value.count++;
```

2、深层响应性

ref 支持深层响应性，这意味着它可以追踪和更新嵌套对象和数组中的变化。这种特性使得 ref 非常适合处理复杂的数据结构，如对象和数组。

```js
import { ref } from 'vue';

let obj = ref({
  user: {
    name: 'xiaoming',
    details: {
      age: 18
    }
  }
});

// 修改嵌套对象
obj.value.user.details.age++;
```

当然，为了减少大型不可变数据的响应式开销，也可以通过使用shallowRef来放弃深层响应性。

3、灵活性

ref 提供了高度的灵活性，尤其在处理普通赋值方面。这种灵活性使得 ref 在开发中的使用更加方便，特别是在进行复杂的数据操作时。

```js
import { ref } from 'vue';

let state = ref({
  count: 0,
  name: 'Vue'
});

// 替换整个对象
state.value = {
  count: 10,
  name: 'Vue 4'
};
// 修改对象内的属性
state.value.count = 20;
state.value.name = 'Vue 5';
// 添加新的属性
state.value.newProperty = 'New Property';
// 删除属性
delete state.value.newProperty;
// 使用解构更新属性（注意要保持响应性）
let { count, name } = state.value;
state.value = { count: count + 1, name };
// 复杂操作，例如根据条件更新属性
if (someCondition) {
  state.value = {
    ...state.value,
    name: 'Updated Name'
  };
}
console.log(state.value)
```

