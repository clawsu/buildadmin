# 前端代码规范（硬性约束）

> 本文档记录前端 Vue 代码的强制规则。适用于 `web/` 目录下的后台管理面板。
> 违反任一规则视为功能未完成。

---

## 1. Vue Router 规范

### F1.1 使用 Hash 模式

**MUST**: Vue Router 使用 `createWebHashHistory()`，URL 路径在 `#` 之后。
**FORBIDDEN**: 使用 `createWebHistory()`（History 模式）。
**FORBIDDEN**: 修改 `web/src/router/index.ts` 中的 history 模式。

**根因**: 项目路由配置在 `web/src/router/index.ts`，使用 Hash 模式。History 模式需要后端配合 fallback，本项目后端是 ThinkPHP 多应用，不提供该 fallback。

```ts
// ✅ 正例：当前配置（不可修改）
const router = createRouter({
    history: createWebHashHistory(),
    routes: staticRoutes,
})
```

### F1.2 后台访问 URL

**MUST**: 后台管理地址：`http://localhost:1818/#/admin/login`（注意 `#`）。
**FORBIDDEN**: 用 `http://localhost:1818/admin/`（无 `#`）访问后台。
**FORBIDDEN**: 用 `http://localhost:8000/admin/` 访问后台（返回 JSON 错误）。

**根因**: 
- 1818 端口是 Vite dev server，返回前端 HTML
- 8000 端口是 PHP API 后端，`/admin/` 被当成 API 路由返回 `{code:303, msg:"请先登录！"}`
- 无 `#` 的 URL 会让 Vue Router 走默认 `/` 路由（前台首页），不显示登录页

| URL | 结果 |
|---|---|
| `http://localhost:1818/#/admin/login` | ✅ 后台登录页 |
| `http://localhost:1818/admin/` | ❌ 显示前台首页（会员中心已关闭，看似空白） |
| `http://localhost:8000/admin/` | ❌ 返回 JSON `{code:303}` |
| `http://localhost:8000/index.html#/user/login` | ❌ 前台会员登录页（已禁用，报"账户不存在"） |

### F1.3 端口职责

**MUST**: 
- `1818` 端口：Vite dev server（前端开发）
- `8000` 端口：PHP API 后端（API 请求）
- 后台管理前端通过 `web/.env.development` 的 `VITE_AXIOS_BASE_URL = 'http://localhost:8000'` 跨域请求 API

---

## 2. Vue 组件规范

### F2.1 后台页面结构（四件套）

**MUST**: 每个后台业务模块的 Vue 页面包含：
- `web/src/views/backend/{module}/{submodule}/index.vue` — 列表页
- `web/src/views/backend/{module}/{submodule}/popupForm.vue` — 表单弹窗

**MUST**: `index.vue` 必须包含以下结构：

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

defineOptions({
    name: '{module}/xxx',  // 必须与菜单 name 一致
})

const formRef = useTemplateRef('formRef')
const tableRef = useTemplateRef('tableRef')

const baTable: baTableClass = new baTableClass(
    new baTableApi('/admin/{module}.Xxx/'),  // 注意 . 分隔
    {
        column: [/* 字段定义 */],
    }
)

provide('baTable', baTable)

onMounted(() => {
    baTable.table.ref = tableRef.value
    baTable.mount()
    baTable.getData()
})
</script>
```

### F2.2 defineOptions name 必须与菜单一致

**MUST**: `defineOptions({ name: '{module}/xxx' })` 必须与 `ba_admin_rule` 表中菜单记录的 `name` 字段完全一致。

**根因**: BuildAdmin 路由通过 name 匹配菜单，不一致会导致页面加载后路由不识别、keep-alive 失效。

```ts
// ✅ 正例
defineOptions({ name: '{module}/{module}' })  // 对应菜单 name: {module}/{module}

// ❌ 反例
defineOptions({ name: '{Module}' })  // 与菜单不匹配
defineOptions({ name: '{module}/{Module}' })  // 大小写不一致
```

### F2.3 API 路径用点分隔

**MUST**: `baTableApi` 的 URL 用 `.` 分隔嵌套控制器：`/admin/{module}.{Module}/`。
**FORBIDDEN**: 用 `/` 分隔：`/admin/{module}/{module}/`。

**根因**: 见后端规范 R3.4，ThinkPHP 嵌套控制器路由规则。

```ts
// ✅ 正例
new baTableApi('/admin/{module}.{Module}/')
new baTableApi('/admin/{module}.Category/')

// ❌ 反例
new baTableApi('/admin/{module}/{module}/')  // 404
```

### F2.4 字段标签用中文

**MUST**: 业务字段标签、placeholder 直接用中文字符串。
**FORBIDDEN**: 用 `t('xxx.title')` 这类 i18n 业务翻译键（翻译文件未维护）。

**例外**: 框架通用键可用 `t('Cancel')`、`t('Save')`、`t('Operate')`、`t('State')` 等。

```vue
<!-- ✅ 正例 -->
<el-form-item label="标题">
<el-input placeholder="请输入标题" />

<!-- ❌ 反例 -->
<el-form-item :label="t('xxx.title')">  <!-- 翻译键不存在 -->
```

---

## 3. 字段渲染规范

### F3.1 状态字段

**MUST**: 状态字段用 `render: 'tag'`，配合 `custom` 和 `replaceValue`。

```ts
{
    label: '状态',
    prop: 'status',
    render: 'tag',
    custom: { 0: 'danger', 1: 'success' },
    replaceValue: { 0: '禁用', 1: '启用' },
}
```

### F3.2 时间字段

**MUST**: 时间字段用 `render: 'datetime'`。

```ts
{ label: '创建时间', prop: 'createtime', render: 'datetime' }
```

### F3.3 图片字段

**MUST**: 图片字段用 `render: 'image'`。

```ts
{ label: '缩略图', prop: 'thumbnail', render: 'image' }
```

### F3.4 颜色字段

**MUST**: 颜色值列表用 `render: 'color'`，表单用 `el-color-picker` + `el-input` 双向绑定。

---

## 4. 表单规范

### F4.1 popupForm.vue 结构

**MUST**: 用 `el-dialog` + `inject('baTable')`，字段用 `v-model="baTable.form.items!.字段名"`。

```vue
<template>
    <el-dialog :model-value="['Add', 'Edit'].includes(baTable.form.operate!)" @close="baTable.toggleForm">
        <el-form ref="formRef" :model="baTable.form.items">
            <el-form-item label="名称" prop="name">
                <el-input v-model="baTable.form.items!.name" />
            </el-form-item>
        </el-form>
    </el-dialog>
</template>

<script setup lang="ts">
import { inject } from 'vue'
import type baTableClass from '/@/utils/baTable'
const baTable = inject('baTable') as baTableClass
</script>
```

### F4.2 远程选择器

**MUST**: 跨模块远程选择用 `type: 'remoteSelect'`，`remoteUrl` 指向目标模块的 index 接口。

```ts
// 表单中选择关联数据
FormItem 用 type="remoteSelect"
input-attr={{
    field: 'name',
    remoteUrl: '/admin/{module}.Category/index',
    placeholder: '点击选择',
}}
```

---

## 5. 环境配置规范

### F5.1 .env 文件

**MUST**: `web/.env` 配置 Vite 端口（`VITE_PORT = 1818`）。
**MUST**: `web/.env.development` 配置 API 地址（`VITE_AXIOS_BASE_URL = 'http://localhost:8000'`）。
**FORBIDDEN**: 修改 `VITE_AXIOS_BASE_URL` 为其他端口（除非后端迁移）。

### F5.2 vite.config.ts

**MUST**: 不配置 `server.proxy`。API 跨域通过 `VITE_AXIOS_BASE_URL` + 后端 CORS（`AllowCrossDomain` 中间件）实现。
**FORBIDDEN**: 添加 proxy 配置（会与 CORS 冲突）。
