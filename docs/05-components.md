# 组件系统

## 图标组件（Icon）

支持四种图标类型：

```vue
<!-- Font Awesome（fa 前缀 + 空格） -->
<Icon name="fa fa-pencil" />

<!-- Element Plus（el-icon 前缀，驼峰命名） -->
<Icon name="el-icon-Close" color="#8595F4" size="20" />

<!-- 本地 SVG（local 前缀，文件名） -->
<Icon name="local-logo" />

<!-- 阿里 iconfont（iconfont 前缀 + 空格） -->
<Icon name="iconfont icon-user" size="20" />
```

### 本地 SVG 图标

将 SVG 文件放入 `/web/src/assets/icons/`，重新编译后自动加载。

### iconfont 配置

在 `/web/src/utils/iconfont.ts` 的 `cssUrls` 中添加 iconfont 项目的 Font class 链接。

## 图标选择器

通过 `BaInput` 或 `FormItem` 使用：

```vue
<BaInput type="icon" v-model="icon" />
<FormItem label="选择图标" type="icon" v-model="icon" />
```

## 输入组件（baInput）

### 数组组件

```vue
<FormItem label="数组" type="array" v-model="state.array" />
```

### 上传组件

```vue
<!-- 图片上传 -->
<FormItem label="头像" type="image" v-model="state.avatar" />
<FormItem label="多图" type="images" v-model="state.images" />

<!-- 文件上传 -->
<FormItem label="文件" type="file" v-model="state.file" />
<FormItem label="多文件" type="files" v-model="state.files" />
```

上传组件属性：

| 属性 | 说明 | 默认值 |
|------|------|--------|
| type | image/images/file/files | image |
| data | 上传额外数据 | {} |
| returnFullUrl | 返回完整路径 | false |
| hideSelectFile | 隐藏选择器 | false |
| forceLocal | 强制本地存储 | false |

### 富文本编辑器

```vue
<FormItem label="编辑器" type="editor" v-model="state.editor" />
<!-- 指定编辑器类型 -->
<FormItem label="MD" type="editor" v-model="state.editor"
    :input-attr="{ editorType: 'md-v3' }" />
```

编辑器代码位于 `/@/components/mixins/editor/`。

### 远程下拉组件

```vue
<FormItem
    label="会员"
    type="remoteSelect"
    v-model="state.userId"
    :input-attr="{
        pk: 'user.id',           // value 字段（关联表需带别名前缀）
        field: 'nickname',       // label 字段
        remoteUrl: '/admin/user.User/index',
    }"
/>
<!-- 多选 -->
<FormItem label="多选" type="remoteSelects" v-model="state.ids"
    :input-attr="{ pk: 'user.id', field: 'nickname', remoteUrl: '...' }" />
```

### 省份城市选择器

```vue
<FormItem label="城市" type="city" v-model="state.city"
    :input-attr="{ level: 3 }" />  <!-- 1=省, 2=省市, 3=省市区 -->
```

## 表单项组件（FormItem）

FormItem 内嵌 `el-form-item` + `baInput`，继承 `el-form-item` 所有属性。

### 可用 type 类型

string, password, number, radio, checkbox, switch, textarea, array, datetime, year, date, time, select, selects, remoteSelect, remoteSelects, editor, city, image, images, file, files, icon, color

### 关键属性

| 属性 | 说明 |
|------|------|
| type | 输入框类型（必填） |
| v-model | 双向绑定值（必填） |
| inputAttr | 输入框附加属性 |
| tip | 输入提示信息 |
| placeholder | placeholder（inputAttr.placeholder 的别名） |
| data | 额外数据（单选/复选选项等） |

### Radio/Checkbox 简写

```vue
<!-- v2.1.0+ 可直接在 inputAttr 中配置 -->
<FormItem label="选项" type="radio" v-model="val"
    :input-attr="{ border: true, content: { a: '选项a', b: '选项b' } }" />
```
