# 表格系统

## 核心概念

BuildAdmin 的表格系统由三部分组成：
1. **Table 组件** - 基于 el-table 封装
2. **baTable 类** - 表格管家（数据、方法、钩子）
3. **baTableApi 类** - 快速生成 CRUD 接口

## 基本用法

```vue
<script setup lang="ts">
import Table from '/@/components/table/index.vue'
import baTableClass from '/@/utils/baTable'
import { baTableApi } from '/@/api/common'
import { defaultOptButtons } from '/@/components/table'

const baTable = new baTableClass(
    new baTableApi('/admin/user.user/'),
    {
        column: [
            { type: 'selection', align: 'center', operator: false },
            { label: 'ID', prop: 'id', align: 'center', operator: 'LIKE', width: 70 },
            { label: '用户名', prop: 'username', align: 'center', operator: 'LIKE' },
            {
                label: '操作', align: 'center', width: 100,
                render: 'buttons',
                buttons: defaultOptButtons(['edit', 'delete']),
                operator: false,
            },
        ],
    },
)

provide('baTable', baTable)
baTable.mount()
baTable.getData()
</script>

<template>
    <Table />
    <PopupForm />
</template>
```

## 表格列属性

```ts
interface TableColumn {
    show?: boolean                          // 是否显示此列
    render?: TableRenderer                  // 单元格渲染器
    replaceValue?: Record<string, any>      // 值替换（字典）
    slotName?: string                       // render=slot 时的 slot 名
    customRender?: string | Component       // render=customRender 时的组件
    customTemplate?: Function               // render=customTemplate 时的 HTML 渲染
    formatter?: Function                    // 渲染前值预处理
    effect?: TagProps['effect']             // render=tag 时的 effect
    timeFormat?: string                     // render=datetime 时的格式
    buttons?: OptButton[]                   // render=buttons 时的按钮数组
    operator?: boolean | OperatorStr        // 公共搜索操作符
    operatorPlaceholder?: string            // 公共搜索 placeholder
    comSearchRender?: string                // 公共搜索渲染方式
}
```

### 渲染器类型（TableRenderer）

buttons, color, customRender, customTemplate, datetime, icon, image, images, switch, tag, tags, url, slot

### 搜索操作符（OperatorStr）

=, <>, >, >=, <, <=, LIKE, NOT LIKE, IN, NOT IN, RANGE, NOT RANGE, NULL, NOT NULL, FIND_IN_SET

## baTable 类

### 常用属性

```ts
baTable.table.data          // 表格数据
baTable.table.column        // 列定义
baTable.table.filter        // 过滤条件（分页、排序、搜索）
baTable.table.loading       // 加载状态
baTable.table.selection     // 当前选中行
baTable.table.total         // 数据总量
baTable.form.items          // 表单项数据
baTable.form.operate        // 当前操作（Add/Edit）
baTable.form.operateIds     // 被操作数据 ID
baTable.extend              // 扩展数据（任意定义）
```

### 常用方法

```ts
baTable.getData()           // 获取表格数据
baTable.mount()             // 表格初始化
baTable.toggleForm()        // 切换表单显示
baTable.onSubmit(formRef)   // 提交表单
baTable.initSort()          // 初始化排序
baTable.dragSort()          // 初始化拖拽排序
baTable.onTableHeaderAction('refresh', { event: 'custom' })  // 刷新
```

### 钩子（Hooks）

```ts
// 前置钩子（返回 false 可取消操作）
baTable.before.getData = () => {}
baTable.before.postDel = ({ ids }) => {}
baTable.before.getEditData = ({ id }) => {}
baTable.before.onSubmit = ({ formEl, operate, items }) => {}
baTable.before.onTableAction = ({ event, data }) => {}
baTable.before.onTableHeaderAction = ({ event, data }) => {}

// 后置钩子
baTable.after.getData = ({ res }) => {}
baTable.after.postDel = ({ res }) => {}
baTable.after.getEditData = ({ res }) => {}
baTable.after.onSubmit = ({ res }) => {}
```

## baTableApi 类

```ts
import { baTableApi } from '/@/api/common'

const api = new baTableApi('/admin/user.user/')
// 自动生成：index, add, edit, del, sortable 的 URL 和请求方法

// 修改 URL
api.actionUrl.set('index', '/admin/user.user/test')
api.actionUrl.set('add', '/admin/user.user/addNew')
```

## 公共搜索配置

```ts
column: [
    {
        label: '性别', prop: 'gender',
        render: 'tag',                              // 单元格渲染为 tag
        replaceValue: { '0': '未知', '1': '女', '2': '男' },  // 字典
        comSearchRender: 'select',                  // 公共搜索渲染为 select
        operator: '=',                              // 搜索操作符
    },
    {
        label: '创建时间', prop: 'createtime',
        render: 'datetime',
        operator: 'RANGE',                          // 范围搜索
    },
]
```

## 自定义单元格渲染

### 方式一：Slot

```ts
column: [
    { label: '自定义', render: 'slot', slotName: 'mySlot', operator: 'LIKE' },
]
```

```vue
<template>
    <Table>
        <template #mySlot>
            <el-table-column prop="username" label="用户名" />
        </template>
    </Table>
</template>
```

### 方式二：customRender（h 函数）

```ts
const renderId = {
    render(context) {
        return h('h1', context.$attrs.renderValue)
    },
}
column: [
    { label: 'ID', prop: 'id', render: 'customRender', customRender: h(renderId) },
]
```

### 方式三：自定义渲染器组件

在 `/components/table/fieldRender/` 下新建组件，文件名即渲染器名。
