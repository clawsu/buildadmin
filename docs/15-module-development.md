# 模块开发

## 模块概述

BuildAdmin 的模块系统允许开发者通过模块修改或扩展系统的任何部分。模块可以：
- 在系统任何位置新增文件
- 添加 composer/npm 依赖（由模块安装器自动安装）
- 覆盖系统已有文件
- 自动导入安装 SQL
- 在启用/禁用/更新时自动执行方法

## 模块目录结构

模块目录结构几乎等同于 BuildAdmin 本身，可包含以下目录：

```
modules/{uid}/
├── info.ini                 # 模块基本信息（必需）
├── config.json              # 模块配置/依赖声明
├── install.sql              # 安装 SQL
├── webBootstrap.stub        # WEB 引导程序（可选）
├── app/                     # 后端应用文件
├── config/                  # 配置文件
├── extend/                  # 扩展目录
├── public/                  # 公共文件
├── vendor/                  # Composer 依赖
├── web/                     # 前端源码
└── web-nuxt/                # Nuxt 工程（可选）
```

安装时，以上目录会直接覆盖到 BuildAdmin 项目目录。

## info.ini（模块基本信息）

```ini
uid = test1              # 模块唯一标识
title = 测试模块          # 模块标题
intro = 测试模块的介绍    # 模块介绍
authorid = 1             # 作者 ID
website = https://...    # 模块主页
version = 1.0.0          # 版本号
state = 0                # 状态（系统自动维护）
```

## config.json（依赖声明）

```json
{
    "require": { "nelexa/zip": "^3.3" },
    "require-dev": { "symfony/var-dumper": "^4.2" },
    "dependencies": { "vue-i18n": "~9.1.9" },
    "devDependencies": { "@types/node": "~17.0.9" },
    "nuxtDependencies": {},
    "nuxtDevDependencies": {},
    "protectedFiles": ["app/common.php"],
    "composerConfig": {}
}
```

| 字段 | 说明 |
|------|------|
| require | Composer 生产依赖 |
| require-dev | Composer 开发依赖 |
| dependencies | NPM 生产依赖 |
| devDependencies | NPM 开发依赖 |
| nuxtDependencies | Nuxt 工程 NPM 依赖 |
| protectedFiles | 禁用模块时不能删除的文件 |
| composerConfig | 更新 composer.json 的 config 字段 |

## install.sql（安装 SQL）

```sql
-- 使用 __PREFIX__ 代表数据表前缀
UPDATE __PREFIX__config SET value='值' WHERE name='site_name';

-- 建表以模块 uid 开头
CREATE TABLE IF NOT EXISTS `__PREFIX__module_NewTable` (
    `id` int(10) UNSIGNED NULL AUTO_INCREMENT COMMENT 'ID',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='模块数据表';
```

## 核心控制器

文件名：模块 uid 首字母大写（如 `test` → `Test.php`），位于模块根目录。

```php
namespace modules\test;

class Test
{
    public function install()   {}  // 安装时执行
    public function uninstall() {}  // 卸载时执行
    public function enable()    {}  // 启用时执行
    public function disable()   {}  // 禁用时执行
    public function update()    {}  // 升级时执行
}
```

### 常用操作示例

```php
// 添加后台菜单
use app\common\library\Menu;
use app\admin\model\AdminRule;

public function install()
{
    $pMenu = AdminRule::where('name', 'routine')->value('id');
    Menu::create([[
        'type'      => 'menu',
        'title'     => '通知公告管理',
        'name'      => 'routine/notice',
        'path'      => 'routine/notice',
        'icon'      => 'el-icon-ChatLineRound',
        'menu_type' => 'tab',
        'component' => '/src/views/backend/routine/notice/index.vue',
        'pid'       => $pMenu ?: 0,
        'children'  => [
            ['type' => 'button', 'title' => '查看', 'name' => 'routine/notice/index'],
            ['type' => 'button', 'title' => '添加', 'name' => 'routine/notice/add'],
            ['type' => 'button', 'title' => '编辑', 'name' => 'routine/notice/edit'],
            ['type' => 'button', 'title' => '删除', 'name' => 'routine/notice/del'],
        ],
    ]]);
}

// 删除菜单
public function uninstall()
{
    Menu::delete('routine/notice', true);
}

// 启用/禁用菜单
public function enable()  { Menu::enable('routine/notice'); }
public function disable() { Menu::disable('routine/notice'); }
```

## 行为事件

### 监听 AppInit

在核心控制器中定义 `AppInit` 方法：

```php
public function AppInit()
{
    // 监听自定义事件
    Event::listen('user_register_successed', function ($user) {
        UserScoreLog::create([
            'user_id' => $user->id,
            'score'   => 10,
            'memo'    => '注册赠送积分',
        ]);
    });
}
```

### BuildAdmin 内置事件

| 事件 | 说明 | 参数 |
|------|------|------|
| backendInit | 管理员验权 | 鉴权类实例 |
| frontendInit | 会员验权 | 鉴权类实例 |
| userRegisterSuccess | 会员注册成功 | 会员数据模型 |
| uploadConfigInit | 上传配置 | App 实例 |
| cacheClearAfter | 清理缓存后 | App 实例 |
| AttachmentDel | 附件删除时 | 附件模型 |

## WEB 引导程序（webBootstrap.stub）

向 main.ts/App.vue/web-nuxt/app.vue 插入自定义代码：

```
#main.ts import code start#
import myPlugin from './my-plugin'
#main.ts import code end#

#main.ts start code start#
    console.log('模块初始化')
#main.ts start code end#

#App.vue import code start#
import myComponent from './components/my.vue'
#App.vue import code end#

#App.vue onMounted code start#
    console.log('App mounted')
#App.vue onMounted code end#

#web-nuxt/app.vue import code start#
import myPlugin from './my-plugin'
#web-nuxt/app.vue import code end#

#web-nuxt/app.vue start code start#
    console.log('Nuxt init')
#web-nuxt/app.vue start code end#
```

## 模块打包与发布

### 打包方式

进入模块目录（`info.ini` 所在目录） → 全选文件 → 打包为 zip

### 发布要求

1. 需经官方审核才可上架
2. 覆盖系统核心文件的模块大概率不上架
3. 模块包内不可含侵权素材
4. 截图字体必须为可商用字体
5. 发送资料到 `hi@buildadmin.com`

### 获取模块信息

```php
$dir = root_path() . 'modules/sms/';
$info = Server::getIni(Filesystem::fsFit($dir));
if ($info && $info['state'] == 1) {
    // 模块已安装
}
```
