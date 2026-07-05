# 状态管理（Pinia）

## 概述

BuildAdmin 使用 Pinia 进行状态管理，配合 `pinia-plugin-persistedstate` 实现状态持久化到 localStorage。

## stores 目录结构

```
stores/
├── adminInfo.ts         # 管理员资料
├── config.ts            # 布局配置
├── memberCenter.ts      # 会员中心
├── navTabs.ts           # 后台标签页
├── siteConfig.ts        # 站点配置
├── terminal.ts          # 终端
├── userInfo.ts          # 用户资料
├── refs.ts              # 全局引用句柄
├── constant/
│   └── cacheKey.ts      # 缓存 Key 定义
└── interface/
    └── index.ts         # 类型定义
```

## 使用示例

```vue
<script setup lang="ts">
import { useAdminInfo } from '/@/stores/adminInfo'
import { useConfig } from '/@/stores/config'
import { useNavTabs } from '/@/stores/navTabs'
import { useTerminal } from '/@/stores/terminal'

const adminInfo = useAdminInfo()
const config = useConfig()
const navTabs = useNavTabs()
const terminal = useTerminal()

console.log('管理员ID:', adminInfo.id)
console.log('当前语言:', config.lang.defaultLang)

navTabs.setFullScreen(true)
terminal.addTask('version-view.npm')
</script>
```

## 定义商店

### 语法一（Options API）

```ts
import { defineStore } from 'pinia'

export const useAdminInfo = defineStore('adminInfo', {
    state: (): AdminInfo => ({
        id: 0,
        username: '',
    }),
    actions: {
        removeToken() { /* ... */ },
    },
    persist: {
        key: ADMIN_INFO,  // 缓存 Key
    },
})
```

### 语法二（Composition API）

```ts
import { defineStore } from 'pinia'
import { reactive } from 'vue'

export const useAdminInfo = defineStore(
    'adminInfo',
    () => {
        const state = reactive({ id: 0, username: '' })
        function removeToken() { /* ... */ }
        return { state, removeToken }
    },
    { persist: { key: ADMIN_INFO } }
)
```

## 缓存 Key 管理

所有缓存 Key 统一定义在 `stores/constant/cacheKey.ts` 中，修改 Key 无需改动具体存储代码。

## 常用 Store

| Store | 用途 | 关键方法/属性 |
|-------|------|--------------|
| `useAdminInfo` | 管理员数据 | `id`, `username`, `getToken()`, `getUserInfo()` |
| `useConfig` | 布局配置 | `lang.defaultLang`, `menuWidth()`, `layout.shrink` |
| `useNavTabs` | 标签页 | `setFullScreen()`, `addTab()`, `closeTab()` |
| `useTerminal` | 终端 | `toggle()`, `addTask()`, `addTaskPM()` |
| `useUserInfo` | 用户数据 | `getToken()`, `getUserInfo()` |
