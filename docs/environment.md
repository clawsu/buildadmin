# 环境配置

## 后端环境（`.env`）

参考 `.env-example`：

```ini
APP_DEBUG = true

[APP]
DEFAULT_TIMEZONE = Asia/Shanghai

[LANG]
default_lang = zh-cn
```

数据库连接通过 `config/database.php` 配置，默认值：

| 配置项 | 默认值 |
|--------|--------|
| type | mysql |
| hostname | 127.0.0.1 |
| database | buildadmin_com |
| username | root |
| password | admin888 |
| hostport | 3306 |
| charset | utf8mb4 |
| prefix | ba_ |

## 前端环境

### 开发环境（`web/.env.development`）

```ini
ENV = 'development'
VITE_BASE_PATH = './'
VITE_AXIOS_BASE_URL = 'http://localhost:8000'
```

### 生产环境（`web/.env.production`）

```ini
VITE_AXIOS_BASE_URL = 'getCurrentDomain'  # 使用当前域名
VITE_OUT_DIR = 'dist'
```

### 关键环境变量

| 变量 | 说明 |
|------|------|
| `VITE_AXIOS_BASE_URL` | API 基础地址 |
| `VITE_PORT` | 开发服务器端口（默认 5173） |
| `VITE_OPEN` | 启动时是否自动打开浏览器 |
| `VITE_BASE_PATH` | 构建基础路径 |
| `VITE_OUT_DIR` | 构建输出目录 |

## BuildAdmin 核心配置（`config/buildadmin.php`）

- CORS 跨域配置
- 验证码设置
- 登录重试限制
- Token 配置（MySQL 或 Redis 存储、过期时间、加密）
- CDN URL
- 默认头像
- 管理员日志自动写入

## Git 配置

- `.gitattributes`：强制 LF 换行符（`* text=auto eol=lf`）
- `.gitignore`：忽略 `/vendor`、`/modules`、`/runtime`、`node_modules`、`dist`、`/.env`、IDE 文件
- `.editorconfig`：UTF-8、4 空格缩进、LF 换行

## 前端 `.npmrc`

`shamefully-hoist` 已注释，使用 pnpm 默认严格模式。

## 前端构建

- `vue-i18n` 在开发和生产环境加载不同的包（CJS）
- SVG 图标通过 `svgBuilder` 插件从 `src/assets/icons/` 自动构建
- `pnpm build` 输出到 `VITE_OUT_DIR` 指定的目录
- CSS 不拆分（`cssCodeSplit: false`）
- Source map 关闭
- chunk 大小警告阈值：1500KB
