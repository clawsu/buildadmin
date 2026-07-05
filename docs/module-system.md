# 模块系统规范

> BuildAdmin 有两套模块概念：**市场插件包**（可安装）和 **CRUD 代码生成器**（开发新功能）。

---

## 1. 市场插件包（可安装模块）

### 1.1 存储位置

- 安装目录：`modules/{uid}/`（动态创建，不入 git）
- 备份目录：`modules/backups/`

### 1.2 文件结构

```
modules/{uid}/
├── info.ini              # 元数据（必须）
├── config.json           # 依赖配置
├── install.sql           # 安装 SQL（__PREFIX__ 自动替换为 ba_）
├── {uid}.php             # 事件类
├── webBootstrap.stub     # 前端代码注入点
└── .runtime              # 运行时缓存（自动生成）
```

### 1.3 info.ini（必须）

```ini
uid = my_module
title = 我的模块
intro = 模块介绍
author = 作者
version = 1.0.0
state = 1
```

必须字段：`uid`、`title`、`intro`、`author`、`version`、`state`

### 1.4 config.json（依赖配置）

```json
{
    "require": {},
    "require-dev": {},
    "dependencies": {},
    "devDependencies": {},
    "nuxtDependencies": {},
    "nuxtDevDependencies": {},
    "composerConfig": {},
    "protectedFiles": []
}
```

- `require` / `require-dev`：composer 依赖
- `dependencies` / `devDependencies`：npm 依赖（web/）
- `protectedFiles`：受保护文件列表（禁止覆盖）

### 1.5 事件类（{uid}.php）

```php
<?php
namespace modules\my_module;

class Event {
    public function install() {
        // 安装后逻辑（SQL 导入后执行）
    }

    public function uninstall() {
        // 卸载前逻辑（清理数据）
    }

    public function enable() {
        // 启用后逻辑
    }

    public function disable() {
        // 禁用前逻辑
    }

    public function update() {
        // 更新后逻辑
    }
}
```

**命名空间**：`modules\{uid}`（`uid` 中的 `-` 会被转为 `_`）

**自动加载**：`composer.json` 已配置 PSR-4：
```json
"autoload": {
    "psr-4": {
        "modules\\": "modules"
    }
}
```

### 1.6 install.sql 规范

```sql
-- 用 __PREFIX__ 代替 ba_，安装时自动替换
CREATE TABLE `__PREFIX__xxx` (
    `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
    `title` varchar(255) NOT NULL DEFAULT '',
    `status` enum('0','1') NOT NULL DEFAULT '1',
    `createtime` bigint(20) NOT NULL DEFAULT 0,
    `updatetime` bigint(20) NOT NULL DEFAULT 0,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='XXX表';
```

### 1.7 WebBootstrap 前端注入

模块可通过 `webBootstrap.stub` 注入代码到前端：

**注入点**：
- `main.ts`：imports 和启动代码
- `App.vue`：imports 和 onMounted 代码

**标记格式**：
```ts
// Code from module 'my_module' start(mti)
import MyComponent from '/@/components/MyComponent.vue'
// Code from module 'my_module' end
```

### 1.8 模块状态

| 状态 | 值 | 说明 |
|------|-----|------|
| `UNINSTALLED` | 0 | 未安装 |
| `INSTALLED` | 1 | 已安装启用 |
| `WAIT_INSTALL` | 2 | 已下载待安装 |
| `CONFLICT_PENDING` | 3 | 有冲突待解决 |
| `DEPENDENT_WAIT_INSTALL` | 4 | 依赖待安装 |
| `DIRECTORY_OCCUPIED` | 5 | 目录被占用 |
| `DISABLE` | 6 | 已禁用 |

### 1.9 模块生命周期

```
下载/上传 ZIP
    ↓
解压到 modules/{uid}/
    ↓
安装前检查（版本、冲突）
    ↓
导入 install.sql（__PREFIX__ → ba_）
    ↓
执行 Event::install()
    ↓
安装 npm/composer 依赖
    ↓
注入 WebBootstrap 前端代码
    ↓
执行 Event::enable()
    ↓
模块启用
```

### 1.10 可覆盖目录

模块可以覆盖以下目录的文件：
`app`、`config`、`database`、`extend`、`modules`、`public`、`vendor`、`web`

覆盖前自动备份到 `modules/backups/`。

### 1.11 后台管理 UI

模块管理页面：`web/src/views/backend/module/`

API 接口：
- `GET /admin/module/index` — 列表
- `GET /admin/module/state` — 检查状态
- `POST /admin/module/install` — 安装
- `POST /admin/module/uninstall` — 卸载
- `POST /admin/module/changeState` — 启用/禁用
- `POST /admin/module/upload` — 上传本地 ZIP

### 1.12 关键后端文件

| 文件 | 职责 |
|------|------|
| `app/admin/controller/Module.php` | 控制器：列表、安装、卸载、状态切换、上传 |
| `app/admin/library/module/Manage.php` | 核心管理：下载、上传、安装、卸载、启用、禁用 |
| `app/admin/library/module/Server.php` | 服务层：预检、SQL 导入、冲突检测、依赖管理 |
| `app/common/service/moduleService.php` | 服务提供者：应用初始化时加载已安装模块 |
| `extend/ba/Depends.php` | 依赖管理：composer.json / package.json 增删 |
| `extend/ba/Filesystem.php` | 文件操作：zip/unzip、目录操作 |
| `extend/ba/Version.php` | 版本比较 |

---

## 2. CRUD 代码生成器

### 2.1 功能说明

通过 `/admin/crud/crud/generate` 接口，从数据库表定义自动生成完整的后台管理模块。

### 2.2 生成的文件

| 类型 | 路径 |
|------|------|
| Model | `app/admin/model/Xxx.php` |
| Controller | `app/admin/controller/{module}/Xxx.php` |
| Validate | `app/admin/validate/Xxx.php` |
| Vue index | `web/src/views/backend/{module}/xxx/index.vue` |
| Vue form | `web/src/views/backend/{module}/xxx/popupForm.vue` |
| 语言包英文 | `web/src/lang/backend/en/xxx.ts` |
| 语言包中文 | `web/src/lang/backend/zh-cn/xxx.ts` |

### 2.3 Stub 模板

```
app/admin/library/crud/stubs/
├── html/
│   ├── index.stub          # index.vue 模板
│   └── form.stub           # popupForm.vue 模板
├── mixins/
│   ├── controller/
│   │   ├── controller.stub
│   │   ├── initialize.stub
│   │   └── index.stub
│   ├── model/
│   │   ├── model.stub
│   │   ├── beforeInsert.stub
│   │   ├── afterInsert.stub
│   │   ├── belongsTo.stub
│   │   ├── getters/*.stub
│   │   └── setters/*.stub
│   └── validate/
│       └── validate.stub
└── backendEntrance.stub
```

占位符语法：`{%variableName%}`，由 `Helper::assembleStub()` 替换。

### 2.4 Helper 类

`app/admin/library/crud/Helper.php` 提供：

| 方法 | 功能 |
|------|------|
| `parseNameData()` | 解析 Model/Controller/Validate 文件路径 |
| `parseWebDirNameData()` | 解析 Vue 和语言文件路径 |
| `writeModelFile()` | 生成 Model 文件 |
| `writeControllerFile()` | 生成 Controller 文件 |
| `writeIndexFile()` | 生成 index.vue |
| `writeFormFile()` | 生成 popupForm.vue |
| `writeWebLangFile()` | 生成语言文件 |
| `createMenu()` | 创建 admin_rule 菜单 |
| `handleTableDesign()` | 通过 Phinx 建表/改表 |
| `parseTableColumns()` | 从 information_schema 读取表结构 |
| `analyseField()` | 数据库列类型映射到前端组件类型 |

---

## 3. 手动新增后台模块流程

### 3.1 标准四件套

| # | 文件 | 路径 |
|---|---|---|
| 1 | PHP 控制器 | `app/admin/controller/{module}/Xxx.php` |
| 2 | PHP 模型 | `app/admin/model/Xxx.php` |
| 3 | 数据库迁移 | `database/migrations/YYYYMMDDHHMMSS_xxx.php` |
| 4 | Vue 页面 | `web/src/views/backend/{module}/xxx/index.vue` + `popupForm.vue` |

### 3.2 开发顺序

1. **建表迁移** → `php think migrate:run` → SQL 确认表结构
2. **Model** → `php -l` 语法检查 → 确认 SoftDelete 路径
3. **Controller** → `php -l` 语法检查 → curl 测试 API 返回 `code=1`
4. **菜单初始化迁移** → 执行 → 确认 `ba_admin_rule` 有记录
5. **Vue 页面** → 创建文件 → 浏览器访问验证

### 3.3 菜单结构

```
menu_dir: {Module}管理 (name: {module})
  ├── menu: {Module}列表 (name: {module}/{module})
  │   ├── button: 查看 (name: {module}/{module}/index)
  │   ├── button: 添加 (name: {module}/{module}/add)
  │   ├── button: 编辑 (name: {module}/{module}/edit)
  │   ├── button: 删除 (name: {module}/{module}/del)
  │   └── button: 排序 (name: {module}/{module}/sortable)
  └── ...
```

### 3.4 菜单迁移模板

```php
<?php
use think\facade\Db;
use think\migration\Migrator;

class InitXxxMenus extends Migrator
{
    public function up(): void
    {
        Db::name('admin_rule')->where('name', 'like', 'xxx%')->delete();
        $now = time();
        Db::startTrans();
        try {
            // 1. 顶级目录
            $dirId = Db::name('admin_rule')->insertGetId([
                'pid' => 0, 'type' => 'menu_dir', 'title' => 'XXX管理',
                'name' => 'xxx', 'path' => 'xxx', 'icon' => 'fa fa-xxx',
                'menu_type' => null, 'component' => '', 'keepalive' => 0,
                'extend' => 'none', 'weigh' => 100, 'status' => 1,
                'update_time' => $now, 'create_time' => $now,
            ]);

            // 2. 子菜单
            $menuId = Db::name('admin_rule')->insertGetId([
                'pid' => $dirId, 'type' => 'menu', 'title' => 'XXX列表',
                'name' => 'xxx/yyy', 'path' => 'xxx/yyy', 'icon' => 'fa fa-list',
                'menu_type' => 'tab', 'component' => '/src/views/backend/xxx/yyy/index.vue',
                'keepalive' => 1, 'extend' => 'none', 'status' => 1,
                'update_time' => $now, 'create_time' => $now,
            ]);

            // 3. 按钮权限
            $buttons = ['index', 'add', 'edit', 'del', 'sortable'];
            foreach ($buttons as $btn) {
                Db::name('admin_rule')->insert([
                    'pid' => $menuId, 'type' => 'button', 'title' => $btn,
                    'name' => 'xxx/yyy/' . $btn, 'path' => '',
                    'menu_type' => null, 'component' => '', 'keepalive' => 0,
                    'extend' => 'none', 'status' => 1,
                    'update_time' => $now, 'create_time' => $now,
                ]);
            }

            Db::commit();
        } catch (\Throwable $e) {
            Db::rollback();
            throw $e;
        }
    }

    public function down(): void
    {
        Db::name('admin_rule')->where('name', 'like', 'xxx%')->delete();
    }
}
```

**注意**：
- `keepalive` 是 tinyint（`1`/`0`），不是字符串
- `title` 不能为空字符串
- `component` 路径格式：`/src/views/backend/{module}/{xxx}/index.vue`
