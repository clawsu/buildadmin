# 后端代码规范（硬性约束）

> 本文档记录后端 PHP 代码的强制规则。每条规则附"根因 + 正例 + 反例"。
> 违反任一规则视为功能未完成。

---

## 1. Model 规范

### R1.1 SoftDelete trait 路径

**MUST**: 使用 `think\model\concern\SoftDelete`
**FORBIDDEN**: 使用 `traits\model\SoftDelete`

**根因**: ThinkPHP 8 已将 SoftDelete 迁移到 `think\model\concern` 命名空间，旧路径 `traits\model\SoftDelete` 不存在，会导致 `Trait not found` 致命错误。

```php
// ✅ 正例
use think\model\concern\SoftDelete;

class Wallpaper extends Model
{
    use SoftDelete;
    protected $deleteTime = 'deletetime';
}

// ❌ 反例
use traits\model\SoftDelete;  // ThinkPHP 8 中不存在
```

### R1.2 SoftDelete 使用前提

**MUST**: 使用 `use SoftDelete` 前，必须确认对应数据库表有 `deletetime` 字段。
**FORBIDDEN**: 对没有 `deletetime` 字段的表使用 SoftDelete。

**根因**: SoftDelete 会在查询时自动加 `WHERE deletetime IS NULL`，如果表没有该字段会触发 `Unknown column` SQL 错误。

```php
// ✅ 正例：banners 表无 deletetime 字段
class Banner extends Model
{
    // 不 use SoftDelete
    protected $name = 'banners';
    protected $autoWriteTimestamp = 'int';
    protected $createTime = 'createtime';
    protected $updateTime = 'updatetime';
    // 不设 $deleteTime
}

// ❌ 反例：表无 deletetime 字段却用了 SoftDelete
class Banner extends Model
{
    use SoftDelete;  // 查询时报 Unknown column 'banner.deletetime'
}
```

**字段对照表**（哪些表有 deletetime）：

| 表 | deletetime | 软删除 |
|---|---|---|
| users | ✅ | ✅ |
| categories | ✅ | ✅ |
| messages | ✅ | ✅ |
| banners | ❌ | ❌ |
| adverts | ❌ | ❌ |
| tags | ❌ | ❌ |
| colors | ❌ | ❌ |
| settings | ❌ | ❌ |

### R1.3 跨模块 Model 引用

**MUST**: 在 `app\admin\` 命名空间内引用业务模型时，用 `app\admin\model\X`，不用 `app\model\X`。
**MUST**: 在 `app\api\` 或 `app\common\` 命名空间内引用用户模型时，用 `app\model\User`（共享模型位于 `app/model/`）。

**根因**: admin 模块有独立的模型副本（含 SoftDelete、关联关系等后台专用逻辑），api 模块用共享模型。混用会导致字段缺失或 trait 重复。

```php
// ✅ 正例：admin 控制器/模型内
namespace app\admin\controller\{module};
use app\admin\model\{Module};       // 后台版本
use app\admin\model\Category;
use app\model\User;                  // 用户表共享，只有一份

// ✅ 正例：api 控制器内
namespace app\api\controller;
use app\model\User;                  // 共享版本
// 不直接 use app\admin\model\* 

// ❌ 反例：admin 内引用错命名空间
use app\model\{Module};  // 不存在或字段不全
```

### R1.4 状态字段值语义

**MUST**: 用户表 `ba_user.status` 使用 `enable` / `disable`，对齐 BuildAdmin 框架语义。
**MUST**: 业务表（users/categories/tags 等）的 `status` 字段按各自迁移定义的 enum 值（通常是 `'0'`/`'1'`）。
**FORBIDDEN**: 在 user 表用 `normal` / 其他自定义值。

**根因**: BuildAdmin 的 `Auth` 类（`app/common/library/Auth.php`）和 `Backend` 基类检查 `status` 时用 `enable/disable`。用其他值会导致登录后 `verifyToken` 误判用户已禁用。

```php
// ✅ 正例：User 模型创建用户
User::create([
    'openid' => $openid,
    'status' => 'enable',  // 对齐 BuildAdmin
]);

// ✅ 正例：AuthService 校验
if (!$user || $user->status === 'disable') {
    return null;  // 只拒绝禁用用户
}

// ❌ 反例
User::create(['status' => 'normal']);              // 框架不认
if ($user->status !== 'normal') { return null; }   // 老用户全部被拒
```

### R1.5 时间字段

**MUST**: `createtime` / `updatetime` / `deletetime` 字段类型为 `biginteger`（unix 时间戳，非 datetime）。
**MUST**: 模型中 `$autoWriteTimestamp = 'int'`，`$createTime = 'createtime'`，`$updateTime = 'updatetime'`。

**根因**: 项目历史约定用 unix 时间戳，避免时区问题，查询性能更好。

---

## 2. Migration 规范

### R2.1 索引操作必须用原始 SQL

**MUST**: 创建/修改索引时用 `$this->execute()` 执行原始 SQL，表名带 `ba_` 前缀。
**FORBIDDEN**: 用 `$this->table('xxx')->addIndex()->update()` 修改已存在表的索引。

**根因**: Phinx 的 `$table->update()` 不会自动应用 `ba_` 表前缀，导致 `Table 'app_db.messages' doesn't exist` 错误（实际表名是 `ba_messages`）。建表时 `create()` 会加前缀，但 `update()` 不会。

```php
// ✅ 正例：原始 SQL
public function up(): void
{
    $exists = $this->fetchRow(
        "SHOW INDEX FROM `ba_messages` WHERE Key_name = 'receiver_id_is_read'"
    );
    if (empty($exists)) {
        $this->execute(
            "CREATE INDEX `receiver_id_is_read` ON `ba_messages` (`receiver_id`, `is_read`)"
        );
    }
}

// ❌ 反例：table API（不加前缀）
public function up(): void
{
    $this->table('messages')  // 实际找的是 messages 而非 ba_messages
         ->addIndex(['receiver_id', 'is_read'])
         ->update();  // 报错：Table 'messages' doesn't exist
}
```

### R2.2 建表可用 table API

**MUST**: 首次建表用 `$this->table('xxx', [...])->addColumn(...)->create()`，此时 Phinx 会自动加 `ba_` 前缀。

```php
// ✅ 正例：建表（create 自动加前缀）
public function up(): void
{
    if (!$this->hasTable('users')) {
        $this->table('users', ['comment' => '用户表'])
            ->addColumn('title', 'string', ['limit' => 255])
            ->addColumn('status', 'enum', ['values' => '0,1,2,3', 'default' => '1'])
            ->create();
    }
}
```

### R2.3 迁移文件命名

**MUST**: 文件名格式 `YYYYMMDDHHMMSS_驼峰名.php`，类名与文件名相同（驼峰）。
**MUST**: 每个迁移有 `up()` 和 `down()` 方法（down 可为空但必须存在）。

### R2.4 Model 与表结构对齐

**MUST**: 写 Model 前必须读对应迁移文件，确认：
- 表是否有 `deletetime` 字段（决定是否用 SoftDelete）
- 字段类型（enum/string/int）
- 字段名拼写（createtime 非 create_time）

**根因**: 之前 Banner/Advert 模型用了 SoftDelete 但表无 deletetime 字段，导致运行时 SQL 错误。

---

## 3. Controller 规范

### R3.1 继承基类

**MUST**: 
- `app\api\controller\*` 继承 `app\common\controller\Api`（小程序 API）
- `app\admin\controller\*` 继承 `app\common\controller\Backend`（后台管理）
- `app\admin\controller\{module}\*` 同样继承 `Backend`

### R3.2 后台控制器 noNeedLogin 声明

**MUST**: 后台控制器中无需登录的方法必须在 `$noNeedLogin` 数组中显式声明。
**MUST**: 登录方法本身必须在 `$noNeedLogin` 中。

**根因**: `Backend::initialize()` 会检查 `noNeedLogin`，未声明的方法默认需要登录，导致登录接口本身返回"请先登录"。

```php
// ✅ 正例：Index 控制器
class Index extends Backend
{
    protected array $noNeedLogin = ['logout', 'login'];
    protected array $noNeedPermission = ['logout', 'login'];
    
    public function login(): void { /* ... */ }
}

// ❌ 反例：未声明 noNeedLogin
class Admin extends Backend
{
    // 没有 $noNeedLogin，所有方法都要求登录
    public function login(): void { /* 永远返回 303 */ }
}
```

### R3.3 API 返回格式

**MUST**: 统一返回 `{ code: 1/0, msg: string, time: int, data: mixed }`。
**MUST**: `code = 1` 成功，`code = 0` 失败。
**MUST**: 分页返回 `data: { list: array, total: int }`。

```php
// ✅ 正例
$this->success('操作成功', ['list' => $list, 'total' => $total]);
$this->error('参数错误');
```

### R3.4 后台控制器路由分隔符

**MUST**: ThinkPHP 嵌套控制器 URL 用 `.` 分隔目录，如 `/admin/{module}.{Module}/index`。
**FORBIDDEN**: 用 `/` 分隔（如 `/admin/{module}/{module}/index` 会报控制器不存在）。

**根因**: ThinkPHP 多应用控制器路由规则：`应用名/目录1.目录2.控制器/方法`。

```php
// ✅ 正例：URL
/admin/{module}.{Module}/index   // app\admin\controller\{module}\{Module}::index
/admin/auth.Group/index            // app\admin\controller\auth\Group::index

// ❌ 反例
/admin/{module}/{module}/index   // 报：控制器不存在 app\admin\controller\{Module}
```

---

## 4. Auth 规范

### R4.1 双认证系统

**MUST**: 明确区分两套认证系统，不可混用：

| 系统 | 用途 | 类 | Token 传递 | 登录接口 |
|---|---|---|---|---|
| BuildAdmin Auth | 后台管理 | `app\common\library\Auth` | `ba-token` header | `/admin/Index/login` |
| AuthService | 小程序 | `app\common\service\AuthService` | `Authorization: Bearer` | `/api/auth/login` |

**根因**: 之前误用 `Authorization: Bearer` 访问后台接口导致 303 错误；误用 `/admin/auth.admin/login`（不存在）导致登录失败。

### R4.2 后台 Token 传递

**MUST**: 后台 API 请求通过 `ba-token` HTTP header 传递 token。
**FORBIDDEN**: 用 `Authorization: Bearer <token>` 访问后台 API。

**根因**: BuildAdmin 的 `get_auth_token()` 函数读取 `ba-token`/`batoken`/`ba_token`，不读 `Authorization`。

```php
// ✅ 正例
curl -H "ba-token: xxx-xxx-xxx" http://localhost:8000/admin/{module}.{Module}/index

// ❌ 反例
curl -H "Authorization: Bearer xxx" http://localhost:8000/admin/...  // 返回 303
```

### R4.3 小程序 Token 传递

**MUST**: 小程序 API 请求通过 `Authorization: Bearer <token>` header 传递 token。
**MUST**: `AuthMiddleware` 检查 `Authorization` header。

### R4.4 禁用前台会员中心

**MUST**: `config/buildadmin.php` 中 `open_member_center = false`。
**根因**: 本项目只需小程序 + 后台管理，不需要 H5 会员中心。小程序用户通过微信登录创建，无密码，访问会员中心会报"账户不存在"。

---

## 5. 数据库规范

### R5.1 表前缀

**MUST**: 所有业务表使用 `ba_` 前缀。
**MUST**: `Db::name('xxx')` 自动加前缀，`Db::table('ba_xxx')` 需手动加。
**MUST**: 迁移中原始 SQL 的表名必须带 `ba_` 前缀。

### R5.2 字段类型

**MUST**: 状态字段用 `enum` 类型，字符串值（如 `'0'`/`'1'`），不用整数。
**MUST**: 时间字段用 `biginteger`（unix 时间戳）。
**MUST**: 主键 `id` 用 `integer unsigned`，`signed: false`。

### R5.3 索引

**MUST**: 外键字段加索引（category_id、user_id、{module}_id 等）。
**MUST**: 常用查询组合加复合索引。
**MUST**: 唯一字段加唯一索引（如 tags.name、user.openid）。
