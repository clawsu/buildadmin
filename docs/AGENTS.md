# AGENTS.md — BuildAdmin AI 开发规范

> **AI 开发时必须遵循本文件。违反 MUST/FORBIDDEN 规则的代码视为未完成。**

---

## 开发前必读

每次开始编码前，先确认你属于哪个场景，读对应的规范文件：

| 场景 | 必读文件 |
|------|----------|
| 新增后台管理模块 | `admin-module.md` → `backend-conventions.md` → `frontend-conventions.md` |
| 修改后端 PHP | `backend-conventions.md` |
| 修改前端 Vue | `frontend-conventions.md` → `code-style.md` |
| 开发市场插件包 | `module-system.md` |
| 修复 Bug | 对应层的 conventions 文件 |
| 声称"已完成"前 | `completion-checklist.md` |

---

## 关键规则速查（内联）

### 后端 MUST

| 规则 | 正确 | 错误 |
|------|------|------|
| SoftDelete trait | `think\model\concern\SoftDelete` | `traits\model\SoftDelete` |
| 时间字段 | `createtime`/`updatetime`（无下划线） | `create_time`/`update_time` |
| `$autoWriteTimestamp` | `'int'` | `true` |
| 表前缀 | `ba_`，迁移原始 SQL 必须带 | 忘记前缀 |
| 索引操作 | `$this->execute()` 原始 SQL | `$this->table()->addIndex()->update()` |
| 控制器 URL 分隔符 | `/admin/{module}.{Module}/index`（`.`） | `/admin/{module}/{module}/index`（`/`） |
| 登录接口 | `/admin/Index/login` | `/admin/auth.admin/login` |
| Token 传递 | 后台 `ba-token` header | `Authorization: Bearer` |
| 状态字段（user 表） | `enable`/`disable` | `normal`/其他值 |
| 控制器 noNeedLogin | 登录方法必须声明 | 忘记声明导致 303 |

### 前端 MUST

| 规则 | 正确 | 错误 |
|------|------|------|
| Router 模式 | `createWebHashHistory()` | `createWebHistory()` |
| 后台 URL | `http://localhost:1818/#/admin/login`（有`#`） | `http://localhost:1818/admin/` |
| `defineOptions name` | 与菜单 `name` 字段完全一致 | 任意命名 |
| API 路径分隔符 | `/admin/{module}.{Module}/`（`.`） | `/admin/{module}/{module}/`（`/`） |
| 字段标签 | 直接用中文 | `t('xxx.title')` |
| `keepalive` 字段 | tinyint `1`/`0` | 字符串 |
| `server.proxy` | 禁止配置 | 添加 proxy |
| 端口 | 1818=前端 dev，8000=后端 API | 混用 |

### 后台模块四件套（缺一不可）

| # | 文件 | 路径 |
|---|------|------|
| 1 | PHP 控制器 | `app/admin/controller/{module}/Xxx.php` |
| 2 | PHP 模型 | `app/admin/model/Xxx.php` |
| 3 | 数据库迁移 | `database/migrations/YYYYMMDDHHMMSS_xxx.php` |
| 4 | Vue 页面 | `web/src/views/backend/{module}/xxx/index.vue` + `popupForm.vue` |

**菜单必须由迁移初始化**（插入 `ba_admin_rule` 表），不可假设"会自动出现"。

### API 响应格式

```json
{ "code": 1, "msg": "成功", "data": {} }
{ "code": 0, "msg": "失败", "data": {} }
```

### 默认账号

| 系统 | 用户名 | 密码 | 地址 |
|------|--------|------|------|
| 后台管理 | admin | Admin123 | `http://localhost:1818/#/admin/login` |

---

## 文件索引

### 第一层：硬性约束（必须遵循）

| 文件 | 覆盖范围 | 关键规则 |
|------|----------|----------|
| [`backend-conventions.md`](backend-conventions.md) | Model / Migration / Controller / Auth / 数据库 | R1.1-R5.3，共 15 条强制规则 |
| [`frontend-conventions.md`](frontend-conventions.md) | Vue Router / 组件 / 表单 / 环境配置 | F1.1-F5.2，共 12 条强制规则 |
| [`admin-module.md`](admin-module.md) | 新增后台模块四件套 / 菜单初始化 / 登录接口 | A1-A4，含迁移模板和验证方法 |
| [`module-system.md`](module-system.md) | 市场插件包结构 / CRUD 代码生成器 | info.ini / config.json / 事件类 / 生命周期 |
| [`completion-checklist.md`](completion-checklist.md) | 功能完成验收清单 | 逐项核对，含验证命令 |
| [`code-style.md`](code-style.md) | Prettier / ESLint / Vue 模板 / Pinia / 指令 | 4空格/无分号/单引号/行宽150 |

### 第二层：框架参考（按需查阅）

| 文件 | 内容 |
|------|------|
| [`architecture.md`](architecture.md) | 后端架构、控制器继承链、Model 模板、迁移规范 |
| [`environment.md`](environment.md) | 环境变量、端口职责、构建配置、插件注册顺序 |
| [`00-overview.md`](00-overview.md) | 项目概览、技术栈 |
| [`01-getting-started.md`](01-getting-started.md) | 环境搭建、安装步骤 |
| [`02-directory-structure.md`](02-directory-structure.md) | Server/Web 端完整目录树 |
| [`03-backend-conventions.md`](03-backend-conventions.md) | 后端开发参考（控制器/数据表/验证码） |
| [`04-frontend-conventions.md`](04-frontend-conventions.md) | 前端开发参考（组件/指令/样式/Git） |
| [`05-components.md`](05-components.md) | 内置组件（Icon/baInput/FormItem） |
| [`06-table-system.md`](06-table-system.md) | 表格系统（baTable/baTableApi） |
| [`07-form-system.md`](07-form-system.md) | 表单验证、弹窗表单 |
| [`08-routing-permissions.md`](08-routing-permissions.md) | 路由与权限 |
| [`09-state-management.md`](09-state-management.md) | Pinia 状态管理 |
| [`10-networking.md`](10-networking.md) | 网络请求 |
| [`11-internationalization.md`](11-internationalization.md) | 国际化多语言 |
| [`12-security.md`](12-security.md) | 安全与防护 |
| [`13-deployment.md`](13-deployment.md) | 部署 |
| [`14-troubleshooting.md`](14-troubleshooting.md) | 常见问题 |
| [`15-module-development.md`](15-module-development.md) | 模块开发（市场插件包）补充参考 |

---

## 常见错误速查

| 错误现象 | 根因 | 修复 |
|----------|------|------|
| `Trait not found` | SoftDelete 路径错 | 改为 `think\model\concern\SoftDelete` |
| `Unknown column deletetime` | 表无字段却用 SoftDelete | 检查表结构，移除 SoftDelete |
| `Table 'xxx' doesn't exist` | Phinx 不加 ba_ 前缀 | 迁移中用 `$this->execute()` |
| 后台返回 `code:303` | 未登录或 token 错 | 用 `ba-token` header |
| 登录页不显示 | URL 缺 `#` | 用 `/#/admin/login` |
| 菜单不显示 | 未初始化菜单数据 | 迁移中插入 `ba_admin_rule` |
| Vue 页面 404 | component 路径错 | 检查 `component` 字段 |
| `Incorrect integer value` | keepalive 填字符串 | 用 `1`/`0` |
| 控制器不存在 | URL 用 `/` 分隔 | 用 `.` 分隔 |
| 用户被误判禁用 | status 语义不一致 | 用 `enable/disable` |
