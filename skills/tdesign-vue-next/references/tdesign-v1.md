# TDesign Vue Next 1.x 版本参考

## 适用场景边界

- TDesign Vue Next 1.x 系列版本的特性和 API
- 从 0.x 升级到 1.x 的迁移指南
- 版本间的 Breaking Changes 说明
- 新版本特性的使用方式

## 版本范围

### 1.x 稳定版

- **Vue 版本要求**：Vue 3.3+
- **Node 版本要求**：Node 18+
- **TypeScript**：完整的类型支持

### 主要特性

| 特性            | 说明                           |
| --------------- | ------------------------------ |
| Composition API | 全面支持 `<script setup>` 语法 |
| TypeScript      | 完整的类型定义和类型推导       |
| Tree Shaking    | 支持按需引入，优化打包体积     |
| CSS 变量        | 基于 CSS 变量的主题系统        |
| 暗黑模式        | 内置暗黑模式支持               |
| SSR             | 支持服务端渲染                 |

## 推荐模式

### 1. 按需引入（推荐）

```ts
// main.ts
import { createApp } from "vue";
import { Button, Table, Form, FormItem } from "tdesign-vue-next";
// 引入组件库全局样式
import "tdesign-vue-next/es/style/index.css";

const app = createApp(App);
app.use(Button).use(Table).use(Form).use(FormItem);
```

### 2. 使用 unplugin-vue-components 自动导入

```ts
// vite.config.ts
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import Components from "unplugin-vue-components/vite";
import { TDesignResolver } from "unplugin-vue-components/resolvers";

export default defineConfig({
  plugins: [
    vue(),
    Components({
      resolvers: [
        TDesignResolver({
          library: "vue-next",
        }),
      ],
    }),
  ],
});
```

## 必须避免的反模式

1. **不要混用全量引入和按需引入**：选择一种方式并保持一致
2. **不要忽略样式引入**：组件需要配合样式文件才能正常显示
3. **不要使用 Vue 2 语法**：TDesign Vue Next 仅支持 Vue 3

## 从 0.x 升级到 1.x

### Breaking Changes

#### 1. 组件命名调整

```diff
- import { TButton } from 'tdesign-vue-next';
+ import { Button } from 'tdesign-vue-next';
```

#### 2. 事件命名规范化

所有事件统一使用 `on` 前缀的驼峰命名：

```diff
- @change="handleChange"
+ @change="handleChange"  // 保持不变，但类型定义更完善
```

#### 3. CSS 变量前缀

```diff
- --td-brand-color
+ --td-brand-color  // 保持不变，但变量更加完善
```

### 升级步骤

1. **更新依赖**

```bash
npm install tdesign-vue-next@latest
npm install tdesign-icons-vue-next@latest
```

2. **检查 Vue 版本**

```bash
npm install vue@^3.3.0
```

3. **更新 TypeScript 配置**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "jsx": "preserve"
  }
}
```

4. **运行类型检查**

```bash
npx vue-tsc --noEmit
```

## 常见问题与建议

### Q: 样式不生效怎么办？

确保引入了全局样式文件：

```ts
import "tdesign-vue-next/es/style/index.css";
```

### Q: TypeScript 类型报错？

1. 确保 `tsconfig.json` 中 `moduleResolution` 设置正确
2. 使用 `vue-tsc` 进行类型检查
3. 组件 props 使用官方导出的类型

```ts
import type { ButtonProps, TableProps } from "tdesign-vue-next";
```

### Q: 图标不显示？

确保安装并正确引入图标库：

```bash
npm install tdesign-icons-vue-next
```

```vue
<script setup lang="ts">
import { SearchIcon } from "tdesign-icons-vue-next";
</script>

<template>
  <SearchIcon />
</template>
```

### Q: SSR 样式闪烁？

使用 CSS 变量方案，确保在 HTML 中提前注入主题变量：

```html
<html data-theme="light">
  <!-- 或 data-theme="dark" -->
</html>
```

## 最小示例

```vue
<template>
  <ConfigProvider>
    <Space direction="vertical" style="width: 100%">
      <Button theme="primary">主要按钮</Button>
      <Button theme="default">次要按钮</Button>
      <Button theme="danger">危险按钮</Button>
    </Space>
  </ConfigProvider>
</template>

<script setup lang="ts">
import { ConfigProvider, Button, Space } from "tdesign-vue-next";
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"选择哪个组件"或"基本用法"，回到主 Skill 的组件选型规则
- 若涉及具体组件的复杂场景，查阅对应组件的 Reference 文档

## 参考文档

- 官方文档：https://tdesign.tencent.com/vue-next/overview
- 更新日志：https://tdesign.tencent.com/vue-next/changelog
- GitHub Releases：https://github.com/Tencent/tdesign-vue-next/releases
