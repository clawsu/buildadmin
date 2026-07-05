# 后端架构

## ThinkPHP 多应用结构

```
app/
├── admin/          # 后台管理应用
├── api/            # 对外接口应用
└── common/         # 公共控制器/模型（禁止 URL 直接访问）
modules/            # 模块市场，支持一键安装
config/             # 全局配置（database.php 等）
database/           # 数据库迁移（Phinx）
extend/             # 扩展类库
```

- 命名空间：`app\` 和 `modules\`
- 数据库表前缀：`ba_`（在 `config/database.php` 配置）
- 自动写入时间戳字段：`create_time`、`update_time`
- CRUD 代码生成器会自动创建数据库表和控制器/模型代码
- 模块系统通过 `modules/` 目录支持，模块可自动安装依赖

## 路由与应用映射

- 默认应用：`api`
- 禁止 URL 访问的应用：`common`
- 路由配置：`config/route.php`
