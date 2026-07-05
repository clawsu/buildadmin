# 常见问题与调试

## 接口提示「请求地址出错:xxxx」

**条件**：非开发环境，浏览器控制台状态码 404

**解决**：
- 宝塔面板：网站 -> 伪静态 -> 选择 ThinkPHP 规则
- LNMP：参考 lnmp 官网文档配置伪静态
- 配置 Nginx/Apache/IIS 的 URL 重写规则

## 后台菜单跳转失败/路由无效

**CRUD 生成的菜单**：
- 开发环境生成 CRUD，开发完毕后再部署
- 生产环境先在 WEB 终端点击「重新发布」

**自建菜单**：
1. 检查组件路径是否正确
2. 后台组件放 `/src/views/backend/`
3. 前台组件放 `/src/views/frontend/`

## 请求跨域错误

**开发环境**：
1. 使用 `localhost:1818` 访问（不要用 IP）
2. 无需修改 `web/.env.development`
3. 检查跨域中间件配置

**生产环境**：
- web 和 server 同站则无此问题
- 独立部署需在 `config/buildadmin.php` 配置跨域域名

## 禁用模块报错

提示 `ENOENT: no such file or directory`：禁用模块时会自动删除文件，刷新浏览器即可。

## 调试接口

直接访问接口的三种方式：

```bash
# 方式一：URL 加参
http://localhost:8000/admin/index/index?server=1

# 方式二：请求加 header
# header: server=1

# 方式三：URL 加 index.php
http://localhost:8000/index.php/admin/index/index
```

## 开启调试模式

将 `.env-example` 重命名为 `.env`，内容：

```bash
APP_DEBUG = true
```

**注意**：请勿在生产环境长期开启调试模式。

## 调试技巧

1. 开启调试后，浏览器 F12 -> Network -> 找 500 请求 -> 查看错误详情
2. 错误信息包含：错误消息、错误文件、错误行号
3. 询问他人时建议直接截图报错信息

## WEB 终端常见问题

- **命令不存在**：检查站点域名是否正确，Linux 尝试 `sudo php think run`
- **权限不足**：检查目录权限和用户组，Linux 尝试 `sudo php think run`
- **终端无法连接**：检查站点目录权限和用户组

## 安装常见问题

- **PHP 函数被禁用**：在 `php.ini` 中解除函数禁用
- **composer 不存在**：尝试 `composer.phar install`，或检查环境变量
- **依赖安装失败**：检查报错下方的 Problem，逐个解决
