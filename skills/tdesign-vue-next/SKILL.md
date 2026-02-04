---
name: tdesign-vue-next
description: TDesign Vue 3 组件库使用指南，覆盖 tdesign-vue-next 1.x 系列，涵盖基础组件、表单、表格、主题定制、暗黑模式及 AI Chat 组件等场景
---

# TDesign Vue Next

## S - Scope

- **Target**: `tdesign-vue-next@^1` with Vue 3.3+
- **Cover**: 基础组件、布局组件、导航组件、输入组件、数据展示组件、消息提醒组件、主题定制、暗黑模式、Chat 组件
- **Avoid**: 未文档化的内部 API、直接操作 DOM 的 hack、TDesign 其他技术栈版本（React/小程序）

### Default assumptions（未明确时的默认假设）

- **Language**: TypeScript + `<script setup>` 语法
- **Styling**: 使用 CSS 变量和 `ConfigProvider` 进行主题配置，避免直接覆写组件内部类名
- **Provider**: 在应用根组件使用单一 `ConfigProvider` 统一配置
- **Icons**: 使用 `tdesign-icons-vue-next` 图标库
- **Imports**: 按需引入组件 `import { Button } from 'tdesign-vue-next'`

### Scope rules（必须遵循）

1. 仅使用 TDesign 官方文档中记录的 API
2. 不得自行发明 props、events 或组件名称
3. 不使用 `@ts-ignore` 绕过类型检查；类型问题优先查阅官方类型定义
4. 若遇到潜在 bug 或文档与行为不一致，需明确告知用户并引导提交 Issue
5. 示例代码必须可直接运行，不使用伪代码或省略号占位

### Complex triggers（必须打开对应 `Reference`）

| 触发条件                                                  | Reference                              |
| --------------------------------------------------------- | -------------------------------------- |
| 动态表单（`FormItem` 动态增删）、跨字段联动校验、异步校验 | `references/form-advanced.md`          |
| 服务端排序/筛选/分页、虚拟滚动、可编辑表格、树形表格      | `references/table-advanced.md`         |
| 远程搜索、大数据量、分页加载、自定义渲染                  | `references/select-advanced.md`        |
| 受控文件列表、断点续传、自定义上传请求                    | `references/upload-advanced.md`        |
| 异步加载节点、checkStrictly、虚拟滚动                     | `references/tree-advanced.md`          |
| 异步加载、动态选项、自定义面板                            | `references/cascader-advanced.md`      |
| Dialog/Drawer 嵌套、命令式调用、关闭拦截                  | `references/dialog-drawer-advanced.md` |
| 深度主题定制、动态切换主题、组件级样式覆盖                | `references/theming-advanced.md`       |
| 暗黑模式切换、系统偏好跟随、局部暗黑                      | `references/dark-mode.md`              |
| AI 对话组件、流式响应、自定义消息渲染                     | `references/chat-advanced.md`          |
| TDesign Vue Next 1.x 版本差异、升级指南                   | `references/tdesign-v1.md`             |

### `Reference` index（中文索引）

| 主题               | 描述                               | `Reference`                            |
| ------------------ | ---------------------------------- | -------------------------------------- |
| 版本参考           | 1.x 版本范围、升级注意事项         | `references/tdesign-v1.md`             |
| Form 高级          | 动态表单、联动校验、异步校验       | `references/form-advanced.md`          |
| Table 高级         | 虚拟滚动、服务端数据、可编辑单元格 | `references/table-advanced.md`         |
| Select 高级        | 远程搜索、分页、自定义选项         | `references/select-advanced.md`        |
| Upload 高级        | 受控上传、自定义请求、断点续传     | `references/upload-advanced.md`        |
| Tree 高级          | 异步加载、checkStrictly、虚拟滚动  | `references/tree-advanced.md`          |
| Cascader 高级      | 异步加载、动态面板                 | `references/cascader-advanced.md`      |
| Dialog/Drawer 高级 | 嵌套弹层、命令式调用               | `references/dialog-drawer-advanced.md` |
| 主题定制           | CSS 变量、Design Token、动态主题   | `references/theming-advanced.md`       |
| 暗黑模式           | 模式切换、系统偏好、局部暗黑       | `references/dark-mode.md`              |
| Chat 组件          | AI 对话、流式响应、消息渲染        | `references/chat-advanced.md`          |

---

## P - Process

### 1) 识别组件层级

```
用户需求
  ├── 基础展示 → Button / Link / Icon / Typography
  ├── 布局结构 → Layout / Grid / Space / Divider
  ├── 导航交互 → Menu / Tabs / Breadcrumb / Steps / Pagination
  ├── 数据录入 → Form / Input / Select / DatePicker / Upload / ...
  ├── 数据展示 → Table / List / Tree / Card / Descriptions / ...
  ├── 反馈提示 → Message / Notification / Dialog / Drawer / Loading
  └── 高阶场景 → Chat (AI 对话)
```

### 2) 澄清上下文后再建议

在给出组件建议前，需确认：

- Vue 版本（3.3+ 推荐）
- 是否已配置 `ConfigProvider`
- 是否需要支持暗黑模式
- 是否有特殊的国际化需求
- 数据量级（影响是否需要虚拟滚动）

### 3) ConfigProvider 配置优先

```vue
<template>
  <ConfigProvider :global-config="globalConfig">
    <App />
  </ConfigProvider>
</template>

<script setup lang="ts">
import { ConfigProvider } from "tdesign-vue-next";

const globalConfig = {
  // 全局配置
};
</script>
```

### 4) 组件选型规则

| 场景         | 推荐组件                           | 备注                 |
| ------------ | ---------------------------------- | -------------------- |
| 简单列表展示 | `List`                             | 数据量小、无复杂交互 |
| 复杂数据表格 | `Table`                            | 支持排序、筛选、分页 |
| 大数据量表格 | `Table` + `virtual-scroll`         | 启用虚拟滚动         |
| 树形数据选择 | `TreeSelect`                       | 单选/多选树形结构    |
| 级联选择     | `Cascader`                         | 多级联动选择         |
| 简单下拉选择 | `Select`                           | 选项数量适中         |
| 远程搜索选择 | `Select` + `filterable` + `remote` | 禁用本地过滤         |
| 文件上传     | `Upload`                           | 支持拖拽、多文件     |
| 表单收集     | `Form` + `FormItem`                | 统一校验和提交       |
| 轻量提示     | `Message`                          | 操作反馈，自动消失   |
| 重要通知     | `Notification`                     | 需用户确认或包含操作 |
| 确认操作     | `Dialog` / `Popconfirm`            | 根据上下文选择       |
| 侧边详情     | `Drawer`                           | 不打断当前页面流程   |
| AI 对话      | `Chat`                             | 流式消息、多轮对话   |

### 5) 表单决策链

```
需要收集用户输入？
  ├── 是 → 使用 Form 包裹
  │     ├── 需要动态增删字段？ → 查阅 form-advanced.md
  │     ├── 需要跨字段联动？ → 查阅 form-advanced.md
  │     └── 简单表单 → 使用 FormItem + rules
  └── 否 → 直接使用输入组件
```

### 6) 表格决策链

```
需要展示列表数据？
  ├── 数据量 > 1000 行？ → 启用虚拟滚动，查阅 table-advanced.md
  ├── 需要服务端分页/排序？ → 查阅 table-advanced.md
  ├── 需要可编辑单元格？ → 查阅 table-advanced.md
  └── 简单表格 → 使用 Table + columns + data
```

### 7) 主题定制决策链

```
需要定制主题？
  ├── 仅修改品牌色 → ConfigProvider theme prop
  ├── 修改多个 Token → 覆盖 CSS 变量
  ├── 深度定制 → 查阅 theming-advanced.md
  └── 暗黑模式 → 查阅 dark-mode.md
```

### 8) 分流复杂场景到 `Reference`

当识别到 Complex triggers 表格中的触发条件时，必须：

1. 告知用户这是复杂场景
2. 打开对应的 Reference 文档
3. 根据 Reference 中的推荐模式给出建议

### 9) 可访问性和性能检查

- 确保表单控件有合适的 `label`
- 大数据量场景启用虚拟滚动或分页
- 避免在 `template` 中使用复杂计算，使用 `computed`
- 列表渲染确保提供稳定的 `key`

---

## O - Output

### Output should include（按需包含）

1. **组件推荐**：组件名称及选择理由
2. **最小配置**：ConfigProvider 必要配置
3. **代码示例**：可直接运行的 `<script setup>` 代码
4. **性能提示**：大数据量、频繁更新等场景的注意事项
5. **Reference 路径**：复杂场景指向对应参考文档
6. **官方文档链接**：相关组件的官方文档地址

### Output forbidden（禁止输出）

1. 未经验证的 API 或 props
2. 依赖特定内部实现的 hack 代码
3. 不完整的代码片段（缺少 import 或关键配置）
4. TDesign 其他技术栈（React/小程序）的代码
5. 与用户 Vue 版本不兼容的语法

### Regression checklist（回归检查清单）

- [ ] **ConfigProvider**：根组件是否配置、主题 token 是否生效
- [ ] **Form**：`rules` 是否定义、校验触发时机（`trigger`）是否正确、`model` 双向绑定是否正确
- [ ] **Table**：`row-key` 是否提供稳定唯一值、`columns` 是否使用 `computed` 缓存
- [ ] **Select**：远程搜索是否禁用本地过滤（`filterable` + `:filter` 返回 true）
- [ ] **Upload**：受控模式 `v-model:files` 是否正确更新、`action` 或 `requestMethod` 是否配置
- [ ] **Tree/TreeSelect**：`keys` 配置是否与数据结构匹配、异步加载 `load` 函数是否返回 Promise
- [ ] **Dialog/Drawer**：`v-model:visible` 双向绑定、`destroyOnClose` 根据场景配置
- [ ] **暗黑模式**：是否在 `<html>` 或根元素添加 `theme-mode` 属性
- [ ] **Icons**：是否从 `tdesign-icons-vue-next` 正确引入
- [ ] **TypeScript**：组件 props 类型是否正确、事件回调参数类型是否匹配

---

## 快速参考

### 安装

```bash
npm install tdesign-vue-next
# 图标库
npm install tdesign-icons-vue-next
```

### 全量引入

```ts
// main.ts
import { createApp } from "vue";
import TDesign from "tdesign-vue-next";
import "tdesign-vue-next/es/style/index.css";
import App from "./App.vue";

createApp(App).use(TDesign).mount("#app");
```

### 按需引入（推荐）

```ts
// main.ts
import { createApp } from "vue";
import { Button, Input, Form, FormItem } from "tdesign-vue-next";
import "tdesign-vue-next/es/style/index.css";
import App from "./App.vue";

const app = createApp(App);
app.use(Button).use(Input).use(Form).use(FormItem);
app.mount("#app");
```

### 最小示例

```vue
<template>
  <ConfigProvider>
    <Form :model="formData" :rules="rules" @submit="onSubmit">
      <FormItem label="用户名" name="username">
        <Input v-model="formData.username" placeholder="请输入用户名" />
      </FormItem>
      <FormItem>
        <Button theme="primary" type="submit">提交</Button>
      </FormItem>
    </Form>
  </ConfigProvider>
</template>

<script setup lang="ts">
import { reactive } from "vue";
import {
  ConfigProvider,
  Form,
  FormItem,
  Input,
  Button,
} from "tdesign-vue-next";
import type { FormRules, SubmitContext } from "tdesign-vue-next";

const formData = reactive({
  username: "",
});

const rules: FormRules<typeof formData> = {
  username: [{ required: true, message: "用户名必填" }],
};

const onSubmit = ({ validateResult }: SubmitContext) => {
  if (validateResult === true) {
    console.log("提交成功", formData);
  }
};
</script>
```

### 官方资源

- 官方文档：https://tdesign.tencent.com/vue-next/overview
- GitHub：https://github.com/Tencent/tdesign-vue-next
- 设计规范：https://tdesign.tencent.com/design/values
