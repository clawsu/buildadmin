# 快速上手与环境搭建

## 环境要求

| 序号 | 描述 | 要求 |
|------|------|------|
| 1 | PHP | >= 8.2.0（x64 架构） |
| 2 | MySQL | >= 5.7（需 InnoDB 引擎） |
| 3 | Node.js | >= 22.13.0 |
| 4 | npm/pnpm | >= 10.9.2 |
| 5 | Composer | Git 克隆包必需，完整包不需要 |

## 安装步骤

```bash
# 1. 克隆代码
git clone https://gitee.com/wonderful-code/buildadmin.git
cd buildadmin

# 2. 安装 PHP 依赖（Git 克隆包需要）
composer install

# 3. 启动安装服务
php think run
# 浏览器访问 http://127.0.0.1:8000/ 完成安装

# 4. 安装完成后，启动前端开发服务
cd web
npm install  # 或 pnpm install
npm run dev  # 或 pnpm dev

# 5. 浏览器访问 http://localhost:1818/#/admin
```

**重要**：开发期间不要创建 Nginx/Apache 网站，全程使用 `php think run`。

## 开发环境六步曲

1. 在本地 PC 安装好 BuildAdmin
2. 全程使用 `php think run` 启动服务（不使用 Nginx/Apache）
3. 确保 MySQL 服务已启动，数据库资料在 `config/database.php`
4. 在 `/web` 目录执行 `npm run dev`，访问 `localhost:1818`
5. 开启 TP 框架调试模式（将 `.env-example` 重命名为 `.env`）
6. 了解如何调试接口

## 停止服务

在命令行窗口按 `Ctrl+C` 即可停止。

## 常见问题

- **为什么用 php think run 而不是 Nginx？** 该命令可读取环境变量，实现 WEB 终端功能
- **每次改动都需要重新发布吗？** 错误。开发环境启动热更新服务即可，上线时才重新发布
- **前端工程化是什么？** 通过 Vite 等工具实现模块化、编译、热更新等现代前端开发能力
