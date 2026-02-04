# 级联选择（Cascader）复杂场景参考

## 适用场景边界

- 异步加载选项（懒加载子级）
- 动态选项（根据上级选择动态变化）
- 自定义面板渲染
- 多选级联
- 选择任意一级
- 搜索过滤
- 自定义触发器

## 推荐模式

### 1. 异步加载使用 load 函数

`load` 函数返回 Promise，用于懒加载子级选项：

```ts
const load: CascaderProps["load"] = async (node) => {
  const children = await fetchChildren(node.value);
  return children;
};
```

### 2. 使用 keys 映射数据字段

确保 `keys` 配置与数据结构匹配：

```vue
<Cascader
  :options="options"
  :keys="{ value: 'id', label: 'name', children: 'subItems' }"
/>
```

## 必须避免的反模式

1. **异步加载不返回 Promise**：导致加载状态异常
2. **修改选项时直接修改原数组**：应该使用新数组引用
3. **多级选项 value 重复**：会导致选中状态异常
4. **不设置 checkStrictly 却期望选择任意级别**

## 常见实践

### 1. 基础用法

```vue
<template>
  <Cascader
    v-model="selectedValue"
    :options="options"
    placeholder="请选择"
    clearable
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Cascader } from "tdesign-vue-next";

const selectedValue = ref<string[]>([]);

const options = ref([
  {
    value: "zhejiang",
    label: "浙江省",
    children: [
      {
        value: "hangzhou",
        label: "杭州市",
        children: [
          { value: "xihu", label: "西湖区" },
          { value: "yuhang", label: "余杭区" },
        ],
      },
      {
        value: "ningbo",
        label: "宁波市",
        children: [
          { value: "haishu", label: "海曙区" },
          { value: "jiangbei", label: "江北区" },
        ],
      },
    ],
  },
  {
    value: "jiangsu",
    label: "江苏省",
    children: [
      {
        value: "nanjing",
        label: "南京市",
        children: [
          { value: "xuanwu", label: "玄武区" },
          { value: "qinhuai", label: "秦淮区" },
        ],
      },
    ],
  },
]);
</script>
```

### 2. 异步加载选项

```vue
<template>
  <Cascader
    v-model="selectedValue"
    :options="options"
    :load="loadOptions"
    :keys="{ value: 'code', label: 'name', children: 'children' }"
    placeholder="请选择地区"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Cascader } from "tdesign-vue-next";
import type { CascaderProps, CascaderOption } from "tdesign-vue-next";

interface RegionItem {
  code: string;
  name: string;
  children?: RegionItem[] | boolean;
}

const selectedValue = ref<string[]>([]);

// 初始只加载省级数据
const options = ref<RegionItem[]>([
  { code: "330000", name: "浙江省", children: true }, // children: true 表示有子级需要加载
  { code: "320000", name: "江苏省", children: true },
  { code: "440000", name: "广东省", children: true },
]);

const loadOptions: CascaderProps["load"] = async (
  node,
): Promise<CascaderOption[]> => {
  console.log("加载子级", node.value);

  // 模拟 API 请求
  await new Promise((resolve) => setTimeout(resolve, 800));

  // 根据层级返回不同数据
  const level = node.getPath().length;

  if (level === 1) {
    // 加载市级数据
    return [
      { code: `${node.value}01`, name: "市 1", children: true },
      { code: `${node.value}02`, name: "市 2", children: true },
    ];
  }

  // 加载区级数据（叶子节点不设置 children）
  return [
    { code: `${node.value}01`, name: "区 1" },
    { code: `${node.value}02`, name: "区 2" },
  ];
};
</script>
```

### 3. 选择任意一级

```vue
<template>
  <Cascader
    v-model="selectedValue"
    :options="options"
    :check-strictly="true"
    placeholder="可选择任意级别"
    @change="onChange"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Cascader } from "tdesign-vue-next";
import type { CascaderValue, CascaderChangeContext } from "tdesign-vue-next";

const selectedValue = ref<string[]>([]);

const options = ref([
  {
    value: "fruit",
    label: "水果",
    children: [
      { value: "apple", label: "苹果" },
      { value: "banana", label: "香蕉" },
    ],
  },
  {
    value: "vegetable",
    label: "蔬菜",
    children: [
      { value: "cabbage", label: "白菜" },
      { value: "carrot", label: "胡萝卜" },
    ],
  },
]);

const onChange = (
  value: CascaderValue,
  context: CascaderChangeContext<any>,
) => {
  console.log("选中值", value);
  console.log("选中节点", context.node);
};
</script>
```

### 4. 多选级联

```vue
<template>
  <Cascader
    v-model="selectedValues"
    :options="options"
    multiple
    :min-collapsed-num="2"
    :max="5"
    placeholder="请选择（最多5项）"
    @change="onChange"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Cascader, MessagePlugin } from "tdesign-vue-next";
import type { CascaderValue } from "tdesign-vue-next";

const selectedValues = ref<CascaderValue>([]);

const options = ref([
  {
    value: "category1",
    label: "分类 1",
    children: [
      { value: "item1-1", label: "项目 1-1" },
      { value: "item1-2", label: "项目 1-2" },
    ],
  },
  {
    value: "category2",
    label: "分类 2",
    children: [
      { value: "item2-1", label: "项目 2-1" },
      { value: "item2-2", label: "项目 2-2" },
    ],
  },
]);

const onChange = (value: CascaderValue) => {
  if (Array.isArray(value) && value.length > 5) {
    MessagePlugin.warning("最多选择5项");
  }
};
</script>
```

### 5. 搜索过滤

```vue
<template>
  <Cascader
    v-model="selectedValue"
    :options="options"
    filterable
    :filter="customFilter"
    placeholder="支持搜索"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Cascader } from "tdesign-vue-next";
import type { CascaderProps } from "tdesign-vue-next";

const selectedValue = ref<string[]>([]);

const options = ref([
  {
    value: "zhejiang",
    label: "浙江省",
    children: [
      {
        value: "hangzhou",
        label: "杭州市",
        children: [{ value: "xihu", label: "西湖区" }],
      },
    ],
  },
  {
    value: "guangdong",
    label: "广东省",
    children: [
      {
        value: "shenzhen",
        label: "深圳市",
        children: [{ value: "nanshan", label: "南山区" }],
      },
    ],
  },
]);

// 自定义过滤函数
const customFilter: CascaderProps["filter"] = (filterWords, node) => {
  const label = node.label?.toLowerCase() || "";
  const value = String(node.value).toLowerCase();
  const keyword = filterWords.toLowerCase();

  // 支持 label 和 value 搜索
  return label.includes(keyword) || value.includes(keyword);
};
</script>
```

### 6. 自定义面板

```vue
<template>
  <Cascader v-model="selectedValue" :options="options">
    <template #panel="{ cascader }">
      <div class="custom-panel">
        <div class="panel-header">自定义面板头部</div>
        <component :is="cascader" />
        <div class="panel-footer">
          <Button size="small" @click="clearSelection">清除</Button>
          <Button size="small" theme="primary" @click="confirmSelection">
            确认
          </Button>
        </div>
      </div>
    </template>
  </Cascader>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Cascader, Button } from "tdesign-vue-next";

const selectedValue = ref<string[]>([]);

const options = ref([
  {
    value: "1",
    label: "选项 1",
    children: [
      { value: "1-1", label: "选项 1-1" },
      { value: "1-2", label: "选项 1-2" },
    ],
  },
]);

const clearSelection = () => {
  selectedValue.value = [];
};

const confirmSelection = () => {
  console.log("确认选择", selectedValue.value);
};
</script>

<style scoped>
.custom-panel {
  display: flex;
  flex-direction: column;
}

.panel-header {
  padding: 8px 12px;
  border-bottom: 1px solid var(--td-border-level-1-color);
  font-weight: 500;
}

.panel-footer {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  padding: 8px 12px;
  border-top: 1px solid var(--td-border-level-1-color);
}
</style>
```

### 7. 自定义触发器

```vue
<template>
  <Cascader v-model="selectedValue" :options="options">
    <template #trigger="{ selectedOptions, visible }">
      <div :class="['custom-trigger', { active: visible }]">
        <span v-if="selectedOptions.length">
          {{ selectedOptions.map((opt) => opt.label).join(" / ") }}
        </span>
        <span v-else class="placeholder">请选择</span>
        <ChevronDownIcon :class="['icon', { rotate: visible }]" />
      </div>
    </template>
  </Cascader>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Cascader } from "tdesign-vue-next";
import { ChevronDownIcon } from "tdesign-icons-vue-next";

const selectedValue = ref<string[]>([]);

const options = ref([
  {
    value: "1",
    label: "选项 1",
    children: [{ value: "1-1", label: "选项 1-1" }],
  },
]);
</script>

<style scoped>
.custom-trigger {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 12px;
  border: 1px solid var(--td-border-level-2-color);
  border-radius: var(--td-radius-default);
  cursor: pointer;
  min-width: 200px;
}

.custom-trigger.active {
  border-color: var(--td-brand-color);
}

.placeholder {
  color: var(--td-text-color-placeholder);
}

.icon {
  transition: transform 0.2s;
}

.icon.rotate {
  transform: rotate(180deg);
}
</style>
```

### 8. 联动更新选项

```vue
<template>
  <Space>
    <Select
      v-model="category"
      :options="categories"
      @change="onCategoryChange"
    />
    <Cascader
      v-model="selectedValue"
      :options="cascaderOptions"
      :disabled="!category"
      placeholder="请先选择类别"
    />
  </Space>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { Cascader, Select, Space } from "tdesign-vue-next";

const category = ref("");
const selectedValue = ref<string[]>([]);

const categories = [
  { label: "电子产品", value: "electronics" },
  { label: "服装", value: "clothing" },
];

const optionsMap: Record<string, any[]> = {
  electronics: [
    {
      value: "phone",
      label: "手机",
      children: [
        { value: "iphone", label: "iPhone" },
        { value: "android", label: "Android" },
      ],
    },
    {
      value: "computer",
      label: "电脑",
      children: [
        { value: "laptop", label: "笔记本" },
        { value: "desktop", label: "台式机" },
      ],
    },
  ],
  clothing: [
    {
      value: "men",
      label: "男装",
      children: [
        { value: "shirt", label: "衬衫" },
        { value: "pants", label: "裤子" },
      ],
    },
    {
      value: "women",
      label: "女装",
      children: [
        { value: "dress", label: "连衣裙" },
        { value: "skirt", label: "半身裙" },
      ],
    },
  ],
};

const cascaderOptions = computed(() => optionsMap[category.value] || []);

const onCategoryChange = () => {
  // 切换类别时清空级联选择
  selectedValue.value = [];
};
</script>
```

## 常见问题与建议

### Q: 异步加载后选项不显示？

确保返回的数据格式正确，且 `keys` 配置与数据字段匹配。

### Q: 多选时如何限制选择数量？

使用 `max` 属性限制最大选择数量，并监听 `change` 事件给出提示。

### Q: 如何获取完整的选中路径？

使用 `change` 事件的 `context.selectedOptions` 获取完整路径：

```ts
const onChange = (value, context) => {
  const path = context.selectedOptions.map((opt) => opt.label).join("/");
  console.log("路径:", path);
};
```

### Q: 如何实现省市区三级联动？

使用 `load` 函数配合后端接口实现懒加载，避免一次性加载全部数据。

## 最小示例

```vue
<template>
  <Cascader v-model="selectedValue" :options="options" placeholder="请选择" />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Cascader } from "tdesign-vue-next";

const selectedValue = ref<string[]>([]);

const options = ref([
  {
    value: "1",
    label: "选项 1",
    children: [
      { value: "1-1", label: "选项 1-1" },
      { value: "1-2", label: "选项 1-2" },
    ],
  },
  {
    value: "2",
    label: "选项 2",
    children: [{ value: "2-1", label: "选项 2-1" }],
  },
]);
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"简单级联选择"，回到主 Skill 的组件选型规则
- 若涉及树形选择，使用 `Tree` 或 `TreeSelect` 组件，查阅 `tree-advanced.md`
- 若涉及表单中的级联校验，查阅 `form-advanced.md`

## 参考文档

- Cascader 组件文档：https://tdesign.tencent.com/vue-next/components/cascader
