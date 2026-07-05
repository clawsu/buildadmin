# 前端代码规范

## 格式化（Prettier）

- 4 空格缩进，无分号，单引号
- 行宽 150，换行符 `lf`
- 尾随逗号 `es5`
- 括号间距 `true`
- Vue 文件脚本/样式不缩进

## Linter（ESLint）

- ESLint + vue + typescript + prettier 插件
- 宽松模式：大量规则设为 off（详见 `web/eslint.config.js`）
- `@typescript-eslint/no-unused-vars` 仅 warn，忽略 `_` 前缀变量

## Vue 组件规范

### 基础结构

```vue
<template>
    <div class="default-main ba-table-box">
        <TableHeader :buttons="['refresh', 'add', 'edit', 'delete', 'quickSearch', 'columnDisplay']" />
        <Table ref="tableRef" />
        <PopupForm ref="formRef" />
    </div>
</template>
<script setup lang="ts">
import { onMounted, provide, useTemplateRef } from 'vue'
import PopupForm from './popupForm.vue'
import { baTableApi } from '/@/api/common'
import { defaultOptButtons } from '/@/components/table'
import TableHeader from '/@/components/table/header/index.vue'
import Table from '/@/components/table/index.vue'
import baTableClass from '/@/utils/baTable'

defineOptions({ name: '{module}/xxx' })

const formRef = useTemplateRef('formRef')
const tableRef = useTemplateRef('tableRef')

const baTable: baTableClass = new baTableClass(
    new baTableApi('/admin/{module}.Xxx/'),
    { column: [/* 列定义 */] }
)

provide('baTable', baTable)

onMounted(() => {
    baTable.table.ref = tableRef.value
    baTable.mount()
    baTable.getData()
})
</script>
```

### 关键约定

- **必须**使用 `<script setup lang="ts">`
- **必须**添加 `defineOptions({ name: '...' })` 组件名（与菜单 `name` 完全一致）
- 模板引用使用 `useTemplateRef()` 而非 `ref()`
- 父子组件共享 `baTable` 使用 `provide/inject`
- **业务字段标签直接用中文**，禁止用 `t('xxx.title')` 这类未维护的翻译键
- 框架通用键可用 `t('Cancel')`、`t('Save')`、`t('Operate')`、`t('State')` 等
- `baTable.mount()` 和 `baTable.getData()` 应在 `onMounted` 中调用
- API 路径用 `.` 分隔：`/admin/{module}.{Module}/`（非 `/admin/{module}/{module}/`）

### PopupForm 模式

```vue
<template>
    <el-dialog class="ba-operate-dialog" :model-value="['Add','Edit'].includes(baTable.form.operate!)" @close="baTable.toggleForm">
        <el-form ref="formRef" :model="baTable.form.items" :rules="rules" @keyup.enter="baTable.onSubmit(formRef)">
            <el-form-item prop="name" label="名称">
                <el-input v-model="baTable.form.items!.name" />
            </el-form-item>
        </el-form>
        <template #footer>
            <el-button @click="baTable.toggleForm('')">{{ t('Cancel') }}</el-button>
            <el-button v-blur :loading="baTable.form.submitLoading" @click="baTable.onSubmit(formRef)" type="primary">
                {{ t('Save') }}
            </el-button>
        </template>
    </el-dialog>
</template>
<script setup lang="ts">
import { inject } from 'vue'
import type baTableClass from '/@/utils/baTable'
import { buildValidatorData } from '/@/utils/validate'

const formRef = useTemplateRef('formRef')
const baTable = inject('baTable') as baTableClass

const rules = reactive({
    name: [buildValidatorData({ name: 'required', title: '名称' })],
})
</script>
```

## API 模块规范

### 文件结构

```
api/
├── common.ts              # 共享 API 函数 + baTableApi 类
├── backend/
│   ├── index.ts           # 后台初始化、登录、登出
│   ├── auth/group.ts      # 按业务模块拆分
│   ├── dashboard.ts
│   └── ...
└── frontend/
    ├── index.ts           # 前台初始化
    └── user/index.ts
```

### baTableApi 类

`baTableApi` 自动从控制器 URL 生成 CRUD 端点：

| 方法 | HTTP | 说明 |
|------|------|------|
| `index(filter)` | GET | 分页列表 |
| `edit(params)` | GET | 编辑 |
| `del(ids)` | DELETE | 删除 |
| `postData(action, data)` | POST | 添加/自定义操作 |
| `sortable(data)` | POST | 排序 |

```typescript
// 使用方式
const api = new baTableApi('/admin/{module}.{Module}/')
// 实际调用：api.index() → GET /admin/{module}.{Module}/index
//           api.postData('add', data) → POST /admin/{module}.{Module}/add
```

### Axios 封装特性

- 每次请求创建新的 Axios 实例
- 自动附加 token：`batoken`（后台）、`ba-user-token`（会员）
- 重复请求自动取消
- 409 响应自动刷新 token
- 响应格式：`{ code: 1, msg: '...', data: {...} }`（code 1 = 成功）
- 选项：`loading`、`showSuccessMessage`、`showCodeMessage`、`cancelDuplicateRequest`
- 自定义 header：`think-lang`（i18n）、`server: true`

## Pinia Store 规范

### 文件结构

```
stores/
├── config.ts              # 布局配置
├── navTabs.ts             # 标签页管理
├── adminInfo.ts           # 管理员信息
├── siteConfig.ts          # 站点设置
├── userInfo.ts            # 前台会员信息
├── terminal.ts            # 终端状态
├── constant/
│   └── cacheKey.ts        # 缓存 key 常量
└── interface/
    └── index.ts           # 类型定义
```

### Store 模板（Composition API 风格）

```typescript
import { defineStore } from 'pinia'
import { reactive } from 'vue'
import { STORE_CONFIG } from './constant/cacheKey'

export const useConfig = defineStore('config', () => {
    const layout = reactive({
        layoutMode: 'Default',
        // ...
    })
    return { layout }
}, { persist: { key: STORE_CONFIG } })
```

### Store 模板（Options API 风格）

```typescript
import { defineStore } from 'pinia'
import { ADMIN_INFO } from './constant/cacheKey'

export const useAdminInfo = defineStore('adminInfo', {
    state: (): AdminInfo => ({
        id: 0,
        username: '',
        nickname: '',
        avatar: '',
        last_login_time: 0,
        token: '',
        refresh_token: '',
        super: false,
    }),
    actions: {
        dataFill(state: Partial<AdminInfo>, exclude: boolean | string[] = true) {
            if (exclude === true) {
                exclude = ['token', 'refresh_token']
            } else if (exclude === false) {
                exclude = []
            }
            if (Array.isArray(exclude)) {
                exclude.forEach((item) => { delete state[item as keyof AdminInfo] })
            }
            this.$patch(state)
        },
    },
    persist: { key: ADMIN_INFO },
})
```

## 路由规范

- 使用 Hash 模式（`createWebHashHistory`）
- 静态路由定义在 `router/static.ts`
- 动态路由从后端数据库加载
- 组件通过 `import.meta.glob('/src/views/backend/**/*.vue')` 自动匹配
- 后台基础路径：`/admin`（在 `static/adminBase.ts` 配置）

## 布局系统

```
layouts/
├── backend/
│   ├── index.vue          # 动态布局选择器
│   ├── container/         # Classic、Default、Double、Streamline、LeftSplit
│   └── components/        # 侧边栏、头部、标签页等
├── frontend/
│   ├── user.vue
│   └── container/
└── common/
    └── components/        # loading、iframe
```

布局通过 `config.layout.layoutMode` 动态切换。

## 工具函数

| 文件 | 用途 |
|------|------|
| `utils/axios.ts` | HTTP 客户端封装 |
| `utils/baTable.ts` | CRUD 表格管理类（684 行） |
| `utils/common.ts` | 权限检查、URL 辅助、防抖、时间格式化 |
| `utils/validate.ts` | 表单验证器 |
| `utils/router.ts` | 动态路由处理、菜单处理 |
| `utils/directives.ts` | 自定义指令 |
| `utils/storage.ts` | localStorage/sessionStorage 封装 |

## 自定义指令

- `v-auth="'add'"` — 权限检查，无权限时移除元素
- `v-drag="['.dialog', '.header']"` — 对话框拖拽
- `v-zoom="'.dialog'"` — 对话框缩放
- `v-blur` — 聚焦时自动失焦（用于 loading 按钮）
- `v-tableLateralDrag` — 表格横向滚动

## 表单验证

使用 `buildValidatorData()` 构建验证规则：

```typescript
const rules = reactive({
    name: [
        buildValidatorData({ name: 'required', title: '名称' }),
    ],
    mobile: [
        buildValidatorData({ name: 'mobile' }),
    ],
})
```

支持类型：`required`、`mobile`、`idNumber`、`account`、`password`、`varName`、`editorRequired`、`number`、`integer`、`float`、`date`、`url`、`email`

## 字段渲染

表格列定义的 `render` 属性：

| render | 用途 | 配合属性 |
|--------|------|----------|
| `tag` | 状态标签 | `custom`（颜色映射）、`replaceValue`（值替换） |
| `datetime` | 时间 | — |
| `image` | 图片 | — |
| `color` | 颜色 | — |
| `buttons` | 操作按钮 | — |

```ts
{
    label: '状态',
    prop: 'status',
    render: 'tag',
    custom: { 0: 'danger', 1: 'success' },
    replaceValue: { 0: '禁用', 1: '启用' },
}
```

## 依赖版本

| 包 | 版本 |
|---|---|
| Vue | 3.5.33 |
| Vue Router | 5.0.6 |
| Pinia | 3.0.4 |
| Element Plus | 2.13.7 |
| TypeScript | 6.0.3 |
| Vite | 8.0.10 |
| Axios | 1.17.0 |
| vue-i18n | 11.4.0 |
| ECharts | 6.0.0 |
| @vueuse/core | 14.2.1 |
