---
tag: vue
---

# Vue 关于props的type不能为null的问题总结

在写组建的时候想着让一个组件的value值限制只能是String类型或者null，于是代码逻辑如下：


```js
props: {  
  value: {  
    type: [String, Null],  // 允许 String 或 null  
    required: true  
  }  
}
```

然而这样直接导致报错：

```sh
Error in nextTick: "TypeError: Right-hand side of 'instanceof' is not an object"
```

Vue 不接受 null 作为 prop 的 type 的具体原因主要与 Vue 的类型校验机制有关。在 Vue 中，props 是用来接收来自父组件的数据的，而 type 属性则用来指定这些数据的类型。Vue 的类型校验系统是基于 JavaScript 的原生构造函数来进行的。根据Vue2的props验证规则，prop的类型可以是 String、Number、Boolean、Array、Object、Date、Function、Symbol这些原生构造函数的中任意一个或多个，此外也能自己定义一个构造函数，并通过**instanceof**进行校验（详情参见官网）。

也就是说，当你为一个 prop 指定了一个 type（如 String、Number、Object 等），Vue 会使用 instanceof 操作符来检查传入的值是否是该类型的实例。例如，如果 type 是 String，Vue 会检查传入的值是否是 String 类型的实例。

然而，null 在 JavaScript 中是一个特殊的值，它表示一个空值或不存在的值。null 不是任何构造函数的实例，因此当你尝试将 type 设置为包含 null 的数组时（如 [String, null]），Vue 的类型校验机制会遇到问题。因为 instanceof 操作符的右侧必须是一个构造函数，而 null 不是构造函数，所以这会引发一个 TypeError（如 “Right-hand side of ‘instanceof’ is not an object”）。

解决方案
为了解决这个问题，Vue 提供了 validator 函数作为自定义校验逻辑的一种方式。通过 validator 函数可以定义更复杂的校验规则，包括允许 null 作为有效的 prop 值。这样就可以绕过 Vue 的内置类型校验机制，实现更灵活的数据校验。

```js
props: {  
  value: {  
    // 不设置 type 为 [String, null]，因为这会引发错误  
    required: true,  
    validator: function (value) {  
      // 自定义验证逻辑，允许 value 是 String 或 null  
      return value === null || typeof value === 'string';  
    }  
  }  
},  
```

