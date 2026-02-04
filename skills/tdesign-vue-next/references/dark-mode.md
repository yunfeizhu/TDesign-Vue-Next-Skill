# 暗黑模式（Dark Mode）复杂场景参考

## 适用场景边界

- 暗黑模式切换
- 跟随系统偏好
- 局部暗黑模式
- 暗黑模式下的自定义颜色
- SSR 下的暗黑模式
- 持久化用户偏好

## 推荐模式

### 1. 使用 theme-mode 属性

TDesign 通过 `<html>` 元素的 `theme-mode` 属性控制暗黑模式：

```html
<html theme-mode="dark"></html>
```

### 2. 使用 CSS 变量覆盖暗黑色值

```css
[theme-mode="dark"] {
  --td-brand-color: #4080ff;
}
```

## 必须避免的反模式

1. **使用 class 而非 attribute**：TDesign 使用 `theme-mode` 属性
2. **在组件级别设置暗黑模式**：应该在根元素统一设置
3. **硬编码颜色值**：应该使用 CSS 变量
4. **忽略系统偏好变化**：用户可能在使用过程中切换系统主题

## 暗黑模式 CSS 变量

```css
/* TDesign 暗黑模式默认变量 */
[theme-mode="dark"] {
  /* 品牌色 */
  --td-brand-color: #4080ff;
  --td-brand-color-hover: #5c9bff;
  --td-brand-color-active: #2e6fe7;

  /* 文字色 */
  --td-text-color-primary: rgba(255, 255, 255, 0.9);
  --td-text-color-secondary: rgba(255, 255, 255, 0.55);
  --td-text-color-placeholder: rgba(255, 255, 255, 0.35);
  --td-text-color-disabled: rgba(255, 255, 255, 0.22);

  /* 背景色 */
  --td-bg-color-page: #0f0f0f;
  --td-bg-color-container: #1a1a1a;
  --td-bg-color-container-hover: #262626;
  --td-bg-color-container-active: #333333;
  --td-bg-color-secondarycontainer: #262626;
  --td-bg-color-component: #333333;

  /* 边框 */
  --td-border-level-1-color: #383838;
  --td-border-level-2-color: #464646;

  /* 功能色 */
  --td-error-color: #e65050;
  --td-warning-color: #ee9c00;
  --td-success-color: #00b374;

  /* 阴影 */
  --td-shadow-1: 0 1px 10px rgba(0, 0, 0, 0.3);
  --td-shadow-2: 0 3px 14px 2px rgba(0, 0, 0, 0.28);
  --td-shadow-3: 0 6px 30px 5px rgba(0, 0, 0, 0.32);
}
```

## 常见实践

### 1. 基础暗黑模式切换

```vue
<template>
  <ConfigProvider>
    <Space>
      <span>主题模式：</span>
      <RadioGroup v-model="themeMode" @change="onThemeChange">
        <RadioButton value="light">浅色</RadioButton>
        <RadioButton value="dark">深色</RadioButton>
        <RadioButton value="auto">跟随系统</RadioButton>
      </RadioGroup>
    </Space>

    <Divider />

    <Space direction="vertical">
      <Card title="示例卡片">
        <p>这是一段文字内容</p>
        <template #actions>
          <Button theme="primary">主要按钮</Button>
          <Button>次要按钮</Button>
        </template>
      </Card>
    </Space>
  </ConfigProvider>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";
import {
  ConfigProvider,
  Space,
  RadioGroup,
  RadioButton,
  Card,
  Button,
  Divider,
} from "tdesign-vue-next";

type ThemeMode = "light" | "dark" | "auto";

const themeMode = ref<ThemeMode>("light");
let mediaQuery: MediaQueryList | null = null;

const applyTheme = (mode: "light" | "dark") => {
  document.documentElement.setAttribute("theme-mode", mode);
};

const getSystemTheme = (): "light" | "dark" => {
  return window.matchMedia("(prefers-color-scheme: dark)").matches
    ? "dark"
    : "light";
};

const handleSystemThemeChange = (e: MediaQueryListEvent) => {
  if (themeMode.value === "auto") {
    applyTheme(e.matches ? "dark" : "light");
  }
};

const onThemeChange = (mode: ThemeMode) => {
  localStorage.setItem("theme-mode", mode);

  if (mode === "auto") {
    applyTheme(getSystemTheme());
  } else {
    applyTheme(mode);
  }
};

onMounted(() => {
  // 恢复用户偏好
  const savedMode = localStorage.getItem("theme-mode") as ThemeMode;
  if (savedMode) {
    themeMode.value = savedMode;
    onThemeChange(savedMode);
  }

  // 监听系统主题变化
  mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");
  mediaQuery.addEventListener("change", handleSystemThemeChange);
});

onUnmounted(() => {
  mediaQuery?.removeEventListener("change", handleSystemThemeChange);
});
</script>
```

### 2. 暗黑模式 Composable

```ts
// composables/useDarkMode.ts
import { ref, watch, onMounted, onUnmounted } from "vue";

export type ThemeMode = "light" | "dark" | "auto";

const STORAGE_KEY = "theme-mode";

export function useDarkMode() {
  const mode = ref<ThemeMode>("light");
  const isDark = ref(false);
  let mediaQuery: MediaQueryList | null = null;

  const getSystemTheme = (): boolean => {
    return window.matchMedia("(prefers-color-scheme: dark)").matches;
  };

  const applyTheme = (dark: boolean) => {
    isDark.value = dark;
    document.documentElement.setAttribute(
      "theme-mode",
      dark ? "dark" : "light",
    );
  };

  const handleSystemChange = (e: MediaQueryListEvent) => {
    if (mode.value === "auto") {
      applyTheme(e.matches);
    }
  };

  const setMode = (newMode: ThemeMode) => {
    mode.value = newMode;
    localStorage.setItem(STORAGE_KEY, newMode);

    if (newMode === "auto") {
      applyTheme(getSystemTheme());
    } else {
      applyTheme(newMode === "dark");
    }
  };

  const toggle = () => {
    if (mode.value === "auto") {
      setMode(isDark.value ? "light" : "dark");
    } else {
      setMode(mode.value === "dark" ? "light" : "dark");
    }
  };

  const init = () => {
    const savedMode = localStorage.getItem(STORAGE_KEY) as ThemeMode | null;

    if (savedMode) {
      setMode(savedMode);
    } else {
      // 默认跟随系统
      setMode("auto");
    }

    mediaQuery = window.matchMedia("(prefers-color-scheme: dark)");
    mediaQuery.addEventListener("change", handleSystemChange);
  };

  const cleanup = () => {
    mediaQuery?.removeEventListener("change", handleSystemChange);
  };

  return {
    mode,
    isDark,
    setMode,
    toggle,
    init,
    cleanup,
  };
}
```

```vue
<!-- App.vue -->
<script setup lang="ts">
import { onMounted, onUnmounted } from "vue";
import { useDarkMode } from "@/composables/useDarkMode";

const { init, cleanup } = useDarkMode();

onMounted(init);
onUnmounted(cleanup);
</script>
```

### 3. 暗黑模式切换按钮组件

```vue
<template>
  <Button
    variant="text"
    shape="square"
    :aria-label="isDark ? '切换到浅色模式' : '切换到深色模式'"
    @click="toggle"
  >
    <SunnyIcon v-if="isDark" />
    <MoonIcon v-else />
  </Button>
</template>

<script setup lang="ts">
import { Button } from "tdesign-vue-next";
import { SunnyIcon, MoonIcon } from "tdesign-icons-vue-next";
import { useDarkMode } from "@/composables/useDarkMode";

const { isDark, toggle } = useDarkMode();
</script>
```

### 4. 局部暗黑模式

```vue
<template>
  <div class="page">
    <!-- 页面使用浅色模式 -->
    <p>这是浅色区域的内容</p>

    <!-- 局部使用暗黑模式 -->
    <div class="dark-section" theme-mode="dark">
      <Card title="暗黑区域">
        <p>这个区域使用暗黑模式</p>
        <Button theme="primary">按钮</Button>
      </Card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { Card, Button } from "tdesign-vue-next";
</script>

<style scoped>
.dark-section {
  padding: 20px;
  background-color: var(--td-bg-color-page);
  border-radius: var(--td-radius-default);
}

/* 局部暗黑模式变量 */
.dark-section[theme-mode="dark"] {
  --td-bg-color-page: #0f0f0f;
  --td-bg-color-container: #1a1a1a;
  --td-text-color-primary: rgba(255, 255, 255, 0.9);
}
</style>
```

### 5. 暗黑模式自定义颜色

```css
/* styles/dark-theme.css */

/* 浅色模式 */
:root {
  --td-brand-color: #0052d9;
  --custom-bg: #ffffff;
  --custom-text: #333333;
}

/* 暗黑模式 */
[theme-mode="dark"] {
  /* 调整品牌色以适应暗黑背景 */
  --td-brand-color: #4080ff;
  --td-brand-color-hover: #5c9bff;
  --td-brand-color-active: #2e6fe7;

  /* 自定义变量 */
  --custom-bg: #1a1a1a;
  --custom-text: #e5e5e5;

  /* 调整功能色 */
  --td-error-color: #e65050;
  --td-warning-color: #ee9c00;
  --td-success-color: #00b374;

  /* 调整图表等可视化元素的颜色 */
  --chart-line-color: #4080ff;
  --chart-area-color: rgba(64, 128, 255, 0.2);
}
```

### 6. SSR 暗黑模式处理

```ts
// server/render.ts (Nuxt/SSR 场景)

// 1. 服务端：从 cookie 或请求头获取主题
const getThemeFromRequest = (req: Request): "light" | "dark" => {
  const cookie = req.headers.get("cookie");
  const themeMatch = cookie?.match(/theme-mode=(light|dark)/);

  if (themeMatch) {
    return themeMatch[1] as "light" | "dark";
  }

  // 从 Sec-CH-Prefers-Color-Scheme 头获取（如果浏览器支持）
  const prefersColorScheme = req.headers.get("Sec-CH-Prefers-Color-Scheme");
  if (prefersColorScheme === "dark") {
    return "dark";
  }

  return "light";
};

// 2. 在 HTML 模板中注入主题
const renderHTML = (theme: "light" | "dark") => `
<!DOCTYPE html>
<html theme-mode="${theme}">
<head>
  <script>
    // 防止闪烁：在 CSS 加载前设置主题
    (function() {
      const saved = localStorage.getItem('theme-mode');
      if (saved === 'auto') {
        const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
        document.documentElement.setAttribute('theme-mode', prefersDark ? 'dark' : 'light');
      } else if (saved) {
        document.documentElement.setAttribute('theme-mode', saved);
      }
    })();
  </script>
</head>
<body>
  <!-- app -->
</body>
</html>
`;
```

### 7. 防止暗黑模式闪烁

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <script>
      // 同步脚本，在 CSS 加载前执行
      (function () {
        const STORAGE_KEY = "theme-mode";
        const saved = localStorage.getItem(STORAGE_KEY);

        let theme = "light";

        if (saved === "dark") {
          theme = "dark";
        } else if (saved === "auto" || !saved) {
          // 跟随系统或默认
          theme = window.matchMedia("(prefers-color-scheme: dark)").matches
            ? "dark"
            : "light";
        }

        document.documentElement.setAttribute("theme-mode", theme);

        // 添加 CSS 类以便在样式中使用
        document.documentElement.classList.add(`theme-${theme}`);
      })();
    </script>

    <!-- 关键 CSS 内联，包含两种主题的基础变量 -->
    <style>
      :root {
        --td-bg-color-page: #f3f3f3;
        --td-text-color-primary: rgba(0, 0, 0, 0.9);
      }
      [theme-mode="dark"] {
        --td-bg-color-page: #0f0f0f;
        --td-text-color-primary: rgba(255, 255, 255, 0.9);
      }
      body {
        background-color: var(--td-bg-color-page);
        color: var(--td-text-color-primary);
      }
    </style>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

### 8. 图片和媒体的暗黑模式适配

```vue
<template>
  <div class="media-container">
    <!-- 使用 picture 元素适配不同主题 -->
    <picture>
      <source
        srcset="/images/logo-dark.svg"
        media="(prefers-color-scheme: dark)"
      />
      <img src="/images/logo-light.svg" alt="Logo" />
    </picture>

    <!-- 或使用 CSS 控制 -->
    <img
      :src="isDark ? '/images/hero-dark.png' : '/images/hero-light.png'"
      alt="Hero"
      class="hero-image"
    />
  </div>
</template>

<script setup lang="ts">
import { useDarkMode } from "@/composables/useDarkMode";

const { isDark } = useDarkMode();
</script>

<style scoped>
/* 反转图片颜色（适用于简单图标） */
.icon-invertible {
  filter: none;
}

[theme-mode="dark"] .icon-invertible {
  filter: invert(1);
}

/* 降低图片亮度（适用于照片） */
[theme-mode="dark"] .photo {
  filter: brightness(0.9);
}
</style>
```

## 常见问题与建议

### Q: 主题切换时页面闪烁？

在 `<head>` 中添加同步脚本，在 CSS 加载前设置主题属性。

### Q: 某些组件颜色不对？

确保使用 CSS 变量而非硬编码颜色值，检查是否有 `!important` 覆盖。

### Q: 如何在暗黑模式下调整第三方组件？

使用 CSS 选择器覆盖：

```css
[theme-mode="dark"] .third-party-component {
  background-color: var(--td-bg-color-container);
  color: var(--td-text-color-primary);
}
```

### Q: 如何检测当前是否为暗黑模式？

```ts
const isDark = document.documentElement.getAttribute("theme-mode") === "dark";
```

## 最小示例

```vue
<template>
  <div>
    <Button @click="toggleTheme">切换主题</Button>
    <p>当前主题：{{ isDark ? "暗黑" : "浅色" }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { Button } from "tdesign-vue-next";

const isDark = ref(false);

const toggleTheme = () => {
  isDark.value = !isDark.value;
  document.documentElement.setAttribute(
    "theme-mode",
    isDark.value ? "dark" : "light",
  );
};

onMounted(() => {
  isDark.value = document.documentElement.getAttribute("theme-mode") === "dark";
});
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"简单主题切换"，回到主 Skill 的主题定制决策链
- 若涉及深度主题定制（颜色、间距等），查阅 `theming-advanced.md`

## 参考文档

- 暗黑模式指南：https://tdesign.tencent.com/vue-next/config#dark-mode
- 设计规范-暗黑模式：https://tdesign.tencent.com/design/dark
