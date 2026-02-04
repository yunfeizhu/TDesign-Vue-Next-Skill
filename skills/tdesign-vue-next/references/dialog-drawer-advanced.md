# 弹窗/抽屉（Dialog/Drawer）复杂场景参考

## 适用场景边界

- 嵌套弹窗/抽屉（多层弹出）
- 命令式调用（非模板声明）
- 关闭拦截（异步确认）
- 弹窗内表单校验
- 自定义页脚按钮
- 全屏与拖拽
- destroyOnClose 与状态保持

## 推荐模式

### 1. 使用 v-model:visible 双向绑定

```vue
<Dialog v-model:visible="visible" @confirm="onConfirm">
  <p>内容</p>
</Dialog>
```

### 2. 命令式调用使用 DialogPlugin

```ts
import { DialogPlugin } from "tdesign-vue-next";

const dialog = DialogPlugin.confirm({
  header: "确认",
  body: "确定要删除吗？",
  onConfirm: () => dialog.hide(),
});
```

## 必须避免的反模式

1. **不控制 visible 状态**：会导致弹窗无法关闭
2. **嵌套弹窗共用同一个 visible**：每个弹窗需要独立状态
3. **在 onClose 中直接操作 DOM**：应该通过状态控制
4. **忘记 destroyOnClose 导致内存泄漏**：长列表或复杂内容需要销毁

## 常见实践

### 1. 基础 Dialog 用法

```vue
<template>
  <Button @click="visible = true">打开弹窗</Button>

  <Dialog
    v-model:visible="visible"
    header="弹窗标题"
    :confirm-btn="confirmBtnProps"
    :cancel-btn="cancelBtnProps"
    @confirm="onConfirm"
    @close="onClose"
  >
    <p>这是弹窗内容</p>
  </Dialog>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Dialog, Button, MessagePlugin } from "tdesign-vue-next";
import type { DialogProps } from "tdesign-vue-next";

const visible = ref(false);

const confirmBtnProps: DialogProps["confirmBtn"] = {
  content: "确定",
  theme: "primary",
};

const cancelBtnProps: DialogProps["cancelBtn"] = {
  content: "取消",
  theme: "default",
};

const onConfirm = () => {
  MessagePlugin.success("确认操作");
  visible.value = false;
};

const onClose = (context: { e: MouseEvent; trigger: string }) => {
  console.log("关闭触发方式:", context.trigger);
  // trigger: 'close-btn' | 'overlay' | 'esc' | 'cancel'
};
</script>
```

### 2. 关闭拦截（异步确认）

```vue
<template>
  <Button @click="visible = true">打开弹窗</Button>

  <Dialog v-model:visible="visible" header="编辑内容" :on-close="onBeforeClose">
    <Form :model="formData">
      <FormItem label="名称" name="name">
        <Input v-model="formData.name" />
      </FormItem>
    </Form>
  </Dialog>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import {
  Dialog,
  Button,
  Form,
  FormItem,
  Input,
  DialogPlugin,
} from "tdesign-vue-next";
import type { DialogCloseContext } from "tdesign-vue-next";

const visible = ref(false);
const formData = reactive({ name: "" });
const originalData = ref({ name: "" });

const hasChanges = () => {
  return formData.name !== originalData.value.name;
};

const onBeforeClose = (
  context: DialogCloseContext,
): boolean | Promise<boolean> => {
  const { trigger } = context;

  // 点击确认按钮时不拦截
  if (trigger === "confirm") {
    return true;
  }

  // 有未保存的更改时提示
  if (hasChanges()) {
    return new Promise((resolve) => {
      const confirmDialog = DialogPlugin.confirm({
        header: "提示",
        body: "有未保存的更改，确定要关闭吗？",
        onConfirm: () => {
          confirmDialog.hide();
          resolve(true);
        },
        onCancel: () => {
          confirmDialog.hide();
          resolve(false);
        },
      });
    });
  }

  return true;
};
</script>
```

### 3. 弹窗内表单校验

```vue
<template>
  <Button @click="visible = true">打开表单弹窗</Button>

  <Dialog
    v-model:visible="visible"
    header="新建用户"
    :confirm-btn="{ loading: submitting }"
    @confirm="onSubmit"
  >
    <Form ref="formRef" :model="formData" :rules="rules">
      <FormItem label="用户名" name="username">
        <Input v-model="formData.username" />
      </FormItem>
      <FormItem label="邮箱" name="email">
        <Input v-model="formData.email" />
      </FormItem>
    </Form>
  </Dialog>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from "vue";
import {
  Dialog,
  Button,
  Form,
  FormItem,
  Input,
  MessagePlugin,
} from "tdesign-vue-next";
import type { FormInstanceFunctions, FormRules } from "tdesign-vue-next";

const visible = ref(false);
const submitting = ref(false);
const formRef = ref<FormInstanceFunctions>();

const formData = reactive({
  username: "",
  email: "",
});

const rules: FormRules<typeof formData> = {
  username: [{ required: true, message: "用户名必填" }],
  email: [
    { required: true, message: "邮箱必填" },
    { email: true, message: "请输入有效的邮箱" },
  ],
};

// 打开弹窗时重置表单
watch(visible, (val) => {
  if (val) {
    formData.username = "";
    formData.email = "";
    formRef.value?.clearValidate();
  }
});

const onSubmit = async () => {
  const result = await formRef.value?.validate();

  if (result !== true) {
    return;
  }

  submitting.value = true;
  try {
    // 模拟提交
    await new Promise((resolve) => setTimeout(resolve, 1000));
    MessagePlugin.success("创建成功");
    visible.value = false;
  } catch (error) {
    MessagePlugin.error("创建失败");
  } finally {
    submitting.value = false;
  }
};
</script>
```

### 4. 命令式调用

```vue
<script setup lang="ts">
import { DialogPlugin, MessagePlugin } from "tdesign-vue-next";
import { h } from "vue";

const showConfirmDialog = () => {
  const dialog = DialogPlugin.confirm({
    header: "删除确认",
    body: "确定要删除这条记录吗？此操作不可恢复。",
    theme: "danger",
    confirmBtn: {
      content: "删除",
      theme: "danger",
    },
    onConfirm: async () => {
      dialog.update({ confirmBtn: { loading: true } });

      try {
        // 模拟删除操作
        await new Promise((resolve) => setTimeout(resolve, 1000));
        MessagePlugin.success("删除成功");
        dialog.hide();
      } catch (error) {
        MessagePlugin.error("删除失败");
        dialog.update({ confirmBtn: { loading: false } });
      }
    },
    onCancel: () => {
      dialog.hide();
    },
  });
};

const showAlertDialog = () => {
  DialogPlugin.alert({
    header: "提示",
    body: "操作已完成",
  });
};

const showCustomDialog = () => {
  const dialog = DialogPlugin({
    header: "自定义内容",
    body: () =>
      h("div", { style: { padding: "20px" } }, [
        h("p", "这是自定义渲染的内容"),
        h(
          "p",
          { style: { color: "var(--td-text-color-secondary)" } },
          "支持使用渲染函数",
        ),
      ]),
    footer: false, // 不显示默认底部按钮
  });

  // 3秒后自动关闭
  setTimeout(() => dialog.hide(), 3000);
};
</script>
```

### 5. 嵌套弹窗

```vue
<template>
  <Button @click="outerVisible = true">打开外层弹窗</Button>

  <!-- 外层弹窗 -->
  <Dialog
    v-model:visible="outerVisible"
    header="外层弹窗"
    width="600px"
    :z-index="1000"
  >
    <p>这是外层弹窗内容</p>
    <Button @click="innerVisible = true">打开内层弹窗</Button>

    <!-- 内层弹窗 -->
    <Dialog
      v-model:visible="innerVisible"
      header="内层弹窗"
      width="400px"
      :z-index="1001"
      :attach="false"
    >
      <p>这是内层弹窗内容</p>
    </Dialog>
  </Dialog>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Dialog, Button } from "tdesign-vue-next";

const outerVisible = ref(false);
const innerVisible = ref(false);
</script>
```

### 6. Drawer 基础用法

```vue
<template>
  <Space>
    <Button @click="openDrawer('right')">右侧抽屉</Button>
    <Button @click="openDrawer('left')">左侧抽屉</Button>
    <Button @click="openDrawer('top')">顶部抽屉</Button>
    <Button @click="openDrawer('bottom')">底部抽屉</Button>
  </Space>

  <Drawer
    v-model:visible="visible"
    :header="header"
    :placement="placement"
    :size="size"
    @confirm="onConfirm"
    @close="onClose"
  >
    <p>抽屉内容...</p>
  </Drawer>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Drawer, Button, Space } from "tdesign-vue-next";
import type { DrawerProps } from "tdesign-vue-next";

const visible = ref(false);
const placement = ref<DrawerProps["placement"]>("right");
const header = ref("抽屉标题");

const size = ref("400px");

const openDrawer = (pos: DrawerProps["placement"]) => {
  placement.value = pos;
  visible.value = true;

  // 根据方向设置尺寸
  if (pos === "top" || pos === "bottom") {
    size.value = "300px";
  } else {
    size.value = "400px";
  }
};

const onConfirm = () => {
  console.log("确认");
  visible.value = false;
};

const onClose = () => {
  console.log("关闭");
};
</script>
```

### 7. Drawer 内的列表与详情

```vue
<template>
  <Table :data="tableData" :columns="columns" row-key="id">
    <template #operation="{ row }">
      <Button variant="text" @click="showDetail(row)">查看详情</Button>
    </template>
  </Table>

  <Drawer
    v-model:visible="drawerVisible"
    :header="`${currentRow?.name} 的详情`"
    size="500px"
    :destroy-on-close="true"
  >
    <template v-if="currentRow">
      <Descriptions :items="detailItems" :column="1" />
    </template>
  </Drawer>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { Table, Button, Drawer, Descriptions } from "tdesign-vue-next";
import type { DescriptionsProps } from "tdesign-vue-next";

interface RowData {
  id: number;
  name: string;
  email: string;
  phone: string;
  address: string;
}

const tableData = ref<RowData[]>([
  {
    id: 1,
    name: "张三",
    email: "zhang@example.com",
    phone: "13800138000",
    address: "北京市朝阳区",
  },
  {
    id: 2,
    name: "李四",
    email: "li@example.com",
    phone: "13800138001",
    address: "上海市浦东新区",
  },
]);

const columns = computed(() => [
  { colKey: "name", title: "姓名" },
  { colKey: "email", title: "邮箱" },
  { colKey: "operation", title: "操作", width: 120 },
]);

const drawerVisible = ref(false);
const currentRow = ref<RowData | null>(null);

const detailItems = computed<DescriptionsProps["items"]>(() => {
  if (!currentRow.value) return [];
  return [
    { label: "姓名", content: currentRow.value.name },
    { label: "邮箱", content: currentRow.value.email },
    { label: "电话", content: currentRow.value.phone },
    { label: "地址", content: currentRow.value.address },
  ];
});

const showDetail = (row: RowData) => {
  currentRow.value = row;
  drawerVisible.value = true;
};
</script>
```

### 8. destroyOnClose 与状态保持

```vue
<template>
  <Space>
    <Button @click="visibleDestroy = true">销毁模式</Button>
    <Button @click="visibleKeep = true">保持模式</Button>
  </Space>

  <!-- 关闭时销毁内容 -->
  <Dialog
    v-model:visible="visibleDestroy"
    header="销毁模式"
    :destroy-on-close="true"
  >
    <ExpensiveComponent />
  </Dialog>

  <!-- 关闭时保持内容 -->
  <Dialog
    v-model:visible="visibleKeep"
    header="保持模式"
    :destroy-on-close="false"
  >
    <FormWithState />
  </Dialog>
</template>

<script setup lang="ts">
import { ref, defineComponent, h, reactive, onMounted } from "vue";
import { Dialog, Button, Space, Input, Form, FormItem } from "tdesign-vue-next";

const visibleDestroy = ref(false);
const visibleKeep = ref(false);

// 模拟复杂组件
const ExpensiveComponent = defineComponent({
  setup() {
    onMounted(() => {
      console.log("ExpensiveComponent 挂载");
    });
    return () => h("div", "每次打开都会重新初始化");
  },
});

// 模拟带状态的表单
const FormWithState = defineComponent({
  setup() {
    const formData = reactive({ value: "" });
    return () =>
      h(Form, { model: formData }, () => [
        h(FormItem, { label: "输入内容" }, () =>
          h(Input, {
            modelValue: formData.value,
            "onUpdate:modelValue": (v: string) => (formData.value = v),
            placeholder: "关闭后内容会保留",
          }),
        ),
      ]);
  },
});
</script>
```

### 9. 全屏与拖拽 Dialog

```vue
<template>
  <Space>
    <Button @click="visible = true">可拖拽弹窗</Button>
    <Button @click="fullscreenVisible = true">全屏弹窗</Button>
  </Space>

  <!-- 可拖拽弹窗 -->
  <Dialog
    v-model:visible="visible"
    header="可拖拽弹窗"
    draggable
    :modal="false"
  >
    <p>可以拖拽标题栏移动弹窗</p>
  </Dialog>

  <!-- 全屏弹窗 -->
  <Dialog
    v-model:visible="fullscreenVisible"
    header="全屏弹窗"
    :fullscreen="isFullscreen"
  >
    <template #header>
      <div class="custom-header">
        <span>全屏弹窗</span>
        <Button variant="text" @click="isFullscreen = !isFullscreen">
          {{ isFullscreen ? "退出全屏" : "全屏" }}
        </Button>
      </div>
    </template>
    <p>点击标题栏按钮切换全屏状态</p>
  </Dialog>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Dialog, Button, Space } from "tdesign-vue-next";

const visible = ref(false);
const fullscreenVisible = ref(false);
const isFullscreen = ref(false);
</script>

<style scoped>
.custom-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}
</style>
```

## 常见问题与建议

### Q: 弹窗打开后滚动条消失？

这是默认行为（防止背景滚动），可以通过 `preventScrollThrough` 控制：

```vue
<Dialog :prevent-scroll-through="false" />
```

### Q: 命令式调用的弹窗如何更新内容？

使用返回的实例方法：

```ts
const dialog = DialogPlugin({ body: "初始内容" });
dialog.update({ body: "更新后的内容" });
```

### Q: 如何在弹窗关闭动画完成后执行操作？

监听 `closed` 事件（注意不是 `close`）：

```vue
<Dialog @closed="onClosed" />
```

### Q: z-index 冲突怎么处理？

使用 `z-index` prop 手动设置，或通过 ConfigProvider 统一配置：

```vue
<ConfigProvider :global-config="{ classPrefix: 't', zIndex: { dialog: 2000 } }">
```

## 最小示例

```vue
<template>
  <Button @click="visible = true">打开弹窗</Button>
  <Dialog v-model:visible="visible" header="标题">
    <p>内容</p>
  </Dialog>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Dialog, Button } from "tdesign-vue-next";

const visible = ref(false);
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"简单弹窗/抽屉展示"，回到主 Skill 的组件选型规则
- 若涉及弹窗内的表单校验，查阅 `form-advanced.md`

## 参考文档

- Dialog 组件文档：https://tdesign.tencent.com/vue-next/components/dialog
- Drawer 组件文档：https://tdesign.tencent.com/vue-next/components/drawer
