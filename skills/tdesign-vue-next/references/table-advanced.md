# 表格（Table）复杂场景参考

## 适用场景边界

- 大数据量表格、虚拟滚动
- 服务端排序/筛选/分页
- 可编辑表格（单元格编辑、行编辑）
- 树形表格
- 固定列/表头
- 行选择、批量操作
- 可展开行
- 自定义渲染、合并单元格

## 推荐模式

### 1. 稳定的 `row-key`

**必须**为每行数据提供稳定且唯一的 `row-key`，这是表格性能和功能正常工作的基础。

```vue
<Table :data="data" row-key="id" :columns="columns" />
```

### 2. 使用 computed 缓存 columns

列配置应使用 `computed` 缓存，避免每次渲染都创建新数组导致性能问题。

```ts
const columns = computed(() => [
  { colKey: "name", title: "姓名" },
  { colKey: "age", title: "年龄" },
]);
```

### 3. 受控状态统一管理

分页、排序、筛选状态通过 `v-model` 或事件统一管理：

```vue
<Table
  :data="data"
  :columns="columns"
  :pagination="pagination"
  :sort="sort"
  @page-change="onPageChange"
  @sort-change="onSortChange"
/>
```

## 必须避免的反模式

1. **不设置 `row-key`**：会导致行选择、展开等功能异常
2. **columns 每次渲染都创建新数组**：严重影响性能
3. **在大数据量场景不启用虚拟滚动**：会导致页面卡顿
4. **混用受控和非受控状态**：导致状态不一致

## 常见实践

### 1. 服务端分页排序筛选

```vue
<template>
  <Table
    :data="tableData"
    :columns="columns"
    :pagination="pagination"
    :sort="sortInfo"
    :filter-value="filterValue"
    :loading="loading"
    row-key="id"
    @page-change="onPageChange"
    @sort-change="onSortChange"
    @filter-change="onFilterChange"
  />
</template>

<script setup lang="ts">
import { ref, reactive, watch, computed } from "vue";
import { Table } from "tdesign-vue-next";
import type {
  TableSort,
  PageInfo,
  FilterValue,
  SortChangeContext,
  PageChangeContext,
  FilterChangeContext,
} from "tdesign-vue-next";

const tableData = ref([]);
const loading = ref(false);

// 分页配置
const pagination = reactive({
  current: 1,
  pageSize: 10,
  total: 0,
});

// 排序配置
const sortInfo = ref<TableSort>({ sortBy: "id", descending: false });

// 筛选配置
const filterValue = ref<FilterValue>({});

// 列配置
const columns = computed(() => [
  { colKey: "id", title: "ID", sortType: "all" as const },
  { colKey: "name", title: "姓名", filter: { type: "input" as const } },
  {
    colKey: "status",
    title: "状态",
    filter: {
      type: "single" as const,
      list: [
        { label: "启用", value: 1 },
        { label: "禁用", value: 0 },
      ],
    },
  },
]);

// 获取数据
const fetchData = async () => {
  loading.value = true;
  try {
    const params = {
      page: pagination.current,
      pageSize: pagination.pageSize,
      sortBy: sortInfo.value?.sortBy,
      descending: sortInfo.value?.descending,
      filters: filterValue.value,
    };

    const response = await fetch("/api/list", {
      method: "POST",
      body: JSON.stringify(params),
    });
    const { data, total } = await response.json();

    tableData.value = data;
    pagination.total = total;
  } finally {
    loading.value = false;
  }
};

// 分页变化
const onPageChange = (pageInfo: PageInfo, context: PageChangeContext) => {
  pagination.current = pageInfo.current;
  pagination.pageSize = pageInfo.pageSize;
  fetchData();
};

// 排序变化
const onSortChange = (sort: TableSort, context: SortChangeContext<any>) => {
  sortInfo.value = sort;
  fetchData();
};

// 筛选变化
const onFilterChange = (
  filters: FilterValue,
  context: FilterChangeContext<any>,
) => {
  filterValue.value = filters;
  pagination.current = 1; // 重置到第一页
  fetchData();
};

// 初始加载
fetchData();
</script>
```

### 2. 行选择与批量操作

```vue
<template>
  <Space direction="vertical" style="width: 100%">
    <Space>
      <Button :disabled="selectedRowKeys.length === 0" @click="batchDelete">
        批量删除 ({{ selectedRowKeys.length }})
      </Button>
    </Space>

    <Table
      :data="data"
      :columns="columns"
      row-key="id"
      :selected-row-keys="selectedRowKeys"
      @select-change="onSelectChange"
    />
  </Space>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import type { SelectChangeContext } from "tdesign-vue-next";

const data = ref([
  { id: 1, name: "张三" },
  { id: 2, name: "李四" },
]);

const selectedRowKeys = ref<(string | number)[]>([]);

const columns = computed(() => [
  { colKey: "row-select", type: "multiple" as const, width: 50 },
  { colKey: "id", title: "ID" },
  { colKey: "name", title: "姓名" },
]);

const onSelectChange = (
  selectedRowKeys: (string | number)[],
  context: SelectChangeContext<any>,
) => {
  selectedRowKeys.value = selectedRowKeys;
  console.log("选中的行", context.selectedRowData);
};

const batchDelete = () => {
  console.log("删除", selectedRowKeys.value);
};
</script>
```

### 3. 虚拟滚动

```vue
<template>
  <Table
    :data="largeData"
    :columns="columns"
    :scroll="{ type: 'virtual' }"
    :height="500"
    row-key="id"
  />
</template>

<script setup lang="ts">
import { ref, computed } from "vue";

// 模拟大数据量
const largeData = ref(
  Array.from({ length: 10000 }, (_, i) => ({
    id: i + 1,
    name: `用户 ${i + 1}`,
    age: Math.floor(Math.random() * 50) + 18,
  })),
);

const columns = computed(() => [
  { colKey: "id", title: "ID", width: 100 },
  { colKey: "name", title: "姓名", width: 200 },
  { colKey: "age", title: "年龄", width: 100 },
]);
</script>
```

### 4. 可编辑表格

```vue
<template>
  <Table
    :data="data"
    :columns="columns"
    row-key="id"
    :editable-row-keys="editableRowKeys"
    @row-edit="onRowEdit"
    @row-validate="onRowValidate"
  />
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import type {
  PrimaryTableCol,
  TableRowEditContext,
  TableRowValidateContext,
} from "tdesign-vue-next";

const data = ref([
  { id: 1, name: "张三", age: 25 },
  { id: 2, name: "李四", age: 30 },
]);

const editableRowKeys = ref<(string | number)[]>([1]); // 可编辑的行

const columns = computed<PrimaryTableCol[]>(() => [
  { colKey: "id", title: "ID", width: 100 },
  {
    colKey: "name",
    title: "姓名",
    edit: {
      component: "Input",
      props: { clearable: true },
      rules: [{ required: true, message: "姓名必填" }],
    },
  },
  {
    colKey: "age",
    title: "年龄",
    edit: {
      component: "InputNumber",
      props: { min: 0, max: 120 },
    },
  },
  {
    colKey: "operate",
    title: "操作",
    cell: (h, { row }) => {
      const isEditing = editableRowKeys.value.includes(row.id);
      if (isEditing) {
        return [
          h(
            "Button",
            { size: "small", onClick: () => saveRow(row.id) },
            "保存",
          ),
          h(
            "Button",
            { size: "small", onClick: () => cancelEdit(row.id) },
            "取消",
          ),
        ];
      }
      return h(
        "Button",
        { size: "small", onClick: () => editRow(row.id) },
        "编辑",
      );
    },
  },
]);

const editRow = (id: number) => {
  editableRowKeys.value = [...editableRowKeys.value, id];
};

const cancelEdit = (id: number) => {
  editableRowKeys.value = editableRowKeys.value.filter((key) => key !== id);
};

const saveRow = (id: number) => {
  // 验证后保存
  editableRowKeys.value = editableRowKeys.value.filter((key) => key !== id);
};

const onRowEdit = (context: TableRowEditContext<any>) => {
  console.log("行编辑", context);
};

const onRowValidate = (context: TableRowValidateContext<any>) => {
  console.log("行校验", context);
};
</script>
```

### 5. 树形表格

```vue
<template>
  <Table
    :data="treeData"
    :columns="columns"
    row-key="id"
    :tree="{ childrenKey: 'children', treeNodeColumnIndex: 0 }"
    :expanded-row-keys="expandedRowKeys"
    @expanded-row-keys-change="onExpandedChange"
  />
</template>

<script setup lang="ts">
import { ref, computed } from "vue";

const treeData = ref([
  {
    id: 1,
    name: "部门 A",
    children: [
      { id: 11, name: "小组 A-1" },
      { id: 12, name: "小组 A-2" },
    ],
  },
  {
    id: 2,
    name: "部门 B",
    children: [{ id: 21, name: "小组 B-1" }],
  },
]);

const expandedRowKeys = ref<(string | number)[]>([1]);

const columns = computed(() => [{ colKey: "name", title: "名称" }]);

const onExpandedChange = (keys: (string | number)[]) => {
  expandedRowKeys.value = keys;
};
</script>
```

### 6. 固定列和表头

```vue
<template>
  <Table
    :data="data"
    :columns="columns"
    row-key="id"
    :max-height="400"
    :scroll="{ x: 1500 }"
  />
</template>

<script setup lang="ts">
import { computed } from "vue";

const columns = computed(() => [
  { colKey: "id", title: "ID", fixed: "left" as const, width: 100 },
  { colKey: "name", title: "姓名", width: 200 },
  { colKey: "age", title: "年龄", width: 150 },
  { colKey: "address", title: "地址", width: 300 },
  { colKey: "email", title: "邮箱", width: 250 },
  { colKey: "phone", title: "电话", width: 200 },
  { colKey: "operate", title: "操作", fixed: "right" as const, width: 150 },
]);
</script>
```

### 7. 可展开行

```vue
<template>
  <Table
    :data="data"
    :columns="columns"
    row-key="id"
    :expand-on-row-click="true"
    :expanded-row="expandedRowRender"
  />
</template>

<script setup lang="ts">
import { h, computed } from "vue";

const data = ref([
  { id: 1, name: "张三", description: "这是张三的详细描述信息..." },
  { id: 2, name: "李四", description: "这是李四的详细描述信息..." },
]);

const columns = computed(() => [
  { colKey: "expand-icon", type: "expand" as const, width: 50 },
  { colKey: "id", title: "ID" },
  { colKey: "name", title: "姓名" },
]);

const expandedRowRender = ({ row }: { row: any }) => {
  return h("div", { style: { padding: "16px" } }, row.description);
};
</script>
```

### 8. 合并单元格

```vue
<template>
  <Table
    :data="data"
    :columns="columns"
    row-key="id"
    :row-spanned-row="rowspanHandler"
  />
</template>

<script setup lang="ts">
import { computed } from "vue";
import type { TableRowspanAndColspanFunc } from "tdesign-vue-next";

const data = ref([
  { id: 1, category: "水果", name: "苹果", price: 5 },
  { id: 2, category: "水果", name: "香蕉", price: 3 },
  { id: 3, category: "蔬菜", name: "白菜", price: 2 },
]);

const columns = computed(() => [
  { colKey: "category", title: "分类" },
  { colKey: "name", title: "名称" },
  { colKey: "price", title: "价格" },
]);

const rowspanHandler: TableRowspanAndColspanFunc<any> = ({ col, rowIndex }) => {
  if (col.colKey === "category") {
    if (rowIndex === 0) {
      return { rowspan: 2 };
    }
    if (rowIndex === 1) {
      return { rowspan: 0 };
    }
  }
  return {};
};
</script>
```

## 常见问题与建议

### Q: 虚拟滚动时行高不一致怎么办？

设置固定行高或使用 `rowHeight` 属性：

```vue
<Table :scroll="{ type: 'virtual', rowHeight: 48 }" />
```

### Q: 大数据量筛选性能差？

使用服务端筛选，禁用本地筛选逻辑。

### Q: 列宽自适应？

使用 `resizable` 开启列宽拖拽，或通过 `width` 设置合适的初始宽度。

### Q: 如何获取当前表格状态？

通过 ref 获取表格实例：

```ts
const tableRef = ref<InstanceType<typeof Table>>();
// 获取选中行
const selected = tableRef.value?.selectedRowKeys;
```

## 最小示例

```vue
<template>
  <Table
    :data="data"
    :columns="columns"
    row-key="id"
    :pagination="pagination"
  />
</template>

<script setup lang="ts">
import { ref, reactive, computed } from "vue";
import { Table } from "tdesign-vue-next";

const data = ref([
  { id: 1, name: "张三", age: 25 },
  { id: 2, name: "李四", age: 30 },
  { id: 3, name: "王五", age: 28 },
]);

const columns = computed(() => [
  { colKey: "id", title: "ID", width: 100 },
  { colKey: "name", title: "姓名", width: 200 },
  { colKey: "age", title: "年龄", width: 100 },
]);

const pagination = reactive({
  current: 1,
  pageSize: 10,
  total: 3,
});
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"是否使用 Table"或"简单列表展示"，回到主 Skill 的组件选型规则
- 若涉及 Table 内嵌 Form 校验，查阅 `form-advanced.md`

## 参考文档

- Table 组件文档：https://tdesign.tencent.com/vue-next/components/table
- EnhancedTable 组件文档：https://tdesign.tencent.com/vue-next/components/enhanced-table
