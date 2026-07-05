# 后端开发规范

## 控制器

### 继承关系

```
BaseController (根抽象控制器)
  └── Api (公共基类：语言包加载、数据库连接检查、IP黑名单、快捷响应)
        ├── Backend (后台基类：token 鉴权、SQL 构建、Trait 功能)
        └── Frontend (前台基类：会员 token 鉴权)
```

### 控制器命名

- 后台控制器继承 `app\common\controller\Backend`
- 前台控制器继承 `app\common\controller\Frontend`
- 在 `initialize()` 方法中初始化 Model

### 快捷响应方法

```php
// 操作成功（code=1）
$this->success('操作成功~');
$this->success('操作成功~', ['key' => 'value']);

// 操作失败（code=0）
$this->error('操作失败~');

// 返回数据（基础方法）
$this->result($msg, $data, $code);
```

**注意**：快捷方法内部抛出 `HttpResponseException`，无需 `return`，但不能在 `try` 块内使用。

### 前台基类属性

```php
protected array $noNeedLogin = [];       // 无需登录的方法
protected array $noNeedPermission = [];  // 无需鉴权的方法
protected Auth $auth;                     // 权限类实例
```

### 权限类常用方法

```php
$this->auth->isLogin();          // 是否登录
$this->auth->getUser();          // 获取会员模型
$this->auth->getAdmin();         // 获取管理员模型
$this->auth->getToken();         // 获取登录 token
$this->auth->getInfo();          // 获取管理员信息（仅允许输出字段）
$this->auth->isSuperAdmin();     // 是否超管
```

## 数据表设计规范

### 字段类型与 CRUD 对应

| 类型 | CRUD 自动生成 |
|------|--------------|
| enum | 单选框 |
| set | 复选框 |
| date/year/time/datetime/timestamp | 对应选择组件 |
| decimal/double/float | Number 输入框（步长根据默认值） |
| int/bigint/mediumint/smallint/tinyint | Number 输入框（步长 1） |
| longtext/text/mediumtext 等 | textarea |

### 特殊字段名

| 字段名 | 用途 |
|--------|------|
| `weigh` | 权重排序字段（int） |
| `create_time` | 创建时间（bigint，自动维护） |
| `update_time` | 更新时间（bigint，自动维护） |

### 字段后缀约定

| 后缀 | 示例 | 生成组件 |
|------|------|----------|
| `_id` | `user_id` | 关联表远程 select（单选） |
| `_ids` | `user_ids` | 关联表远程 select（多选） |
| `image/avatar` | `desc_image` | 上传图片（单图） |
| `images/avatars` | `descimages` | 上传图片（多图） |
| `file` | `attachfile` | 上传文件（单文件） |
| `files` | `attachfiles` | 上传文件（多文件） |
| `icon` | `icon` | 图标选择器 |
| `status/state/type` | `status` | 单选框 |
| `switch/toggle` | `log_switch` | 开关组件 |
| `content/editor` | `content` | 富文本编辑器 |
| `textarea/multiline/rows` | `rows` | Textarea |

### 字段注释（字典）

```
状态:0=禁用,1=启用
菜单类型:tab=选项卡,link=链接,iframe=Iframe
```

### 注意事项

- 不支持复合主键，必须设置单字段主键
- 表注释会自动解析为管理页面标题（如 `会员组表` → `会员组管理`）

## 数据权限控制

在控制器中重写以下属性：

```php
protected bool|string|int $dataLimit = false;           // 数据限制类型
protected string $dataLimitField = 'admin_id';          // 数据限制字段
protected bool $dataLimitFieldAutoFill = true;           // 自动填充当前管理员 ID
```

`$dataLimit` 可选值：

| 值 | 说明 |
|----|------|
| `false` | 关闭数据权限 |
| `personal` | 仅限个人（谁添加的谁可查） |
| `allAuth` | 拥有添加人所有权限时可查 |
| `allAuthAndOthers` | 拥有添加人所有权限且还有其他权限时可查 |
| `parent` | 上级分组中的管理员可查 |
| 数字（角色组 ID） | 指定角色组中的管理员可查 |

## 输入过滤与反 XSS

### 全局过滤（filter）

```php
// app/Request.php 已配置默认过滤规则
// 过滤：trim + strip_tags + htmlspecialchars

// 还原过滤后的数据
htmlspecialchars_decode_improve($string);
```

### 反 XSS（clean_xss）

```php
// 用于富文本数据过滤
$clean = clean_xss($harmString);

// 设置请求输入过滤
$this->request->filter('clean_xss');
```

## 验证码

### 普通验证码

```php
use ba\Captcha;

// 生成
$captcha = new Captcha();
return $captcha->entry($captchaId);

// 验证
$captcha = new Captcha();
if (!$captcha->check($code, $captchaId)) {
    $this->error('验证码错误！');
}
```

### 点选文字验证码

```php
use ba\ClickCaptcha;

$captcha = new ClickCaptcha();
if (!$captcha->check($captchaId, $captchaInfo)) {
    $this->error('验证码错误！');
}
```

## 调试接口

直接访问接口的三种方式：
1. URL 加参：`?server=1`
2. 请求加 header：`server=1`
3. URL 加 index.php：`/index.php/admin/...`
