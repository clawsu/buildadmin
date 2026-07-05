# 前端开发规范

## Vue 组件规范

- 使用 `<script setup lang="ts">` 语法
- 所有组件必须添加 `defineOptions({ name: '...' })`
- 模板引用使用 `useTemplateRef()` 而非 `ref()`
- 路径别名 `/@` → `src/`

## 网络请求

### Axios 封装（Web 端）

```ts
import createAxios from '/@/utils/axios'

export function getData() {
    return createAxios({
        url: '/admin/controller/action',
        method: 'get',
    })
}
```

**createAxios 参数**：

| 参数 | 说明 |
|------|------|
| axiosConfig | Axios 标准配置（url 必填，默认 get） |
| options.CancelDuplicateRequest | 取消重复请求（默认 true） |
| options.loading | loading 层效果（默认 false） |
| options.reductDataFormat | 简洁响应（默认 true，返回 ApiPromise） |
| options.showErrorMessage | 错误信息提示（默认 true） |
| options.showCodeMessage | code!=0 时提示（默认 true） |
| options.showSuccessMessage | code=1 时提示（默认 false） |
| loading | Element Plus loading 配置 |

### 响应格式

```ts
interface ApiResponse {
    code: number    // 1=成功, 0=失败
    msg: string
    data: any
}
```

## 内置指令

| 指令 | 用途 | 示例 |
|------|------|------|
| `v-auth` | 前端鉴权（按钮级） | `<el-button v-auth="'add'">` |
| `v-drag` | 拖动元素 | `v-drag="['.dialog', '.header']"` |
| `v-zoom` | 缩放元素 | `v-zoom="'.dialog'"` |
| `v-blur` | 点击后失焦 | `<el-button v-blur>` |
| `v-table-lateral-drag` | 表格横向滚动 | `<el-table v-table-lateral-drag>` |

### 前端鉴权（函数式）

```ts
import { auth } from '/@/utils/common'

// 按当前路由 path 鉴权
if (auth('add')) { /* 有权限 */ }

// 按路由 name 鉴权（推荐）
auth({ name: '/admin/auth/group', subNodeName: '/admin/auth/group/add' })
```

## 辅助工具函数

```ts
import { loadCss, loadJs, setTitle, isExternal } from '/@/utils/common'
import { debounce } from '/@/utils/common'
import { onResetForm } from '/@/utils/common'
import { isAdminApp, isMobile } from '/@/utils/common'
import { randomNum, uuid, shortUuid } from '/@/utils/random'
import { routePush } from '/@/utils/router'
import { Local, Session } from '/@/utils/storage'
```

## 本地缓存

```ts
Local.set('key', 'value')
Local.get('key')
Local.remove('key')
Local.clear()

Session.set('key', 'value')
Session.get('key')
Session.remove('key')
Session.clear()
```

## 样式规范

### styles 目录文件

| 文件 | 用途 |
|------|------|
| `var.scss` | 全局 CSS 变量定义 |
| `dark.scss` | 暗黑模式变量 |
| `element.scss` | Element Plus 样式覆盖 |
| `mixins.scss` | SCSS mixins 与 function |

### 预设颜色变量

```scss
// BuildAdmin 变量
--ba-color-primary-light: #3F6AD8
--ba-bg-color: #F5F5F5 (暗黑: #141414)
--ba-bg-color-overlay: #FFFFFF (暗黑: #1D1E1F)
--ba-border-color: #F6F6F6 (暗黑: #58585B)

// Element Plus 常用变量
--el-color-primary: #409EFF
--el-text-color-primary: #303133 (暗黑: #E5EAF3)
```

### 自定义变量

```scss
// 批量注册（自动加 --ba-vars- 前缀）
$vars: map.merge(('color-1': #f5f7fa), $vars);
@include set-component-css-var('vars', $vars);

// 单独定义（变量名 --ba-var-color）
@include set-css-var-value('var-color', #ffffff);
```

## Git 提交规范

```
type(<scope>):subject

<Body>

<Footer>
```

### type 类型

| 类型 | 说明 |
|------|------|
| feat | 新增功能 |
| fix | 修复 bug |
| docs | 文档变更 |
| style | 代码格式（不影响功能） |
| refactor | 代码重构 |
| perf | 改善性能 |
| test | 测试 |
| build | 构建或外部依赖变更 |
| chore | 构建流程或辅助工具 |
| revert | 代码回退 |

### scope 范围

Terminal、CRUD、BaTable、BaInput、Install、Utils、Layout、Lang

### subject 规范

- 动词开头，第一人称现在时
- 首字母小写
- 结尾不加句号
- 长度 ≤ 50 字符
