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
│   │   └── model/          # 共享模型
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
pnpm dev              # 启动前端开发服务器（端口 5173）
pnpm build            # 生产构建
pnpm lint             # ESLint 检查
pnpm lint-fix         # ESLint 自动修复
pnpm format           # Prettier 格式化
pnpm typecheck        # TypeScript 类型检查
```

## 开发规范

### 前端

- 使用 `<script setup lang="ts">` 语法，所有 Vue 组件必须添加 `defineOptions({ name: '...' })`
- 模板引用使用 `useTemplateRef()` 而非 `ref()`
- 父子组件共享 `baTable` 状态使用 `provide/inject` 模式
- 国际化使用 `useI18n()` + `t()` 函数
- 所有 API 调用通过 `createAxios()` 封装，响应格式：`{ code: 1, msg: '...', data: {...} }`
- Store 使用 `pinia-plugin-persistedstate`，缓存 key 定义在 `stores/constant/cacheKey.ts`
- 接口类型定义在 `stores/interface/index.ts`
- 路径别名 `/@` → `src/`
- 自定义指令：`v-auth`（权限）、`v-drag`（拖拽）、`v-zoom`（缩放）、`v-blur`（失焦）

### 后端

- 控制器继承 `app\common\controller\Backend`，在 `initialize()` 中初始化 Model
- 响应格式：`$this->success('msg', data)` / `$this->error('msg')`
- 自动时间戳字段：`create_time`、`update_time`
- 数据权限通过 `$dataLimit` 属性控制
- API URL 格式：`/admin/{Controller}.{action}`（如 `/admin/user.User/index`）
- 多步操作使用事务：`$this->model->startTrans()` / `->commit()` / `->rollback()`

## 核心约定

- CRUD 页面使用 `baTableClass` 管理表格和表单状态
- 后端路由从数据库动态加载，前端通过 `import.meta.glob` 自动匹配组件
- 布局模式通过 `config.layout.layoutMode` 动态切换
- 代码提交前会自动运行 lint-staged

## Read-On-Demand Index

| 场景 | 读取文档 | 触发条件 |
| --- | --- | --- |
| 修改后端代码或理解后端结构 | [docs/architecture.md](docs/architecture.md) | 编辑 `app/`、`config/`、`database/`、`modules/` 目录下文件时 |
| 修改前端代码或理解前端结构 | [docs/code-style.md](docs/code-style.md) | 编辑 `web/src/` 下文件、涉及 Vue/TS 规范时 |
| 配置环境变量或排查连接问题 | [docs/environment.md](docs/environment.md) | 涉及 `.env`、数据库连接、API 地址配置时 |

## Priority

1. 用户当前的明确指令
2. 本文件
3. `docs/` 中的按需文档
