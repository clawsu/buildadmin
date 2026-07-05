# 路由与权限

## 菜单规则管理

BuildAdmin 的路由与权限在后台可视化管理：`权限管理 -> 菜单规则管理`。

### 关键字段

| 字段 | 说明 |
|------|------|
| 规则名称 | 英文，自动注册为 Vue Router 的 route name，用 `/` 分隔层次 |
| 路由路径 | 如 `/admin/auth/group`，建议与规则名称一致 |
| 菜单类型 | 选项卡（有 vue+控制器代码）、链接、Iframe |
| 组件路径 | 如 `/src/views/backend/auth/group/index.vue` |
| 规则权重 | 数字越大越靠前（控制台=999） |
| 扩展属性 | 可配置只注册为路由或菜单，实现同一路由多菜单 |
| 缓存 | 开启后使用 keep-alive 保留标签页状态 |
| 上级菜单 | 父级规则（不能是按钮） |

### 权限节点

在菜单规则管理中可添加按钮级权限节点（如 add、edit、delete），这些节点会自动加载到管理员的后台。

## 前端鉴权

### 指令式

```vue
<el-button v-auth="'add'">添加</el-button>
<el-button v-auth="'edit'">编辑</el-button>
<el-button v-auth="'del'">删除</el-button>
```

### 函数式

```ts
import { auth } from '/@/utils/common'

// 按当前路由 path 鉴权
if (auth('add')) { /* 有权限 */ }

// 按路由 name 鉴权（推荐，不受当前 path 限制）
auth({ name: '/admin/auth/group', subNodeName: '/admin/auth/group/add' })

// 菜单权限检查（拥有该菜单下任意一个权限节点即可）
auth({ name: '/admin/auth/group' })
```

## 权限实现原理

1. 管理员访问后台时，请求 `/admin/index/index` 获取菜单规则
2. 通过 `\ba\Auth` 类只获取管理员拥有的规则
3. WEB 端将规则解析为 Vue Router 路由并自动注册
4. 设定后台左侧菜单，记录按钮权限
5. 使用 `v-auth` 或 `auth()` 时，拼接 `当前路由/权限名称` 与按钮权限比对

### 鉴权拼接示例

```
当前路由：/#/admin/auth/group
v-auth="'add'"  → 拼接为 auth/group/add
v-auth="'del'"  → 拼接为 auth/group/del
```

## 前后台双鉴权

- **后台**：通过管理员 token 鉴权，继承 `Backend` 基类
- **前台**：通过会员 token 鉴权，继承 `Frontend` 基类
- 两种鉴权完全独立

## 数据权限

不同于菜单权限，数据权限控制的是：同一个管理功能中，不同管理员只能查看/编辑有权的数据行。

详见后端开发规范中的「数据权限控制」部分。
