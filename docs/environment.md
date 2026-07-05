# 环境配置

## 后端

- 配置文件：项目根目录 `.env`（参考 `.env-example`）
- 数据库默认连接：`127.0.0.1:3306`，数据库 `buildadmin_com`，用户 `root`，密码 `admin888`
- 数据库配置详情：`config/database.php`
- 表前缀：`ba_`

## 前端

- 开发环境：`web/.env.development`
  - `VITE_AXIOS_BASE_URL = 'http://localhost:8000'`
- 生产环境：`web/.env.production`
- 构建输出目录：`VITE_OUT_DIR`
- 基础路径：`VITE_BASE_PATH`
