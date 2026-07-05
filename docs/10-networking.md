# 网络请求

## Web 端（Axios）

### 创建请求

```ts
import createAxios from '/@/utils/axios'

export function getMenuRules() {
    return createAxios({
        url: '/admin/auth.menu/index',
        method: 'get',
    })
}

// 使用
getMenuRules().then((res) => {
    console.log(res)
})
```

### 参数说明

```ts
createAxios(axiosConfig, options, loading)
```

**axiosConfig**：Axios 标准配置，`url` 必填，默认 `method: 'get'`

**options**：

| 选项 | 默认值 | 说明 |
|------|--------|------|
| CancelDuplicateRequest | true | 取消重复请求 |
| loading | false | loading 层效果 |
| reductDataFormat | true | 简洁响应（ApiPromise） |
| showErrorMessage | true | 接口错误提示 |
| showCodeMessage | true | code!=0 时提示 |
| showSuccessMessage | false | code=1 时提示 |

**loading**：Element Plus loading 配置对象

### 带类型的请求

```ts
export function postData(data: anyObj): ApiPromise<TableDefaultData> {
    return createAxios(
        { url: actionUrl.get('edit'), method: 'post', data },
        { showSuccessMessage: true }
    ) as ApiPromise
}
```

## WebNuxt 端（useFetch/$fetch）

### Http.fetch（服务端预渲染）

```ts
export function initialize() {
    return Http.fetch({
        url: '/api/index/index',
        method: 'get',
    })
}

// 使用
const { data } = await initialize()
if (data.value?.code == 1) { /* 成功 */ }
```

### Http.$fetch（用户交互后）

```ts
export function sendSms(mobile: string) {
    return Http.$fetch(
        { url: apiSendSms, method: 'POST', body: { mobile } },
        { showSuccessMessage: true }
    )
}

// 使用
sendSms('18888888888').then((res) => {
    if (res.code == 1) { /* 成功 */ }
})
```

### 区别

| 特性 | Http.fetch | Http.$fetch |
|------|-----------|-------------|
| 用途 | 服务端预渲染获取数据 | 用户交互后请求 |
| 响应数据 | `res.data.value` | `res` |
| 渲染时机 | 组件渲染前 | 组件渲染后 |

## API URL 格式

```
/admin/{Controller}.{action}
```

示例：
- `/admin/user.User/index` - 查看用户列表
- `/admin/user.User/add` - 添加用户
- `/admin/user.User/edit` - 编辑用户
- `/admin/user.User/del` - 删除用户
- `/admin/user.User/sortable` - 排序

## 文件上传

```ts
import { fileUpload } from '/@/api/common'

// 上传接口位于 /@/api/common.ts 的 fileUpload 函数
```
