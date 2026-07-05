# 开发命令

## 后端（ThinkPHP）

```bash
# 启动后端开发服务器（默认端口 8000）
php think run --port=8000

# 数据库迁移（使用 Phinx）
php think migrate:run
php think migrate:create <name>
```

## 前端（Vue3，工作目录 `web/`）

```bash
cd web
pnpm install          # 安装依赖（首次必装）
pnpm dev              # 启动前端开发服务器（默认端口 5173）
pnpm build            # 生产构建
pnpm lint             # ESLint 检查
pnpm lint-fix         # ESLint 自动修复
pnpm format           # Prettier 格式化
pnpm typecheck        # TypeScript 类型检查
```

## 注意事项

- 前端 `web/` 的 `.npmrc` 中 `shamefully-hoist` 已注释，使用 pnpm 默认严格模式
- `pnpm build` 输出到 `VITE_OUT_DIR` 环境变量指定的目录
