# 功能完成验收清单（硬性约束）

> 本清单是"功能完成"的硬性定义。
> **任何声称"已完成"的任务，必须逐项确认本清单全部通过。**
> 违反此清单的"完成"声明等同于欺骗。

---

## 通用规则

### C1.1 完成的定义

**MUST**: "功能完成"= 实现完整 + 验证通过 + 已提交。
**FORBIDDEN**: 只写代码不验证就声称完成。
**FORBIDDEN**: 只完成 API 层就声称整个功能完成（见 [admin-module.md](admin-module.md#a11-完整定义) 四件套）。
**FORBIDDEN**: 用"理论上应该可以"代替实际测试。

### C1.2 反懒惰协议

**MUST**: 运行验证命令，不凭肉眼判断代码正确性。
**MUST**: 遇到错误必须修复或记录原因，不可跳过。
**MUST**: 同一问题失败 3 次必须停下来重新分析根因，不可重复尝试。

---

## 后端 API 验收清单

### C2.1 PHP 语法检查

**MUST**: 所有新增/修改的 PHP 文件通过 `php -l` 检查。

```bash
cd buildadmin
php -l app/admin/controller/{module}/Xxx.php
php -l app/admin/model/Xxx.php
php -l app/api/controller/XxxController.php
# 输出必须是 "No syntax errors detected"
```

### C2.2 Trait 路径检查

**MUST**: Model 文件中不出现 `traits\model\SoftDelete`。

```bash
grep -rn "traits\\\\model\\\\SoftDelete" app/
# 必须无输出
```

### C2.3 数据库迁移验证

**MUST**: 迁移执行成功，表结构符合预期。

```bash
php think migrate:run
# 然后验证表存在
php -r "require 'vendor/autoload.php'; (new think\App())->initialize(); var_dump(\think\facade\Db::query('SHOW TABLES LIKE \"ba_xxx\"'));"
```

### C2.4 API 接口测试

**MUST**: 每个新增 API 接口用 curl 测试，返回 `code=1`。

```bash
# 小程序 API
curl -s http://localhost:8000/api/xxx/yyy | python3 -m json.tool
# 必须包含 "code": 1

# 后台 API（需登录获取 token）
LOGIN=$(curl -s -X POST http://localhost:8000/admin/Index/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"Admin123","keep":false}')
TOKEN=$(echo "$LOGIN" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['userInfo']['token'])")
curl -s http://localhost:8000/admin/{module}.Xxx/index -H "ba-token: $TOKEN" | python3 -m json.tool
# 必须包含 "code": 1
```

---

## 后台模块验收清单

### C3.1 菜单数据验证

**MUST**: `ba_admin_rule` 表有对应菜单记录。

```bash
php -r "require 'vendor/autoload.php'; (new think\App())->initialize(); 
\$count = \think\facade\Db::name('admin_rule')->where('name','like','{module}/xxx%')->count();
echo 'count=' . \$count . PHP_EOL;
exit(\$count > 0 ? 0 : 1);"
```

### C3.2 菜单 title 非空

**MUST**: 所有菜单记录的 `title` 字段非空。

```bash
php -r "require 'vendor/autoload.php'; (new think\App())->initialize(); 
\$empty = \think\facade\Db::name('admin_rule')->where('name','like','{module}%')->where('title','')->count();
echo 'empty_title_count=' . \$empty . PHP_EOL;
exit(\$empty === 0 ? 0 : 1);"
```

**根因**: 之前有两条菜单记录 title 为空，导致前端 Vue 渲染不出菜单项。

### C3.3 登录后菜单返回

**MUST**: 登录后台 `/admin/Index/index` 接口返回的 menus 中包含新模块。

### C3.4 Vue 文件存在

**MUST**: `index.vue` 和 `popupForm.vue` 都存在。

```bash
ls -la web/src/views/backend/{module}/xxx/index.vue
ls -la web/src/views/backend/{module}/xxx/popupForm.vue
# 两个文件都必须存在
```

### C3.5 defineOptions name 一致性

**MUST**: Vue 文件中的 `defineOptions({ name: 'xxx' })` 与数据库菜单 `name` 字段完全一致。

```bash
# 检查 Vue 文件中的 name
grep "defineOptions" web/src/views/backend/{module}/xxx/index.vue
# 检查数据库菜单 name
php -r "... Db::name('admin_rule')->where('name','like','{module}/xxx%')->select() ..."
# 两者必须匹配
```

---

## 前端验证清单

### C4.1 Vite 进程运行

**MUST**: Vite dev server 在 1818 端口运行。

```bash
lsof -nP -iTCP:1818 -sTCP:LISTEN
# 必须有 node 进程监听
```

### C4.2 前端可访问

**MUST**: `http://localhost:1818/` 返回 HTML（包含 `<div id="app">`）。

```bash
curl -s http://localhost:1818/ | grep '<div id="app">'
# 必须有输出
```

### C4.3 后台登录页可访问

**MUST**: `http://localhost:1818/#/admin/login` 能加载登录表单（人工验证）。

---

## 提交前最终检查

### C5.1 Git 状态

**MUST**: 所有新增/修改文件已 `git add`。

### C5.2 无遗留测试文件

**MUST**: 项目根目录无 `test_*.php`、`verify_*.py` 等临时测试文件。

```bash
ls test_*.php verify_*.py 2>/dev/null
# 必须无输出（或文件已删除）

---

## 常见错误对照表

| 错误现象 | 根因 | 规则 | 验证方法 |
|---|---|---|---|
| `Trait not found` | trait 路径错 | [R1.1](backend-conventions.md#r11-softdelete-trait-路径) | `grep -rn "traits\\\\model" app/` |
| `Unknown column deletetime` | 表无字段却用 SoftDelete | [R1.2](backend-conventions.md#r12-softdelete-使用前提) | 对照迁移文件 |
| `Table 'xxx' doesn't exist` | Phinx 不加 ba_ 前缀 | [R2.1](backend-conventions.md#r21-索引操作必须用原始-sql) | 用 `execute()` |
| 后台返回 `code:303` | 未登录或 token 错 | [R4.2](backend-conventions.md#r42-后台-token-传递) | 用 `ba-token` header |
| 登录页不显示 | URL 缺 `#` | [F1.2](frontend-conventions.md#f12-后台访问-url) | 用 `/#/admin/login` |
| 菜单不显示 | 未初始化菜单数据 | [A2.1](admin-module.md#a21-菜单必须由迁移初始化) | 查 `ba_admin_rule` 表 |
| Vue 页面 404 | component 路径错 | [A2.3](admin-module.md#a23-菜单字段规范) | 检查 `component` 字段 |
| `Incorrect integer value` | keepalive 填字符串 | [A2.3](admin-module.md#a23-菜单字段规范) | 用 `1`/`0` |
| 控制器不存在 | URL 用 `/` 分隔 | [R3.4](backend-conventions.md#r34-后台控制器路由分隔符) | 用 `.` 分隔 |
| 用户被误判禁用 | status 语义不一致 | [R1.4](backend-conventions.md#r14-状态字段值语义) | 用 `enable/disable` |
