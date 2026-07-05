# Agent Instructions

> Scope: BuildAdmin 后台管理系统（Vue3 + ThinkPHP8 + TypeScript + Vite + Element Plus）
> 本文件是路由入口，仅保留必须始终遵循的规则；详细文档按需加载。

## Core Principles

- 前后端完全分离：`web/` 是独立 Vue3 应用，后端是 ThinkPHP8 多应用架构
- `web/` 的前端 API 地址在 `web/.env.development` 中配置（默认 `http://localhost:8000`）
- 数据库表前缀 `ba_`，连接配置在项目根目录 `.env`（参考 `.env-example`）
- CRUD 代码生成器会自动创建数据库表和控制器/模型代码
- `app/common` 应用禁止 URL 直接访问
- 路径别名 `/@` → `web/src/`

## Read-On-Demand Index

| 场景 | 读取文档 | 触发条件 |
| --- | --- | --- |
| 执行开发/构建/检查命令 | [docs/commands.md](docs/commands.md) | 需要运行任何构建、启动、lint、typecheck 命令时 |
| 修改后端代码或理解后端结构 | [docs/architecture.md](docs/architecture.md) | 编辑 `app/`、`config/`、`database/`、`modules/` 目录下文件时 |
| 修改前端代码或理解前端结构 | [docs/code-style.md](docs/code-style.md) | 编辑 `web/src/` 下文件、涉及 Vue/TS 规范时 |
| 配置环境变量或排查连接问题 | [docs/environment.md](docs/environment.md) | 涉及 `.env`、数据库连接、API 地址配置时 |

## Always-On Rules

- 后端入口：`php think run --port=8000`
- 前端入口：`cd web && pnpm dev`（端口 5173）
- Prettier：4 空格缩进，无分号，单引号，行宽 150，换行符 `lf`
- Vue 文件使用 `<script setup lang="ts">` + `useTemplateRef`
- 状态管理：Pinia + `pinia-plugin-persistedstate`
- UI 组件：Element Plus
- PHP >= 8.2，命名空间 `app\` 和 `modules\`，自动时间戳 `create_time`、`update_time`

## Priority

1. 用户当前的明确指令
2. 本文件
3. `docs/` 中的按需文档
