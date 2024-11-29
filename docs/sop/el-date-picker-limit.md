---
tag: element-plus
---

# element-plus组件el-date-picker限制选择时间，精确到小时

需求：在日期时间选择控件上，用户只能选择当前时间之前的时刻，需要精确到小时。

实现：

1.实现对当前日期的限制：disabled-date

disabled-date是一个用来判断该日期是否被禁用的函数，接受一个 Date 对象作为参数，结果返回一个 Boolean 值

```js
const disabledDateFunc= (time) => {
    return time.getTime() > Date.now()
}
```

2.实现对当前时间小时的限制：disabled-hours

disabled-hours是一个限制时间中小时数的选择，是一个数组，eg：[12,13,14,15]

```js
const disabledHoursFn = (time) => {
    let disabledHours = []
    // 获取当前时间
    let data = new Date()
    // 如果选择的时间为当前时间的日期，则需要限制小时数，否则不需要限制小时数
    let flag = time.getFullYear() == data.getFullYear() && time.getMonth() == data.getMonth() && time.getDay() == data.getDay()
    if (flag) {
        disabledHours  = range(data.getHours(), 24)
    }
    return disabledHours 
}
```

disabledHoursFn函数可以在calendar-change事件触发时调用。

vue3示例：

```html
<el-date-picker
  v-model="value"
  type="datetime"
  :disabled-date="disabledDateFunc"
  :disabled-hours="disabledHours"
  placeholder="Select date and time"
/>
```

需要特别说明的是，旧版本的picker-options属性已经不适用当前版本时间组件；此外，如果限制时间需要精确到分秒的话，需要对所选时间的小时数进行判断，分别使用disabled-minutes、disabled-seconds属性进行限制。
