# Agent Instructions

> Scope: BuildAdmin 后台管理系统（Vue3 + ThinkPHP8 + TypeScript + Vite + Element Plus）
> 本文件是路由入口，仅保留必须始终遵循的规则；详细文档按需加载。

## 技术栈

- **框架**: Vue 3.5 + TypeScript 6 + Vite 8
- **UI 组件**: Element Plus 2.13
- **状态管理**: Pinia 3 + pinia-plugin-persistedstate
- **路由**: Vue Router 5（Hash 模式）
- **包管理器**: pnpm（必须使用 pnpm，禁止使用 npm/yarn）
- **HTTP**: Axios 1.17（通过自定义 `createAxios` 封装）
- **后端**: ThinkPHP 8.1（PHP >= 8.2）
- **数据库**: MySQL 8.x，表前缀 `ba_`

## 目录结构

```
├── app/                    # ThinkPHP8 后端（多应用架构）
│   ├── admin/              # 后台管理应用
│   │   ├── controller/     # 控制器（auth/, crud/, routine/, security/, user/）
│   │   ├── model/          # 模型
│   │   ├── validate/       # 验证器
│   │   └── library/        # Traits、Auth、CRUD 辅助、Stubs
│   ├── api/                # 对外接口应用（前端会员端）
│   ├── common/             # 公共应用（禁止 URL 直接访问）
│   │   ├── controller/     # Backend.php、Api.php、Frontend.php 基类
│   │   ├── model/          # 共享模型
│   │   └── library/        # Auth.php（Token 管理）等
│   └── BaseController.php  # 根抽象控制器
├── config/                 # ThinkPHP 配置
├── database/migrations/    # 数据库迁移（Phinx）
├── extend/ba/              # 自定义扩展（Tree、Auth、Captcha 等）
├── modules/                # 模块市场（可一键安装）
├── web/                    # Vue3 前端（独立应用）
│   └── src/
│       ├── api/            # API 请求模块
│       ├── components/     # 全局业务组件（table、formItem、baInput 等）
│       ├── lang/           # i18n 多语言
│       ├── layouts/        # 布局组件（backend/、frontend/、common/）
│       ├── router/         # 路由配置
│       ├── stores/         # Pinia store
│       ├── styles/         # SCSS 样式
│       ├── utils/          # 工具函数
│       └── views/          # 页面视图（backend/、frontend/、common/）
└── .env                    # 后端环境配置（参考 .env-example）
```

## 常用命令

```bash
# 后端
php think run --port=8000          # 启动后端开发服务器
php think migrate:run              # 执行数据库迁移
php think migrate:create <name>    # 创建迁移文件

# 前端（工作目录 web/）
cd web
pnpm install          # 安装依赖（首次必装）
pnpm dev              # 启动前端开发服务器（端口 1818）
pnpm build            # 生产构建
pnpm lint             # ESLint 检查
pnpm lint-fix         # ESLint 自动修复
pnpm format           # Prettier 格式化
pnpm typecheck        # TypeScript 类型检查
```

## 硬性约束（MUST / FORBIDDEN）

> 违反以下任一规则视为功能未完成。详细说明见 `docs/` 对应文档。

### 后端

| 规则 | 说明 | 详见 |
|------|------|------|
| SoftDelete 路径 | `think\model\concern\SoftDelete`（**禁止** `traits\model\SoftDelete`） | [backend-conventions.md](docs/backend-conventions.md) |
| 时间字段 | `createtime`/`updatetime`（**无下划线**），unix 时间戳 | [backend-conventions.md](docs/backend-conventions.md) |
| `$autoWriteTimestamp` | 必须设为 `'int'`（**禁止** `true`） | [backend-conventions.md](docs/backend-conventions.md) |
| 表前缀 | `ba_`，迁移中原始 SQL **必须**带前缀 | [backend-conventions.md](docs/backend-conventions.md) |
| 索引操作 | **必须**用 `$this->execute()` 原始 SQL（Phinx `update()` 不加前缀） | [backend-conventions.md](docs/backend-conventions.md) |
| 控制器 URL | 用 `.` 分隔：`/admin/wallpaper.Wallpaper/index`（**禁止** `/` 分隔） | [backend-conventions.md](docs/backend-conventions.md) |
| 登录接口 | `/admin/Index/login`（**禁止** `/admin/auth.admin/login`） | [admin-module.md](docs/admin-module.md) |
| Token 传递 | 后台用 `ba-token` header，前台用 `Authorization: Bearer` | [backend-conventions.md](docs/backend-conventions.md) |
| 状态字段语义 | user 表用 `enable`/`disable`，业务表按迁移定义 | [backend-conventions.md](docs/backend-conventions.md) |
| 跨模块 Model | admin 内用 `app\admin\model\X`，用户模型用 `app\model\User` | [backend-conventions.md](docs/backend-conventions.md) |

### 前端

| 规则 | 说明 | 详见 |
|------|------|------|
| Router 模式 | **必须** Hash（`createWebHashHistory`），**禁止**改为 History | [frontend-conventions.md](docs/frontend-conventions.md) |
| 后台 URL | `http://localhost:1818/#/admin/login`（**必须**有 `#`） | [frontend-conventions.md](docs/frontend-conventions.md) |
| defineOptions name | **必须**与菜单 `name` 字段完全一致 | [frontend-conventions.md](docs/frontend-conventions.md) |
| API 路径 | 用 `.` 分隔：`/admin/wallpaper.Wallpaper/`（**禁止** `/` 分隔） | [frontend-conventions.md](docs/frontend-conventions.md) |
| 字段标签 | 直接用中文，**禁止**用 `t('wallpaper.xxx.title')` 这类未维护的翻译键 | [frontend-conventions.md](docs/frontend-conventions.md) |
| baTableApi URL | 末尾带 `/`：`'/admin/wallpaper.Wallpaper/'` | [code-style.md](docs/code-style.md) |
| keepalive 字段 | tinyint（`1`/`0`），**禁止**填字符串 | [admin-module.md](docs/admin-module.md) |
| server.proxy | **禁止**配置，API 跨域通过 CORS 实现 | [frontend-conventions.md](docs/frontend-conventions.md) |
| 端口 | 1818=前端 dev，8000=后端 API（**禁止**混淆） | [environment.md](docs/environment.md) |

### 后台模块四件套（缺一不可）

| # | 文件 | 路径 |
|---|------|------|
| 1 | PHP 控制器 | `app/admin/controller/{module}/Xxx.php` |
| 2 | PHP 模型 | `app/admin/model/Xxx.php` |
| 3 | 数据库迁移 | `database/migrations/YYYYMMDDHHMMSS_xxx.php` |
| 4 | Vue 页面 | `web/src/views/backend/{module}/xxx/index.vue` + `popupForm.vue` |

**菜单也必须由迁移初始化**（`ba_admin_rule` 表），不可假设"会自动出现"。
详见 [admin-module.md](docs/admin-module.md)。

### API 响应格式

```json
{ "code": 1, "msg": "成功", "data": {} }   // 成功
{ "code": 0, "msg": "失败", "data": {} }   // 失败
```

### 核心约定

- CRUD 页面使用 `baTableClass` 管理表格和表单状态
- 后端路由从数据库动态加载，前端通过 `import.meta.glob` 自动匹配组件
- 布局模式通过 `config.layout.layoutMode` 动态切换
- 代码提交前会自动运行 lint-staged
- 默认账号：用户名 `admin`，密码 `Admin123`

## Read-On-Demand Index

### 硬性约束（开发前必读）

| 场景 | 文档 | 触发条件 |
|------|------|----------|
| 后端开发 | [docs/backend-conventions.md](docs/backend-conventions.md) | 编写 PHP 代码、Model、迁移时 |
| 前端开发 | [docs/frontend-conventions.md](docs/frontend-conventions.md) | 编写 Vue 页面、表单、表格列时 |
| 新增后台模块 | [docs/admin-module.md](docs/admin-module.md) | 新建 Controller/Model/Vue/迁移时 |
| 模块/插件开发 | [docs/module-system.md](docs/module-system.md) | 开发可安装模块、理解模块生命周期时 |
| 功能完成验收 | [docs/completion-checklist.md](docs/completion-checklist.md) | 声称"已完成"功能前必须逐项核对 |
| 前端代码格式 | [docs/code-style.md](docs/code-style.md) | Prettier/ESLint/Vue 模板/Pinia 规范 |

### 框架参考（按需查阅）

| 场景 | 文档 | 触发条件 |
|------|------|----------|
| 理解后端架构 | [docs/architecture.md](docs/architecture.md) | 需要理解控制器继承链、Model 模板时 |
| 环境配置 | [docs/environment.md](docs/environment.md) | 涉及 `.env`、端口、构建配置时 |
| 项目概览 | [docs/00-overview.md](docs/00-overview.md) | 首次接触项目 |
| 目录结构 | [docs/02-directory-structure.md](docs/02-directory-structure.md) | 需要完整目录树时 |
| 内置组件 | [docs/05-components.md](docs/05-components.md) | 使用 Icon/baInput/FormItem 时 |
| 表格系统 | [docs/06-table-system.md](docs/06-table-system.md) | 使用 baTable/baTableApi 时 |
| 表单验证 | [docs/07-form-system.md](docs/07-form-system.md) | 使用 buildValidatorData 时 |
| 路由与权限 | [docs/08-routing-permissions.md](docs/08-routing-permissions.md) | 配置菜单规则、v-auth 时 |
| 状态管理 | [docs/09-state-management.md](docs/09-state-management.md) | 使用 Pinia store 时 |
| 网络请求 | [docs/10-networking.md](docs/10-networking.md) | 使用 createAxios/useFetch 时 |
| 国际化 | [docs/11-internationalization.md](docs/11-internationalization.md) | 多语言功能 |
| 安全 | [docs/12-security.md](docs/12-security.md) | XSS/验证码/安全防范 |
| 部署 | [docs/13-deployment.md](docs/13-deployment.md) | 上线部署 |
| 问题排查 | [docs/14-troubleshooting.md](docs/14-troubleshooting.md) | 遇到跨域/路由/终端问题 |

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

## Priority

1. 用户当前的明确指令
2. 本文件（硬性约束）
3. `docs/` 中的按需文档
