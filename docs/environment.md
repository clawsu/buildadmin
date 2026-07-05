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

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `VITE_AXIOS_BASE_URL` | API 基础地址 | `http://localhost:8000` |
| `VITE_PORT` | 开发服务器端口 | `1818` |
| `VITE_OPEN` | 启动时是否自动打开浏览器 | — |
| `VITE_BASE_PATH` | 构建基础路径 | `./` |
| `VITE_OUT_DIR` | 构建输出目录 | `dist` |

### 端口职责

| 端口 | 服务 | 说明 |
|------|------|------|
| 1818 | Vite dev server | 前端开发服务器 |
| 8000 | PHP API 后端 | API 请求 |

**后台访问地址**：`http://localhost:1818/#/admin/login`（注意 `#`）
**禁止**用 `http://localhost:8000/admin/` 访问后台（返回 JSON 错误）

## BuildAdmin 核心配置（`config/buildadmin.php`）

| 配置项 | 说明 |
|--------|------|
| `open_member_center` | 是否开放前台会员中心（本项目设为 `false`） |
| `admin_sso` / `user_sso` | 单点登录开关 |
| `admin_token_keep_time` | 后台 token 有效期 |
| `user_token_keep_time` | 会员 token 有效期 |
| CORS 跨域配置 | `AllowCrossDomain` 中间件 |
| `click_captcha` | 验证码配置 |
| Token 存储 | MySQL 或 Redis |
| `proxy_server_ip` | 代理服务器 IP |
| CDN URL | 静态资源 CDN |
| 管理员日志 | 自动写入开关 |

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
- **不配置** `server.proxy`，API 跨域通过 `VITE_AXIOS_BASE_URL` + 后端 CORS 实现

## 插件注册顺序（`main.ts`）

1. `app.use(pinia)` — Pinia 最先注册（router guards 依赖 store）
2. `await loadLang(app)` — 异步加载语言包（必须在 router 之前完成）
3. `app.use(router)` — 路由
4. `app.use(ElementPlus)` — UI 组件库
5. `directives(app)` — 自定义指令
6. `registerIcons(app)` — 图标注册
7. `app.mount('#app')` — 挂载
8. `app.config.globalProperties.eventBus = mitt()` — 事件总线（挂载后赋值）
