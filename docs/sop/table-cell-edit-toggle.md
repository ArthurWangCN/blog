---
tag: element-plus
---

# element-plus table组件，实现双击可编辑单元格，失去焦点，提交数据，支持下拉选择框选择

vue3+element-plus在使用<el-table>组件时，实现双击可编辑单元格，单元格使用input，失去焦点后，提交数据。支持下拉选择框选择。

主要内容：

- 实现可编辑单元格、支持输入框input、支持下拉选择框；
- 实现失去焦点/回车，或者改变select时，保存数据。

`<template>`代码：

```html
<template>
	<el-table :data="state.tableList">
      <el-table-column
        label="说明"
        align="center"
        prop="describe"
      >
        <template #default="scope">
          <!-- 判断为编辑状态 -->
          <el-input
            v-if="
              state.tableRowEditIndex === scope.$index &&
              state.tableColumnEditIndex == scope.column.id
            "
            ref="tableRowInputRef"
            v-model="scope.row.describe"
            @keyup.enter="
              $event => {
                $event.target.blur()
              }
            "
            @blur="onInputTableBlur(scope)"
          />
          <!-- 判断为显示状态 -->
          <p v-else class="eidt-row-p" @dblclick="dbClickCell(scope)">
            {{ scope.row.describe }}
          </p>
        </template>
      </el-table-column>
      <el-table-column label="复核状态" prop="checkStateAr">
        <template #default="scope">
          <el-select
            v-model="scope.row.checkStateAr"
            @change="onInputTableBlur(scope)"
            placeholder="复核状态"
          >
            <el-option
              v-for="item in reviewStatus"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            />
          </el-select>
        </template>
      </el-table-column>
	</el-table>
</template>
```

通过state.tableRowEditIndex和state.tableColumnEditIndex 来判断是否存在编辑的单元格。

通过双击 @dblclick="dbClickCell(scope)"显示状态的单元格，变为编辑状态的单元格。

input 通过失去焦点事件 @blur="onInputTableBlur(scope)"，和回车事件@keyup.enter来提交修改。

select 通过@change="onInputTableBlur(scope)"来提交修改。


`<script>` 逻辑代码：

```html
<script lang="ts" setup>
import { reactive, ref } from 'vue'
const tableRowInputRef: any = ref(null)
const reviewStatus = [
  {
    value: 0,
    label: '未开始'
  },
  {
    value: 1,
    label: '正确'
  },
  {
    value: 2,
    label: '错误'
  },
] // 复核状态
const state = reactive({
  tableList: [], // 表单数据
  // 编辑的表格行
  tableRowEditIndex: undefined,
  // 编辑的表格列
  tableColumnEditIndex: undefined
})
// 双击可编辑的单元格
const dbClickCell = (scope: any) => {
  console.log(scope)
  // 找到单个格子独有的属性 - 这里是用所在行跟列id区别显示
  state.tableRowEditIndex = scope.$index
  state.tableColumnEditIndex = scope.column.id
  nextTick(() => { 
  	// 获取焦点
    tableRowInputRef.value.focus()
  })
}
// 表格中input取消焦点,select变化
const onInputTableBlur = (scope: any) => {
  console.log('取消焦点', scope)
  state.tableRowEditIndex = undefined
  state.tableColumnEditIndex = undefined
  const id = scope.row.id
  const key = scope.column.property
  const value = scope.row[key]
  // 更新给后端的数据接口
  // 提交数据........
}
</script>
<style lang="scss" scoped>
.eidt-row-p {
  width: 100%;
  height: 32px;
  line-height: 32px;
  text-align: center;
  margin: 0;
}
</style>
```
