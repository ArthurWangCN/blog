---
tag: plugins
---

# json-bigint 解决 websocket 接收消息体转换后精度丢失问题

websocket 接收的消息是对象字符串，前端 JSON.parse 之后 id 和数据库里的不对，排查后发现是精度丢失，超出了安全范围之后的数会被替换为0，浏览器 Network 显示的也是精度丢失后的结果：

```js
{"id":1813024828676980738,"name":"xxx"}
```

**前端解决方案：**

解决方法很简单，只需要引入 json-bigint 库进行 parse 即可。

```shell
npm install json-bigint
```

```js
const jsonStr = '{"id":1813024828676980738,"name":"xxx"}'

// JSONBig 可以处理数据中超出 JavaScript 安全整数范围的问题
console.log(JSONBig.parse(jsonStr)) // 把 JSON 格式的字符串转为 JavaScript 对象

// 使用的时候需要把 BigNumber 类型的数据转为字符串来使用
console.log(JSONBig.parse(jsonStr).id.toString()) // 1813024828676980738
```

json-bigint 会把超出 JS 安全整数范围的数字转为一个 BigNumber 类型的对象，对象数据是它内部的一个算法处理之后的，我们要做的就是在使用的时候转为字符串来使用。

还有一个简单的方法，让后端把数字类型改成字符串类型！
