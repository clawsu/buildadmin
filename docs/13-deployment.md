# 部署

## 后台部署

### 部署步骤

1. 使用 git 管理代码，检查 `.gitignore` 中的部署规则
2. 删除 `/install` 目录
3. 线上可不上传 `/web` 目录，只同步 `重新发布` 后的 `/public/assets/` 和 `/public/index.html`
4. 使用 Nginx/Apache 运行站点（不再使用 `php think run`）
5. 站点根目录：`buildadmin`，运行目录：`buildadmin/public`
6. 配置 ThinkPHP URL 重写规则
7. 同步数据库

### URL 重写规则

**Nginx**：

```nginx
if (!-e $request_filename) {
    rewrite ^(.*)$ /index.php?s=/$1 last;
    break;
}
```

**Apache**（`.htaccess` 放在 `public/` 目录）：

```apache
<IfModule mod_rewrite.c>
    Options +FollowSymlinks -Multiviews
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ index.php?s=/$1 [QSA,PT,L]
</IfModule>
```

## Web 工程独立部署

1. 修改 `/web/.env.production` 中的 `VITE_AXIOS_BASE_URL`
2. 服务端在 `config/buildadmin.php` 中允许前端域名跨域
3. 执行 `pnpm build`，将 `dist` 部署为静态站点

## WebNuxt 部署

### 快速预览

```bash
cd web-nuxt
pnpm build
node .output/server/index.mjs
# 访问 http://localhost:3000
```

### PM2 部署

```js
// ecosystem.config.cjs
module.exports = {
    apps: [{
        name: 'BANuxt-3000',
        port: '3000',
        exec_mode: 'cluster',
        instances: 'max',
        script: './.output/server/index.mjs',
    }],
}
```

```bash
pnpm start  # 启动 PM2
```

### Nginx 反向代理

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 客户端真实 IP

1. Nginx 配置 `proxy_set_header X-Real-IP $remote_addr`
2. 后端 `config/buildadmin.php` 中设置 `proxy_server_ip` 为 Nuxt 服务器 IP

## 常见问题

- **PM2 报错 `require() of ES Module`**：将 `ecosystem.config.js` 改名为 `.cjs`
- **宝塔面板**：添加 Node 项目，启动选项选 `start`，端口 3000
