---
tag: element-plus
---

# element-plus 单元格合并（满足相互依赖关系）

该逻辑涉及各列之间的相互依赖关系，例如：第2列的合并，是在第1列满足的基础上，在判断第二列是否满足合并条件，依次类推

```vue
<template>
    <div class="table">
        <el-table :data="tableData" :span-method="arraySpanMethod" border style="width: 100%; margin-top: 20px">
            <el-table-column prop="id" label="ID" width="180"> </el-table-column>
            <el-table-column prop="name" label="姓名"> </el-table-column>
            <el-table-column prop="amount1" label="数值 1（元）"> </el-table-column>
            <el-table-column prop="amount2" label="数值 2（元）"> </el-table-column>
            <el-table-column prop="amount3" label="数值 3（元）"> </el-table-column>
        </el-table>
    </div>
</template>

<script>
export default {
    data() {
        return {
            // 模拟后台返回数据
            tableData: [
                {
                    id: "12987122",
                    name: "王大虎",
                    amount1: "234",
                    amount2: "3.2",
                    amount3: 10,
                },
                {
                    id: "12987122",
                    name: "王大虎",
                    amount1: "165",
                    amount2: "4.43",
                    amount3: 12,
                },
                {
                    id: "12987122",
                    name: "王二虎",
                    amount1: "324",
                    amount2: "1.9",
                    amount3: 9,
                },
                {
                    id: "12987123",
                    name: "王二虎",
                    amount1: "621",
                    amount2: "2.2",
                    amount3: 17,
                },
                {
                    id: "12987123",
                    name: "王小虎",
                    amount1: "539",
                    amount2: "4.1",
                    amount3: 15,
                },
                {
                    id: "12987123",
                    name: "王小虎",
                    amount1: "539",
                    amount2: "4.1",
                    amount3: 15,
                },
            ],
            // 需要合并项的列
            needMergeArr: [
                {
                    colName: "id",
                    mergeCheckNames: ["id"],
                },
                {
                    colName: "name",
                    mergeCheckNames: ["id", "name"],
                },
            ],
            rowMergeArrs: {}, // 包含需要一个或多个合并项信息的对象
        };
    },
    methods: {
        arraySpanMethod({ row, column, rowIndex, columnIndex }) {
            let needMerge = this.needMergeArr.some((item) => {
                return item.colName === column.property;
            });
            if (needMerge === true) {
                return this.mergeAction(column.property, rowIndex, column);
            }
        },
        mergeAction(val, rowIndex, colData) {
            let _row = this.rowMergeArrs[val].rowArr[rowIndex];
            let _col = _row > 0 ? 1 : 0;
            return [_row, _col];
        },
        rowMergeHandle(arr, data) {
            if (!Array.isArray(arr) && !arr.length) return false;
            if (!Array.isArray(data) && !data.length) return false;
            let needMerge = {};

            arr.forEach((mergeItem) => {
                // 创建合并管理对象
                needMerge[mergeItem.colName] = {
                    rowArr: [],
                    rowMergeNum: 0,
                };
                let currentMergeItemData = needMerge[mergeItem.colName];

                // 进行合并管理对象数据的遍历整理
                data.forEach((item, index) => {
                    if (index === 0) {
                        currentMergeItemData.rowArr.push(1);
                        currentMergeItemData.rowMergeNum = 0;
                    } else {
                        let currentRowData = data[index];
                        let preRowData = data[index - 1];

                        if (this.colMergeCheck(currentRowData, preRowData, mergeItem.mergeCheckNames)) {
                            currentMergeItemData.rowArr[currentMergeItemData.rowMergeNum] += 1;
                            currentMergeItemData.rowArr.push(0);
                        } else {
                            currentMergeItemData.rowArr.push(1);
                            currentMergeItemData.rowMergeNum = index;
                        }
                    }
                });
            });
            return needMerge;
        },
        colMergeCheck(currentRowData, preRowData, mergeCheckNames) {
            if (!Array.isArray(mergeCheckNames) && !mergeCheckNames.length) return false;
            let result = true;
            for (let index = 0; index < mergeCheckNames.length; index++) {
                const mergeCheckName = mergeCheckNames[index];
                if (currentRowData[mergeCheckName] !== preRowData[mergeCheckName]) {
                    result = false;
                    break;
                }
            }
            return result;
        },
    },
    mounted() {
        // 处理数据,如有分页需要，此行放在加载数据成功之后
        this.rowMergeArrs = this.rowMergeHandle(this.needMergeArr, this.tableData);
    },
};
</script>
```
