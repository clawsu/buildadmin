# 后台模块开发规范（硬性约束）

> 本文档记录新增后台业务模块的强制规则。
> 之前的多次错误都源于"只完成 API 就声称功能完成"，导致后台管理界面缺失。

---

## 1. 后台模块四件套

### A1.1 完整定义

**MUST**: 新增后台业务模块时，必须同时完成以下 4 件套，缺一不可：

| # | 文件 | 路径 | 作用 |
|---|---|---|---|
| 1 | PHP 控制器 | `app/admin/controller/{module}/Xxx.php` | 处理 CRUD 请求 |
| 2 | PHP 模型 | `app/admin/model/Xxx.php` | 数据模型（含 SoftDelete、关联） |
| 3 | 数据库迁移 | `database/migrations/YYYYMMDDHHMMSS_xxx.php` | 建表 + **初始化菜单** |
| 4 | Vue 页面 | `web/src/views/backend/{module}/xxx/index.vue` + `popupForm.vue` | 后台界面 |

**FORBIDDEN**: 只完成 1-2 件就声称"功能完成"或"已交付"。
**FORBIDDEN**: 把"API 写完"等同于"功能完成"。

**根因**: 只完成 API 控制器就声称功能完成，菜单数据未初始化、Vue 页面未开发，导致用户登录后台后看不到管理界面。

---

## 2. 菜单初始化规范

### A2.1 菜单必须由迁移初始化

**MUST**: 每个后台模块的菜单记录必须通过数据库迁移插入 `ba_admin_rule` 表。
**FORBIDDEN**: 手动用 SQL 客户端插入菜单（不可复现、不可版本控制）。
**FORBIDDEN**: 假设菜单"会自动出现"。

**根因**: 迁移文件只建了业务表，没有初始化菜单，导致后台登录后左侧没有管理菜单。

### A2.2 菜单结构

**MUST**: 菜单层级为 `menu_dir > menu > button`，三层结构。

```
menu_dir: {Module}管理 (name: {module})
  ├── menu: {Module}列表 (name: {module}/{module})
  │   ├── button: 查看 (name: {module}/{module}/index)
  │   ├── button: 添加 (name: {module}/{module}/add)
  │   ├── button: 编辑 (name: {module}/{module}/edit)
  │   ├── button: 删除 (name: {module}/{module}/del)
  │   └── button: 排序 (name: {module}/{module}/sortable)
  ├── menu: 分类管理 (name: {module}/category)
  │   └── ... 同上
  └── ...
```

### A2.3 菜单字段规范

**MUST**: `menu` 类型记录的字段填写规则：

| 字段 | 值 | 说明 |
|---|---|---|
| `pid` | 父目录 ID | 顶级目录用 0 |
| `type` | `menu_dir` / `menu` / `button` | 三选一 |
| `title` | 中文名称 | 不能为空 |
| `name` | `{module}/xxx` | 与 Vue defineOptions name 一致 |
| `path` | `{module}/xxx` | 路由路径 |
| `icon` | `fa fa-xxx` | FontAwesome 图标 |
| `menu_type` | `tab`（menu 类型） / `null`（button 类型） | |
| `component` | `/src/views/backend/{module}/xxx/index.vue` | Vue 文件路径 |
| `keepalive` | `1`（menu） / `0`（button） | **tinyint，不是字符串** |
| `extend` | `none` | |
| `status` | `1` | 启用 |

**FORBIDDEN**: `keepalive` 字段填字符串（如菜单 name），它是 tinyint。
**FORBIDDEN**: `title` 字段为空字符串。

**根因**: `keepalive` 填了字符串而非 tinyint，导致 `Incorrect integer value` SQL 错误，事务回滚。

### A2.4 迁移文件模板

```php
<?php
use think\facade\Db;
use think\migration\Migrator;

class InitXxxMenus extends Migrator
{
    public function up(): void
    {
        // 清理旧脏数据
        Db::name('admin_rule')->where('name', 'like', '{module}/xxx%')->delete();

        $now = time();
        Db::startTrans();
        try {
            // 1. 顶级目录（如果是新目录）
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
            $buttons = [
                ['title' => '查看', 'name' => 'index'],
                ['title' => '添加', 'name' => 'add'],
                ['title' => '编辑', 'name' => 'edit'],
                ['title' => '删除', 'name' => 'del'],
                ['title' => '排序', 'name' => 'sortable'],
            ];
            foreach ($buttons as $btn) {
                Db::name('admin_rule')->insert([
                    'pid' => $menuId, 'type' => 'button', 'title' => $btn['title'],
                    'name' => 'xxx/yyy/' . $btn['name'], 'path' => '',
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

### A2.5 菜单验证

**MUST**: 迁移执行后，必须验证：

```bash
# 1. 数据库记录数
php -r "require 'vendor/autoload.php'; (new think\App())->initialize(); echo \think\facade\Db::name('admin_rule')->where('name','like','{module}%')->count();"

# 2. 登录后台 API 返回菜单
LOGIN=$(curl -s -X POST http://localhost:8000/admin/Index/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"Admin123","keep":false}')
TOKEN=$(echo "$LOGIN" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['userInfo']['token'])")
curl -s http://localhost:8000/admin/Index/index -H "ba-token: $TOKEN" | python3 -m json.tool
```

---

## 3. 后台登录接口

### A3.1 正确的登录接口

**MUST**: 后台登录接口是 `/admin/Index/login`（`app\admin\controller\Index::login`）。
**FORBIDDEN**: 用 `/admin/auth.admin/login`（该方法不存在，返回 303）。

**根因**: 之前测试用 `/admin/auth.admin/login` 始终返回"请先登录"，因为 `auth.Admin` 控制器的 `login` 方法不存在，且未在 `noNeedLogin` 中声明。

```bash
# ✅ 正例
curl -X POST http://localhost:8000/admin/Index/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"Admin123"}'

# ❌ 反例
curl -X POST http://localhost:8000/admin/auth.admin/login \  # 303
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"Admin123"}'
```

### A3.2 默认账号

| 系统 | 用户名 | 密码 | 地址 |
|---|---|---|---|
| 后台管理 | admin | Admin123 | `http://localhost:1818/#/admin/login` |
| 小程序 | 无 | 无 | 通过微信 code 登录 `/api/auth/login` |

---

## 4. 新增模块检查清单

新增模块时对照此清单，确保不遗漏：

- [ ] 迁移文件建表（含 deletetime 字段，如需软删除）
- [ ] 迁移文件初始化菜单（menu_dir + menu + 5 buttons）
- [ ] Model 文件（trait 路径正确、SoftDelete 与表结构对齐）
- [ ] Controller 文件（继承 Backend、声明 noNeedLogin）
- [ ] Vue index.vue（defineOptions name 与菜单一致）
- [ ] Vue popupForm.vue
- [ ] curl 测试 API 返回 code=1
- [ ] 登录后台看到菜单
