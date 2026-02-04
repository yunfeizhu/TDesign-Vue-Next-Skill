# 树组件（Tree）复杂场景参考

## 适用场景边界

- 异步加载节点（懒加载子节点）
- checkStrictly（父子节点选中状态不关联）
- 虚拟滚动（大数据量）
- 可拖拽排序
- 节点搜索过滤
- 自定义节点渲染
- 节点编辑（增删改）

## 推荐模式

### 1. 稳定的 keys 配置

确保 `keys` 配置与数据结构匹配：

```vue
<Tree
  :data="treeData"
  :keys="{ value: 'id', label: 'name', children: 'children' }"
/>
```

### 2. 异步加载使用 Promise

`load` 函数必须返回 Promise：

```ts
const load: TreeProps["load"] = async (node) => {
  const children = await fetchChildren(node.value);
  return children;
};
```

## 必须避免的反模式

1. **keys 配置与数据结构不匹配**：导致节点无法正确渲染
2. **异步加载不返回 Promise**：导致加载状态异常
3. **大数据量不启用虚拟滚动**：导致页面卡顿
4. **频繁修改整个 data 数组**：应使用实例方法操作节点

## 常见实践

### 1. 异步加载节点

```vue
<template>
  <Tree
    :data="treeData"
    :keys="keys"
    :load="loadChildren"
    :expand-on-click-node="false"
    @expand="onExpand"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Tree } from "tdesign-vue-next";
import type { TreeProps, TreeNodeModel, TreeNodeValue } from "tdesign-vue-next";

const keys = { value: "id", label: "name", children: "children" };

const treeData = ref([
  { id: "1", name: "节点 1", children: true }, // children: true 表示有子节点需要加载
  { id: "2", name: "节点 2", children: true },
]);

const loadChildren: TreeProps["load"] = async (node) => {
  // 模拟 API 请求
  await new Promise((resolve) => setTimeout(resolve, 1000));

  const parentId = node.value;
  return [
    { id: `${parentId}-1`, name: `${node.label} 的子节点 1` },
    { id: `${parentId}-2`, name: `${node.label} 的子节点 2`, children: true },
  ];
};

const onExpand = (
  value: TreeNodeValue[],
  context: { node: TreeNodeModel; e?: MouseEvent },
) => {
  console.log("展开节点", context.node);
};
</script>
```

### 2. 父子节点选中不关联（checkStrictly）

```vue
<template>
  <Tree
    v-model="checkedValues"
    :data="treeData"
    checkable
    :check-strictly="true"
    @change="onChange"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Tree } from "tdesign-vue-next";
import type { TreeNodeValue, TreeNodeState } from "tdesign-vue-next";

const checkedValues = ref<TreeNodeValue[]>([]);

const treeData = ref([
  {
    value: "1",
    label: "父节点",
    children: [
      { value: "1-1", label: "子节点 1" },
      { value: "1-2", label: "子节点 2" },
    ],
  },
]);

const onChange = (
  value: TreeNodeValue[],
  context: { node: TreeNodeState; e: Event },
) => {
  console.log("选中变化", value, context.node);
};
</script>
```

### 3. 虚拟滚动

```vue
<template>
  <Tree
    :data="largeTreeData"
    :scroll="{ type: 'virtual' }"
    :height="400"
    checkable
  />
</template>

<script setup lang="ts">
import { computed } from "vue";

// 生成大数据量树形结构
const generateTree = (level: number, parentKey: string = ""): any[] => {
  if (level === 0) return [];

  return Array.from({ length: 10 }, (_, i) => {
    const key = parentKey ? `${parentKey}-${i}` : String(i);
    return {
      value: key,
      label: `节点 ${key}`,
      children: generateTree(level - 1, key),
    };
  });
};

const largeTreeData = computed(() => generateTree(4)); // 10000+ 节点
</script>
```

### 4. 可拖拽排序

```vue
<template>
  <Tree
    :data="treeData"
    draggable
    @drag-start="onDragStart"
    @drag-end="onDragEnd"
    @drop="onDrop"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Tree, MessagePlugin } from "tdesign-vue-next";
import type {
  TreeNodeModel,
  TreeNodeValue,
  DropContext,
} from "tdesign-vue-next";

const treeData = ref([
  {
    value: "1",
    label: "节点 1",
    children: [
      { value: "1-1", label: "节点 1-1" },
      { value: "1-2", label: "节点 1-2" },
    ],
  },
  {
    value: "2",
    label: "节点 2",
  },
]);

const onDragStart = (context: { node: TreeNodeModel; e: DragEvent }) => {
  console.log("开始拖拽", context.node.label);
};

const onDragEnd = (context: { node: TreeNodeModel; e: DragEvent }) => {
  console.log("结束拖拽", context.node.label);
};

const onDrop = (context: DropContext<any>) => {
  const { dragNode, dropNode, dropPosition } = context;
  console.log(
    `将 ${dragNode?.label} 拖拽到 ${dropNode?.label} 的 ${dropPosition}`,
  );

  // 可以在这里调用 API 保存排序结果
  // await saveTreeOrder(context);

  MessagePlugin.success("拖拽完成");
};
</script>
```

### 5. 节点搜索过滤

```vue
<template>
  <Space direction="vertical" style="width: 100%">
    <Input v-model="keyword" placeholder="搜索节点" @change="onSearch" />
    <Tree
      ref="treeRef"
      :data="treeData"
      :filter="filterNode"
      :expand-all="!!keyword"
    />
  </Space>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Tree, Input, Space } from "tdesign-vue-next";
import type { TreeProps, TreeInstanceFunctions } from "tdesign-vue-next";

const keyword = ref("");
const treeRef = ref<TreeInstanceFunctions>();

const treeData = ref([
  {
    value: "1",
    label: "北京",
    children: [
      { value: "1-1", label: "海淀区" },
      { value: "1-2", label: "朝阳区" },
    ],
  },
  {
    value: "2",
    label: "上海",
    children: [
      { value: "2-1", label: "浦东新区" },
      { value: "2-2", label: "黄浦区" },
    ],
  },
]);

const filterNode: TreeProps["filter"] = (node) => {
  if (!keyword.value) return true;
  return node.label?.toLowerCase().includes(keyword.value.toLowerCase());
};

const onSearch = () => {
  // 触发过滤
  treeRef.value?.filter(keyword.value);
};
</script>
```

### 6. 自定义节点渲染

```vue
<template>
  <Tree :data="treeData">
    <template #label="{ node }">
      <div class="custom-node">
        <FolderIcon v-if="node.children?.length" />
        <FileIcon v-else />
        <span class="node-label">{{ node.label }}</span>
        <Space class="node-actions" size="small">
          <Button size="small" variant="text" @click.stop="onEdit(node)">
            <EditIcon />
          </Button>
          <Button size="small" variant="text" @click.stop="onDelete(node)">
            <DeleteIcon />
          </Button>
        </Space>
      </div>
    </template>
  </Tree>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Tree, Button, Space } from "tdesign-vue-next";
import {
  FolderIcon,
  FileIcon,
  EditIcon,
  DeleteIcon,
} from "tdesign-icons-vue-next";
import type { TreeNodeModel } from "tdesign-vue-next";

const treeData = ref([
  {
    value: "1",
    label: "文件夹 1",
    children: [
      { value: "1-1", label: "文件 1.txt" },
      { value: "1-2", label: "文件 2.txt" },
    ],
  },
]);

const onEdit = (node: TreeNodeModel) => {
  console.log("编辑节点", node);
};

const onDelete = (node: TreeNodeModel) => {
  console.log("删除节点", node);
};
</script>

<style scoped>
.custom-node {
  display: flex;
  align-items: center;
  gap: 4px;
  width: 100%;
}

.node-label {
  flex: 1;
}

.node-actions {
  opacity: 0;
  transition: opacity 0.2s;
}

.custom-node:hover .node-actions {
  opacity: 1;
}
</style>
```

### 7. 节点编辑操作

```vue
<template>
  <Space direction="vertical">
    <Space>
      <Button @click="appendNode">添加根节点</Button>
      <Button @click="appendChildNode">添加子节点到选中项</Button>
      <Button theme="danger" @click="removeSelectedNodes">删除选中节点</Button>
    </Space>
    <Tree
      ref="treeRef"
      v-model:expanded="expandedKeys"
      v-model:actived="activedKeys"
      :data="treeData"
      activable
    />
  </Space>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Tree, Button, Space } from "tdesign-vue-next";
import type { TreeInstanceFunctions, TreeNodeValue } from "tdesign-vue-next";

const treeRef = ref<TreeInstanceFunctions>();
const expandedKeys = ref<TreeNodeValue[]>([]);
const activedKeys = ref<TreeNodeValue[]>([]);

const treeData = ref([
  {
    value: "1",
    label: "节点 1",
    children: [{ value: "1-1", label: "节点 1-1" }],
  },
]);

let nodeId = 100;

const appendNode = () => {
  const newNode = {
    value: String(nodeId++),
    label: `新节点 ${nodeId}`,
  };
  treeRef.value?.appendTo("", newNode);
};

const appendChildNode = () => {
  if (activedKeys.value.length === 0) {
    return;
  }

  const parentKey = activedKeys.value[0];
  const newNode = {
    value: String(nodeId++),
    label: `子节点 ${nodeId}`,
  };

  treeRef.value?.appendTo(parentKey, newNode);

  // 展开父节点
  if (!expandedKeys.value.includes(parentKey)) {
    expandedKeys.value = [...expandedKeys.value, parentKey];
  }
};

const removeSelectedNodes = () => {
  activedKeys.value.forEach((key) => {
    treeRef.value?.remove(key);
  });
  activedKeys.value = [];
};
</script>
```

### 8. 单选树

```vue
<template>
  <Tree
    v-model:actived="selectedKey"
    :data="treeData"
    activable
    :active-multiple="false"
    @active="onActive"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Tree } from "tdesign-vue-next";
import type { TreeNodeValue, TreeNodeModel } from "tdesign-vue-next";

const selectedKey = ref<TreeNodeValue[]>([]);

const treeData = ref([
  {
    value: "1",
    label: "选项 1",
    children: [
      { value: "1-1", label: "选项 1-1" },
      { value: "1-2", label: "选项 1-2" },
    ],
  },
  { value: "2", label: "选项 2" },
]);

const onActive = (
  value: TreeNodeValue[],
  context: { node: TreeNodeModel; e?: MouseEvent },
) => {
  console.log("选中", value[0], context.node);
};
</script>
```

## 常见问题与建议

### Q: 异步加载后节点展开失败？

确保 `load` 函数返回正确格式的数据，且数据中包含 `keys` 配置中指定的字段。

### Q: checkStrictly 模式下如何获取半选状态？

使用 `getIndeterminate` 实例方法：

```ts
const indeterminateKeys = treeRef.value?.getIndeterminate();
```

### Q: 如何获取所有展开/选中的节点？

```ts
// 获取所有选中节点
const checkedNodes = treeRef.value?.getChecked({ includeHalfChecked: false });

// 获取所有展开节点
const expandedNodes = expandedKeys.value;
```

### Q: 如何定位到指定节点？

```ts
// 滚动到指定节点
treeRef.value?.scrollTo(nodeValue);
```

## 最小示例

```vue
<template>
  <Tree
    v-model="checkedValues"
    :data="treeData"
    checkable
    :default-expand-all="true"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Tree } from "tdesign-vue-next";
import type { TreeNodeValue } from "tdesign-vue-next";

const checkedValues = ref<TreeNodeValue[]>([]);

const treeData = ref([
  {
    value: "1",
    label: "节点 1",
    children: [
      { value: "1-1", label: "节点 1-1" },
      { value: "1-2", label: "节点 1-2" },
    ],
  },
  { value: "2", label: "节点 2" },
]);
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"简单树形展示"，回到主 Skill 的组件选型规则
- 若涉及树形选择器，使用 `TreeSelect` 组件
- 若涉及级联选择，查阅 `cascader-advanced.md`

## 参考文档

- Tree 组件文档：https://tdesign.tencent.com/vue-next/components/tree
- TreeSelect 组件文档：https://tdesign.tencent.com/vue-next/components/tree-select
