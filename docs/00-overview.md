# BuildAdmin 项目概览

## 技术栈

| 层级 | 技术 | 版本要求 |
|------|------|----------|
| 后端框架 | ThinkPHP 8 | PHP >= 8.2 |
| 前端框架 | Vue 3 + TypeScript | Vue 3.x |
| 构建工具 | Vite | - |
| UI 组件库 | Element Plus | - |
| 状态管理 | Pinia | vue3 官方推荐 |
| 数据库 | MySQL | >= 5.7（需 InnoDB） |
| 包管理器 | pnpm/npm/yarn | Node >= 22.13 |
| HTTP 客户端 | Axios（Web）/ useFetch（WebNuxt） | - |
| 国际化 | vue-i18n（Web）/ TP 多语言（Server） | - |

## 项目定位

BuildAdmin 是基于 ThinkPHP8 + Vue3 的开源后台管理系统，支持：
- 可视化 CRUD 代码生成
- 内置 WEB 终端
- 五种后台布局方式
- 前后端双鉴权（菜单级 + 按钮级）
- 数据权限控制
- 国际化多语言
- 暗黑模式

## 核心架构

```
buildadmin/
├── app/                  # ThinkPHP8 后端（多应用模式）
│   ├── admin/            # 后台管理应用
│   ├── api/              # 前台会员端 API
│   └── common/           # 公共应用（基类、共享逻辑）
├── config/               # 配置文件（buildadmin.php, terminal.php 等）
├── database/             # 数据库迁移（Phinx）
├── extend/               # 扩展目录（\ba\ 命名空间）
├── public/               # 运行目录（入口文件、静态资源、上传文件）
├── runtime/              # 运行时缓存、日志
├── web/                  # Vue3 前端源码
│   └── src/
│       ├── api/          # 接口请求函数
│       ├── components/   # 全局组件
│       ├── lang/         # 语言包
│       ├── layouts/      # 布局组件
│       ├── router/       # 路由配置
│       ├── stores/       # Pinia 状态管理
│       ├── styles/       # SCSS 样式
│       ├── utils/        # 工具函数
│       └── views/        # 页面视图（backend/frontend/common）
└── web-nuxt/             # 可选的 Nuxt 工程（SEO、SSR）
```

## 关键约定

1. **开发环境**：使用 `php think run` 启动后端（端口 8000），`npm run dev` 启动前端（端口 1818）
2. **生产环境**：使用 Nginx/Apache，站点根目录为项目目录，运行目录为 `public/`
3. **API URL 格式**：`/admin/{Controller}.{action}`（如 `/admin/user.User/index`）
4. **响应格式**：`{ code: 1, msg: '...', data: {...} }`，code=1 为成功，code=0 为失败
5. **时间戳字段**：`createtime`、`updatetime`（无下划线，unix 时间戳）
