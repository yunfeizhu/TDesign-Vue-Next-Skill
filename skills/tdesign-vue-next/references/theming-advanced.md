# 主题定制（Theming）复杂场景参考

## 适用场景边界

- 深度主题定制（超出简单品牌色修改）
- 动态切换主题（运行时切换）
- 组件级样式覆盖
- CSS 变量与 Design Token 使用
- 多品牌/多租户主题
- 响应式主题（根据屏幕尺寸调整）

## 推荐模式

### 1. 使用 CSS 变量覆盖 Design Token

TDesign 基于 CSS 变量构建主题系统，优先通过覆盖 CSS 变量定制：

```css
:root {
  --td-brand-color: #0052d9;
  --td-brand-color-hover: #0034b5;
  --td-brand-color-active: #002a9c;
}
```

### 2. ConfigProvider 配置全局主题

```vue
<ConfigProvider :global-config="globalConfig">
  <App />
</ConfigProvider>
```

## 必须避免的反模式

1. **直接修改组件内部类名样式**：升级后可能失效
2. **使用 `!important` 强制覆盖**：难以维护和调试
3. **在每个组件上单独设置主题**：应该统一在根级别配置
4. **修改 node_modules 中的样式文件**：不会被版本控制

## CSS 变量体系

### 基础色彩变量

```css
:root {
  /* 品牌色 */
  --td-brand-color: #0052d9;
  --td-brand-color-1: #f2f3ff;
  --td-brand-color-2: #d9e1ff;
  --td-brand-color-3: #b5c7ff;
  --td-brand-color-4: #8eabff;
  --td-brand-color-5: #618dff;
  --td-brand-color-6: #366ef4;
  --td-brand-color-7: #0052d9;
  --td-brand-color-8: #003cab;
  --td-brand-color-9: #002a7c;
  --td-brand-color-10: #001a57;

  /* 功能色 */
  --td-error-color: #e34d59;
  --td-warning-color: #ed7b2f;
  --td-success-color: #00a870;

  /* 文字色 */
  --td-text-color-primary: rgba(0, 0, 0, 0.9);
  --td-text-color-secondary: rgba(0, 0, 0, 0.6);
  --td-text-color-placeholder: rgba(0, 0, 0, 0.4);
  --td-text-color-disabled: rgba(0, 0, 0, 0.26);

  /* 背景色 */
  --td-bg-color-page: #f3f3f3;
  --td-bg-color-container: #fff;
  --td-bg-color-container-hover: #f3f3f3;

  /* 边框 */
  --td-border-level-1-color: #e7e7e7;
  --td-border-level-2-color: #dcdcdc;

  /* 圆角 */
  --td-radius-small: 3px;
  --td-radius-default: 6px;
  --td-radius-medium: 9px;
  --td-radius-large: 12px;

  /* 阴影 */
  --td-shadow-1: 0 1px 10px rgba(0, 0, 0, 0.05);
  --td-shadow-2: 0 3px 14px 2px rgba(0, 0, 0, 0.03);
  --td-shadow-3: 0 6px 30px 5px rgba(0, 0, 0, 0.05);
}
```

## 常见实践

### 1. 修改品牌色

```css
/* styles/theme.css */
:root {
  /* 定义新的品牌色 */
  --td-brand-color: #722ed1;
  --td-brand-color-hover: #531dab;
  --td-brand-color-active: #391085;
  --td-brand-color-disabled: #d3adf7;
  --td-brand-color-light: #f9f0ff;

  /* 品牌色梯度 */
  --td-brand-color-1: #f9f0ff;
  --td-brand-color-2: #efdbff;
  --td-brand-color-3: #d3adf7;
  --td-brand-color-4: #b37feb;
  --td-brand-color-5: #9254de;
  --td-brand-color-6: #722ed1;
  --td-brand-color-7: #531dab;
  --td-brand-color-8: #391085;
  --td-brand-color-9: #22075e;
  --td-brand-color-10: #120338;
}
```

```ts
// main.ts
import "tdesign-vue-next/es/style/index.css";
import "./styles/theme.css"; // 在组件样式之后引入
```

### 2. 动态切换主题

```vue
<template>
  <ConfigProvider>
    <Space>
      <span>选择主题：</span>
      <RadioGroup v-model="currentTheme" @change="onThemeChange">
        <Radio value="default">默认</Radio>
        <Radio value="blue">蓝色</Radio>
        <Radio value="green">绿色</Radio>
        <Radio value="purple">紫色</Radio>
      </RadioGroup>
    </Space>
    <Divider />
    <Button theme="primary">主题按钮</Button>
  </ConfigProvider>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import {
  ConfigProvider,
  Space,
  RadioGroup,
  Radio,
  Button,
  Divider,
} from "tdesign-vue-next";

const currentTheme = ref("default");

const themes: Record<string, Record<string, string>> = {
  default: {
    "--td-brand-color": "#0052d9",
    "--td-brand-color-hover": "#0034b5",
    "--td-brand-color-active": "#002a9c",
  },
  blue: {
    "--td-brand-color": "#1890ff",
    "--td-brand-color-hover": "#40a9ff",
    "--td-brand-color-active": "#096dd9",
  },
  green: {
    "--td-brand-color": "#52c41a",
    "--td-brand-color-hover": "#73d13d",
    "--td-brand-color-active": "#389e0d",
  },
  purple: {
    "--td-brand-color": "#722ed1",
    "--td-brand-color-hover": "#9254de",
    "--td-brand-color-active": "#531dab",
  },
};

const applyTheme = (themeName: string) => {
  const theme = themes[themeName];
  const root = document.documentElement;

  Object.entries(theme).forEach(([key, value]) => {
    root.style.setProperty(key, value);
  });
};

const onThemeChange = (value: string) => {
  applyTheme(value);
  // 持久化主题选择
  localStorage.setItem("theme", value);
};

onMounted(() => {
  const savedTheme = localStorage.getItem("theme");
  if (savedTheme && themes[savedTheme]) {
    currentTheme.value = savedTheme;
    applyTheme(savedTheme);
  }
});
</script>
```

### 3. 多品牌主题

```ts
// themes/brand-a.ts
export const brandATheme = {
  "--td-brand-color": "#ff6b00",
  "--td-brand-color-hover": "#ff8533",
  "--td-brand-color-active": "#cc5500",
  "--td-text-color-brand": "#ff6b00",
};

// themes/brand-b.ts
export const brandBTheme = {
  "--td-brand-color": "#00b96b",
  "--td-brand-color-hover": "#20c77c",
  "--td-brand-color-active": "#009456",
  "--td-text-color-brand": "#00b96b",
};
```

```vue
<script setup lang="ts">
import { onMounted } from "vue";
import { brandATheme } from "@/themes/brand-a";
import { brandBTheme } from "@/themes/brand-b";

const loadBrandTheme = (brand: string) => {
  const theme = brand === "brand-a" ? brandATheme : brandBTheme;
  const root = document.documentElement;

  Object.entries(theme).forEach(([key, value]) => {
    root.style.setProperty(key, value);
  });
};

onMounted(() => {
  // 根据域名、配置等判断当前品牌
  const brand = window.location.hostname.includes("brand-a")
    ? "brand-a"
    : "brand-b";
  loadBrandTheme(brand);
});
</script>
```

### 4. 组件级样式覆盖

```vue
<template>
  <Button class="custom-button" theme="primary">自定义按钮</Button>
</template>

<style scoped>
.custom-button {
  /* 使用 CSS 变量确保主题一致性 */
  --td-button-primary-bg-color: var(--custom-primary-color, #0052d9);
  --td-button-primary-border-color: var(--custom-primary-color, #0052d9);

  /* 直接覆盖样式 */
  border-radius: 20px;
  font-weight: 600;
}
</style>
```

### 5. Less 变量覆盖（构建时）

```less
// styles/variables.less
@brand-color: #722ed1;
@brand-color-hover: #9254de;
@brand-color-active: #531dab;
```

```ts
// vite.config.ts
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";

export default defineConfig({
  plugins: [vue()],
  css: {
    preprocessorOptions: {
      less: {
        modifyVars: {
          "brand-color": "#722ed1",
        },
        javascriptEnabled: true,
      },
    },
  },
});
```

### 6. 响应式主题

```css
:root {
  /* 默认桌面端 */
  --td-comp-size-xxxs: 16px;
  --td-comp-size-xxs: 20px;
  --td-comp-size-xs: 24px;
  --td-comp-size-s: 32px;
  --td-comp-size-m: 40px;
  --td-comp-size-l: 48px;
  --td-font-size-base: 14px;
}

/* 移动端适配 */
@media (max-width: 768px) {
  :root {
    --td-comp-size-xxxs: 20px;
    --td-comp-size-xxs: 24px;
    --td-comp-size-xs: 28px;
    --td-comp-size-s: 36px;
    --td-comp-size-m: 44px;
    --td-comp-size-l: 52px;
    --td-font-size-base: 16px;
  }
}
```

### 7. 主题配置持久化

```ts
// composables/useTheme.ts
import { ref, watch } from "vue";

export interface ThemeConfig {
  mode: "light" | "dark";
  primaryColor: string;
  borderRadius: "small" | "default" | "large";
}

const STORAGE_KEY = "tdesign-theme-config";

const defaultConfig: ThemeConfig = {
  mode: "light",
  primaryColor: "#0052d9",
  borderRadius: "default",
};

export function useTheme() {
  const config = ref<ThemeConfig>({ ...defaultConfig });

  // 从存储中加载配置
  const loadConfig = () => {
    try {
      const saved = localStorage.getItem(STORAGE_KEY);
      if (saved) {
        config.value = { ...defaultConfig, ...JSON.parse(saved) };
      }
    } catch (e) {
      console.warn("Failed to load theme config", e);
    }
  };

  // 应用主题配置
  const applyConfig = (newConfig: Partial<ThemeConfig>) => {
    config.value = { ...config.value, ...newConfig };

    const root = document.documentElement;

    // 应用模式
    root.setAttribute("theme-mode", config.value.mode);

    // 应用主色
    root.style.setProperty("--td-brand-color", config.value.primaryColor);

    // 应用圆角
    const radiusMap = {
      small: "3px",
      default: "6px",
      large: "12px",
    };
    root.style.setProperty(
      "--td-radius-default",
      radiusMap[config.value.borderRadius],
    );

    // 持久化
    localStorage.setItem(STORAGE_KEY, JSON.stringify(config.value));
  };

  // 重置配置
  const resetConfig = () => {
    applyConfig(defaultConfig);
  };

  return {
    config,
    loadConfig,
    applyConfig,
    resetConfig,
  };
}
```

```vue
<script setup lang="ts">
import { onMounted } from "vue";
import { useTheme } from "@/composables/useTheme";

const { config, loadConfig, applyConfig } = useTheme();

onMounted(() => {
  loadConfig();
  applyConfig(config.value);
});
</script>
```

## 常见问题与建议

### Q: 主题变量没有生效？

1. 确保自定义样式在组件库样式之后引入
2. 检查 CSS 变量名是否正确
3. 使用浏览器开发者工具检查变量是否被覆盖

### Q: 如何查看所有可用的 CSS 变量？

查看 `node_modules/tdesign-vue-next/es/style/` 目录下的样式文件，或参考官方设计规范文档。

### Q: 动态主题切换后部分组件样式没更新？

某些样式可能在组件初始化时计算，使用 CSS 变量可以确保动态更新。避免在 JavaScript 中硬编码颜色值。

### Q: 如何在 SSR 中使用主题？

确保主题相关的 CSS 变量在服务端渲染的 HTML 中包含：

```html
<html data-theme="light">
  <head>
    <style>
      :root {
        --td-brand-color: #0052d9;
      }
    </style>
  </head>
</html>
```

## 最小示例

```vue
<template>
  <ConfigProvider>
    <Button theme="primary">主题按钮</Button>
  </ConfigProvider>
</template>

<script setup lang="ts">
import { ConfigProvider, Button } from "tdesign-vue-next";
</script>

<style>
:root {
  --td-brand-color: #722ed1;
}
</style>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"简单品牌色修改"，回到主 Skill 的主题定制决策链
- 若涉及暗黑模式，查阅 `dark-mode.md`

## 参考文档

- 设计规范：https://tdesign.tencent.com/design/color
- 主题配置：https://tdesign.tencent.com/vue-next/config
- CSS 变量列表：https://github.com/Tencent/tdesign-common/tree/develop/style
