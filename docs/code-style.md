# 前端代码规范

## 格式化（Prettier）

- 4 空格缩进，无分号，单引号
- 行宽 150，换行符 `lf`
- 尾随逗号 `es5`
- 括号间距 `true`
- Vue 文件脚本/样式不缩进

## Linter（ESLint）

- ESLConfig + vue + typescript + prettier 插件
- 宽松模式：大量规则设为 off（详见 `web/eslint.config.js`）
- `@typescript-eslint/no-unused-vars` 仅 warn，忽略 `_` 前缀变量

## Vue 文件规范

- 使用 `<script setup lang="ts">` + `useTemplateRef`
- 状态管理：Pinia + `pinia-plugin-persistedstate`
- UI 组件：Element Plus
- 路径别名：`/@` → `src/`
- SVG 图标通过 `svgBuilder` 插件从 `src/assets/icons/` 自动构建
- `vue-i18n` 在开发和生产环境加载不同的包（CJS）
