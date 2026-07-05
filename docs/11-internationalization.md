# 国际化（多语言）

## Web 端

### 语言包结构

```
lang/
├── autoload.ts             # 按需加载映射表
├── globs-en.ts             # 全局英文
├── globs-zh-cn.ts          # 全局中文
├── common/                 # 公共页面语言包（随处可用）
├── backend/                # 后台页面语言包
│   ├── zh-cn.ts            # 后台公共中文
│   └── en.ts               # 后台公共英文
└── frontend/               # 前台页面语言包
    ├── zh-cn.ts            # 前台公共中文
    └── en.ts               # 前台公共英文
```

### 定义语言包

```ts
// 全局语言包 /src/lang/globs-zh-cn.ts
export default {
    Home: '首页',
}

// 后台页面语言包 /src/lang/backend/zh-cn/dashboard.ts
export default {
    republish: '重新发布',
}
```

### 使用语言包

```vue
<!-- 模板中 -->
<span>{{ t('Home') }}</span>
<span>{{ t('dashboard.republish') }}</span>

<!-- script 中 -->
<script setup lang="ts">
import { useI18n } from 'vue-i18n'
const { t } = useI18n()
const title = t('Home')
</script>
```

### 自动加载机制

- 根据路由的 `path` 和 `name` 自动加载对应目录的语言包
- 支持无限目录层级，以 `.` 分隔 key
- 示例：`/src/lang/backend/zh-cn/auth/menu.ts` → `t('auth.menu.name')`

### 注意事项

- 语言翻译 key 请勿带英文句号（`.`）
- 切换语言包会自动刷新站点
- Element Plus 仅载入中文和英文

## Server 端

### 语言包目录

```
app/admin/lang/
├── lang/
│   ├── en.php              # 应用公共英文
│   ├── zh-cn.php           # 应用公共中文
│   ├── en/
│   │   └── user/
│   │       └── moneylog.php
│   └── zh-cn/
│       └── user/
│           └── moneylog.php
```

### 使用翻译

```php
// 全局公共函数 __()
__('Hello %s', ['BuildAdmin']);
__('Hello %s', ['BuildAdmin'], 'en');

// 控制器继承 Api 基类后自动加载语言包
// 访问 /admin/auth.admin/index 自动加载：
// app/admin/lang/zh-cn.php
// app/admin/lang/zh-cn/auth/admin.php
```
