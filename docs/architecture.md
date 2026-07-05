# 后端架构

## ThinkPHP 多应用结构

```
app/
├── admin/              # 后台管理应用
│   ├── controller/     # 控制器
│   │   ├── auth/       # 权限管理（Admin.php、Group.php、Rule.php）
│   │   ├── crud/       # CRUD 代码生成
│   │   ├── routine/    # 常规功能（Attachment.php、Config.php、Menu.php）
│   │   ├── security/   # 安全（DataRecycle.php、SensitiveData.php）
│   │   ├── user/       # 会员管理
│   │   ├── Index.php   # 后台初始化/登录
│   │   ├── Ajax.php    # 通用 AJAX 接口
│   │   ├── Dashboard.php
│   │   └── Module.php  # 模块管理
│   ├── model/          # Eloquent 风格模型
│   ├── validate/       # 验证器
│   ├── library/        # 核心库
│   │   ├── traits/Backend.php  # CRUD Trait（核心！）
│   │   ├── Auth.php            # 权限认证
│   │   └── stubs/              # 代码生成模板
│   └── lang/           # 语言包
├── api/                # 对外接口应用（前端会员端）
│   └── controller/     # Index、Account、User、Common、Install 等
├── common/             # 公共应用（禁止 URL 直接访问）
│   ├── controller/
│   │   ├── Backend.php     # 后台控制器基类
│   │   ├── Api.php         # API 控制器基类
│   │   └── Frontend.php    # 前台控制器基类
│   ├── model/              # 共享模型
│   ├── library/            # 共享库（Auth.php 等）
│   └── service/            # 服务层（moduleService.php 等）
├── BaseController.php  # 根抽象控制器
├── AppService.php
├── common.php          # 公共函数
├── event.php
├── ExceptionHandle.php
├── middleware.php
├── provider.php
├── Request.php
└── service.php
```

## 控制器继承链

```
app\BaseController (abstract)
  └── app\common\controller\Api
        └── app\common\controller\Backend (使用 app\admin\library\traits\Backend)
              └── app\admin\controller\{Module}\{Controller}
```

## 后台控制器模板

```php
<?php
namespace app\admin\controller\auth;

use app\common\controller\Backend;
use app\admin\model\Admin as AdminModel;

class Admin extends Backend {
    protected object $model;
    protected array|string $preExcludeFields = ['create_time', 'update_time', 'password', 'salt'];
    protected array|string $quickSearchField = ['username', 'nickname'];
    protected string|int|bool $dataLimit = 'allAuthAndOthers';
    protected string $dataLimitField = 'id';

    public function initialize(): void {
        parent::initialize();
        $this->model = new AdminModel();
    }

    // 仅在需要自定义逻辑时覆盖 CRUD 方法
    // 默认的 index()、add()、edit()、del() 已在 Backend trait 中实现
}
```

## Backend Trait 核心属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `$noNeedLogin` | array | 不需要登录的方法 |
| `$noNeedPermission` | array | 不需要权限验证的方法 |
| `$preExcludeFields` | array | 保存前排除的字段 |
| `$model` | object | ThinkPHP Model 实例 |
| `$weighField` | string | 排序权重字段（默认 `weigh`） |
| `$defaultSortField` | string | 默认排序字段 |
| `$quickSearchField` | array | 快速搜索字段 |
| `$dataLimit` | string\|bool | 数据权限（`false`/`personal`/`parent`/`allAuth`/`allAuthAndOthers`） |
| `$dataLimitField` | string | 数据权限关联字段 |
| `$withJoinTable` | array | 预加载关联 |

## Model 模板

```php
<?php
namespace app\admin\model;

use think\Model;
use think\model\concern\SoftDelete;

class Wallpaper extends Model {
    use SoftDelete;

    protected $autoWriteTimestamp = 'int';
    protected $createTime = 'createtime';
    protected $updateTime = 'updatetime';
    protected $deleteTime = 'deletetime';

    // 属性访问器
    public function getAvatarAttr($value): string {
        return full_url($value);
    }

    // 关联
    public function category(): \think\model\relation\BelongsTo {
        return $this->belongsTo(Category::class, 'category_id');
    }
}
```

**注意**：
- 时间字段是 `createtime`/`updatetime`/`deletetime`（**无下划线**），unix 时间戳
- `$autoWriteTimestamp = 'int'` 而非 `true`
- SoftDelete 路径是 `think\model\concern\SoftDelete`（非 `traits\model\SoftDelete`）
- 使用 SoftDelete 前必须确认表有 `deletetime` 字段，否则报 `Unknown column` 错误

## 后端响应格式

```php
$this->success('操作成功', ['key' => 'value']);   // code: 1
$this->error('操作失败');                         // code: 0
$this->success('', ['list' => [...], 'total' => N, 'remark' => '']);
```

## API URL 约定

- 后台：`/admin/{Controller}.{action}`（如 `/admin/user.User/index`）
- 前台：`/api/{controller}/{action}`（如 `/api/index/index`）
- 嵌套控制器用 `.` 分隔目录：`/admin/wallpaper.Wallpaper/index`（非 `/admin/wallpaper/wallpaper/index`）

## 路由与应用映射

- 默认应用：`api`
- 禁止 URL 访问的应用：`common`
- 路由配置：`config/route.php`
- 数据库表前缀：`ba_`（在 `config/database.php` 配置）
- 自动写入时间戳字段：`createtime`、`updatetime`

## 数据库迁移

- 使用 Phinx 迁移：`php think migrate:create <name>`
- 执行迁移：`php think migrate:run`
- 迁移文件位于 `database/migrations/`
- **建表**可用 `$this->table('xxx')->create()`（自动加 `ba_` 前缀）
- **索引操作**必须用 `$this->execute()` 执行原始 SQL（Phinx 的 `update()` 不加前缀）
- 迁移中原始 SQL 表名必须带 `ba_` 前缀

## 模块系统

- 模块通过 `modules/` 目录支持
- 模块可自动安装依赖（`package.json` 和 `composer.json`）
- CRUD 代码生成器会自动创建数据库表和控制器/模型代码
