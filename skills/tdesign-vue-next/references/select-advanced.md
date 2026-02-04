# 选择器（Select）复杂场景参考

## 适用场景边界

- 远程搜索（异步加载选项）
- 大数据量选项（虚拟滚动）
- 分页加载选项
- 自定义选项渲染
- 多选限制与标签管理
- 创建新选项
- 级联选择器联动

## 推荐模式

### 1. 远程搜索禁用本地过滤

远程搜索时，必须禁用本地过滤逻辑，让服务端处理搜索：

```vue
<Select
  v-model="value"
  filterable
  :filter="() => true"
  :loading="loading"
  @search="onRemoteSearch"
/>
```

### 2. 使用防抖控制请求频率

```ts
import { useDebounceFn } from "@vueuse/core";

const onRemoteSearch = useDebounceFn((keyword: string) => {
  fetchOptions(keyword);
}, 300);
```

## 必须避免的反模式

1. **远程搜索时不禁用本地过滤**：导致搜索结果被本地过滤器二次过滤
2. **大数据量不使用虚拟滚动**：会导致下拉框卡顿
3. **频繁触发搜索不加防抖**：造成大量无效请求
4. **选项列表每次渲染都重新创建**：影响性能

## 常见实践

### 1. 远程搜索

```vue
<template>
  <Select
    v-model="selectedValue"
    filterable
    :filter="() => true"
    :options="options"
    :loading="loading"
    placeholder="输入关键字搜索"
    @search="onSearch"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Select } from "tdesign-vue-next";
import type { SelectOption } from "tdesign-vue-next";

const selectedValue = ref("");
const options = ref<SelectOption[]>([]);
const loading = ref(false);

// 使用防抖
let searchTimer: ReturnType<typeof setTimeout>;

const onSearch = (keyword: string) => {
  clearTimeout(searchTimer);

  if (!keyword) {
    options.value = [];
    return;
  }

  searchTimer = setTimeout(async () => {
    loading.value = true;
    try {
      const response = await fetch(
        `/api/search?keyword=${encodeURIComponent(keyword)}`,
      );
      const data = await response.json();
      options.value = data.map((item: any) => ({
        label: item.name,
        value: item.id,
      }));
    } finally {
      loading.value = false;
    }
  }, 300);
};
</script>
```

### 2. 分页加载（滚动加载更多）

```vue
<template>
  <Select
    v-model="selectedValue"
    :options="options"
    :loading="loading"
    :popup-props="{ onScroll: onPopupScroll }"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import type { SelectOption } from "tdesign-vue-next";

const selectedValue = ref("");
const options = ref<SelectOption[]>([]);
const loading = ref(false);
const page = ref(1);
const hasMore = ref(true);

const fetchOptions = async (pageNum: number) => {
  if (loading.value || !hasMore.value) return;

  loading.value = true;
  try {
    const response = await fetch(`/api/options?page=${pageNum}&pageSize=20`);
    const { data, total } = await response.json();

    const newOptions = data.map((item: any) => ({
      label: item.name,
      value: item.id,
    }));

    if (pageNum === 1) {
      options.value = newOptions;
    } else {
      options.value = [...options.value, ...newOptions];
    }

    hasMore.value = options.value.length < total;
    page.value = pageNum;
  } finally {
    loading.value = false;
  }
};

const onPopupScroll = ({ e }: { e: Event }) => {
  const target = e.target as HTMLElement;
  const { scrollTop, scrollHeight, clientHeight } = target;

  // 滚动到底部加载更多
  if (scrollTop + clientHeight >= scrollHeight - 20) {
    fetchOptions(page.value + 1);
  }
};

// 初始加载
fetchOptions(1);
</script>
```

### 3. 虚拟滚动

```vue
<template>
  <Select
    v-model="selectedValue"
    :options="largeOptions"
    :scroll="{ type: 'virtual' }"
    :popup-props="{ overlayInnerStyle: { maxHeight: '300px' } }"
  />
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import type { SelectOption } from "tdesign-vue-next";

const selectedValue = ref("");

// 大数据量选项
const largeOptions = computed<SelectOption[]>(() =>
  Array.from({ length: 10000 }, (_, i) => ({
    label: `选项 ${i + 1}`,
    value: i + 1,
  })),
);
</script>
```

### 4. 自定义选项渲染

```vue
<template>
  <Select v-model="selectedValue" :options="options">
    <template #option="{ option }">
      <div class="custom-option">
        <Avatar :image="option.avatar" size="small" />
        <span class="name">{{ option.label }}</span>
        <span class="desc">{{ option.description }}</span>
      </div>
    </template>
  </Select>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Select, Avatar } from "tdesign-vue-next";

const selectedValue = ref("");
const options = ref([
  {
    label: "张三",
    value: 1,
    avatar: "/avatar1.png",
    description: "前端工程师",
  },
  {
    label: "李四",
    value: 2,
    avatar: "/avatar2.png",
    description: "后端工程师",
  },
]);
</script>

<style scoped>
.custom-option {
  display: flex;
  align-items: center;
  gap: 8px;
}
.name {
  font-weight: 500;
}
.desc {
  color: var(--td-text-color-secondary);
  font-size: 12px;
}
</style>
```

### 5. 多选限制与标签管理

```vue
<template>
  <Select
    v-model="selectedValues"
    multiple
    :max="3"
    :min-collapsed-num="2"
    :options="options"
    :tag-props="{ closable: true }"
    placeholder="最多选择 3 项"
    @exceed="onExceed"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Select, MessagePlugin } from "tdesign-vue-next";

const selectedValues = ref<number[]>([]);

const options = ref([
  { label: "选项 1", value: 1 },
  { label: "选项 2", value: 2 },
  { label: "选项 3", value: 3 },
  { label: "选项 4", value: 4 },
  { label: "选项 5", value: 5 },
]);

const onExceed = () => {
  MessagePlugin.warning("最多只能选择 3 项");
};
</script>
```

### 6. 创建新选项

```vue
<template>
  <Select
    v-model="selectedValue"
    filterable
    creatable
    :options="options"
    placeholder="输入或选择"
    @create="onCreate"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import type { SelectOption } from "tdesign-vue-next";

const selectedValue = ref("");
const options = ref<SelectOption[]>([
  { label: "选项 1", value: "option1" },
  { label: "选项 2", value: "option2" },
]);

const onCreate = (value: string) => {
  // 添加新选项
  options.value.push({
    label: value,
    value: value,
  });
};
</script>
```

### 7. 分组选项

```vue
<template>
  <Select v-model="selectedValue" :options="groupedOptions" />
</template>

<script setup lang="ts">
import { ref } from "vue";
import type { SelectOption } from "tdesign-vue-next";

const selectedValue = ref("");

const groupedOptions = ref<SelectOption[]>([
  {
    label: "华东地区",
    group: "east",
    children: [
      { label: "上海", value: "shanghai" },
      { label: "杭州", value: "hangzhou" },
      { label: "南京", value: "nanjing" },
    ],
  },
  {
    label: "华南地区",
    group: "south",
    children: [
      { label: "广州", value: "guangzhou" },
      { label: "深圳", value: "shenzhen" },
    ],
  },
]);
</script>
```

### 8. 联动选择器

```vue
<template>
  <Space>
    <Select
      v-model="province"
      :options="provinceOptions"
      placeholder="选择省份"
      @change="onProvinceChange"
    />
    <Select
      v-model="city"
      :options="cityOptions"
      :disabled="!province"
      placeholder="选择城市"
    />
  </Space>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { Select, Space } from "tdesign-vue-next";

const province = ref("");
const city = ref("");

const provinceOptions = ref([
  { label: "广东省", value: "guangdong" },
  { label: "浙江省", value: "zhejiang" },
]);

const cityData: Record<string, Array<{ label: string; value: string }>> = {
  guangdong: [
    { label: "广州", value: "guangzhou" },
    { label: "深圳", value: "shenzhen" },
  ],
  zhejiang: [
    { label: "杭州", value: "hangzhou" },
    { label: "宁波", value: "ningbo" },
  ],
};

const cityOptions = computed(() => {
  return province.value ? cityData[province.value] || [] : [];
});

const onProvinceChange = () => {
  city.value = ""; // 清空城市选择
};
</script>
```

### 9. 带前缀/后缀图标

```vue
<template>
  <Select v-model="selectedValue" :options="options">
    <template #prefixIcon>
      <SearchIcon />
    </template>
  </Select>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Select } from "tdesign-vue-next";
import { SearchIcon } from "tdesign-icons-vue-next";

const selectedValue = ref("");
const options = ref([
  { label: "选项 1", value: 1 },
  { label: "选项 2", value: 2 },
]);
</script>
```

## 常见问题与建议

### Q: 远程搜索首次展开无数据？

初始化时加载一批默认选项，或在 `onVisibleChange` 时触发首次加载。

### Q: 多选时标签过多导致换行？

使用 `minCollapsedNum` 设置最小折叠数量，超出部分显示为 `+N`。

### Q: 如何在选项中显示更多信息？

使用 `option` 插槽自定义渲染，或使用 `content` 属性。

### Q: 如何清除选择？

设置 `clearable` 属性，用户可点击清除图标。

### Q: 键盘导航如何使用？

Select 默认支持键盘导航：

- `↑` `↓` 切换选项
- `Enter` 选中当前项
- `Esc` 关闭下拉

## 最小示例

```vue
<template>
  <Select
    v-model="selectedValue"
    :options="options"
    placeholder="请选择"
    clearable
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Select } from "tdesign-vue-next";

const selectedValue = ref("");

const options = ref([
  { label: "选项 1", value: 1 },
  { label: "选项 2", value: 2 },
  { label: "选项 3", value: 3 },
]);
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"简单下拉选择"，回到主 Skill 的组件选型规则
- 若涉及树形选择，查阅 `tree-advanced.md` 或使用 `TreeSelect` 组件
- 若涉及级联选择，查阅 `cascader-advanced.md`

## 参考文档

- Select 组件文档：https://tdesign.tencent.com/vue-next/components/select
