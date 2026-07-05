# 表单系统

## 表单验证

### 快速构建验证规则

```ts
import { buildValidatorData } from '/@/utils/validate'

const rules = reactive({
    name: [
        buildValidatorData({ name: 'required', title: '变量名' }),
        buildValidatorData({ name: 'varName' }),
    ],
    group: [
        buildValidatorData({ name: 'required', trigger: 'change', message: '请选择分组' }),
    ],
})
```

### 可用验证规则

**常用规则（支持 title 属性）**：

| 规则名 | 说明 |
|--------|------|
| required | 必填 |
| number | 数字 |
| integer | 整数 |
| float | 浮点数 |
| date | 日期 |
| url | URL |
| email | Email |

**预设规则（需用 message）**：

| 规则名 | 说明 |
|--------|------|
| mobile | 手机号 |
| account | 账户 |
| password | 密码 |
| varName | 变量名称 |
| editorRequired | 富文本内容不能为空 |

### 表单验证示例

```vue
<template>
    <el-form ref="formRef" :rules="rules" :model="form" label-width="120px">
        <FormItem label="名称" type="string" v-model="form.name" prop="name" />
        <FormItem label="分组" type="select" v-model="form.group" prop="group"
            :data="{ content: { a: '分组1', b: '分组2' } }" />
        <el-button @click="onSubmit(formRef)">提交</el-button>
    </el-form>
</template>

<script setup lang="ts">
import { buildValidatorData } from '/@/utils/validate'

const formRef = ref()
const form = reactive({ name: '', group: '' })
const rules = reactive({
    name: [buildValidatorData({ name: 'required', title: '名称' })],
})

const onSubmit = (formEl) => {
    formEl?.validate((valid) => {
        if (valid) { /* 验证通过 */ }
    })
}
</script>
```

## 表单与表格配合

### 弹窗表单（PopupForm）

```vue
<template>
    <el-dialog
        class="ba-operate-dialog"
        :close-on-click-modal="false"
        :model-value="['Add', 'Edit'].includes(baTable.form.operate!)"
        @close="baTable.toggleForm"
        width="50%"
    >
        <template #header>
            <div class="title" v-drag="['.ba-operate-dialog', '.el-dialog__header']">
                {{ baTable.form.operate ? t(baTable.form.operate) : '' }}
            </div>
        </template>
        <el-scrollbar v-loading="baTable.form.loading">
            <el-form
                ref="formRef"
                @keyup.enter="baTable.onSubmit(formRef)"
                :model="baTable.form.items"
                :rules="rules"
            >
                <FormItem label="字段" type="string"
                    v-model="baTable.form.items!.field" prop="field" />
            </el-form>
        </el-scrollbar>
        <template #footer>
            <el-button @click="baTable.toggleForm()">取消</el-button>
            <el-button :loading="baTable.form.submitLoading"
                @click="baTable.onSubmit(formRef)" type="primary">保存</el-button>
        </template>
    </el-dialog>
</template>

<script setup lang="ts">
import { inject } from 'vue'
import type baTableClass from '/@/utils/baTable'
const baTable = inject('baTable') as baTableClass
</script>
```

## 表单交互特性

- 表格双击行可快速打开编辑
- 支持批量编辑
- ESC 键退出表单
- Enter 保存表单（或进入下一项）
- 多行输入框中 Ctrl+Enter 保存
- 编辑和添加使用同一表单组件
