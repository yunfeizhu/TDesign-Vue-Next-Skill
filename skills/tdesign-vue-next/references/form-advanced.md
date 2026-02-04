# 表单（Form）复杂场景参考

## 适用场景边界

- 动态表单（动态增删 `FormItem`）
- 跨字段联动校验（字段 A 的值影响字段 B 的校验规则）
- 异步校验（如用户名唯一性检查）
- 复杂嵌套数据结构
- 表单值由外部状态驱动
- 分步表单 / 多表单联动

## 推荐模式

### 1. 表单作为数据源

Form 组件通过 `model` prop 绑定数据对象，使用 `v-model` 或直接修改 `model` 对象来更新表单值。

```vue
<template>
  <Form ref="formRef" :model="formData" :rules="rules">
    <FormItem label="用户名" name="username">
      <Input v-model="formData.username" />
    </FormItem>
  </Form>
</template>

<script setup lang="ts">
import { reactive, ref } from "vue";
import { Form, FormItem, Input } from "tdesign-vue-next";
import type { FormInstanceFunctions, FormRules } from "tdesign-vue-next";

const formRef = ref<FormInstanceFunctions>();
const formData = reactive({
  username: "",
});

const rules: FormRules<typeof formData> = {
  username: [{ required: true, message: "用户名必填" }],
};
</script>
```

### 2. 使用 name 属性进行嵌套字段绑定

对于嵌套数据结构，使用数组形式的 `name`：

```vue
<FormItem label="省份" :name="['address', 'province']">
  <Input v-model="formData.address.province" />
</FormItem>
```

## 必须避免的反模式

1. **不要在输入组件上同时使用 `v-model` 和 Form 的 `initialData`**：会导致状态不一致
2. **不要动态修改 `rules` 对象的引用**：使用 `computed` 或保持引用稳定
3. **不要在校验函数中执行副作用**：校验函数应该是纯函数
4. **不要忘记为动态表单项设置唯一的 `name`**：否则校验会出错

## 常见实践

### 1. 动态表单字段

```vue
<template>
  <Form :model="formData" :rules="rules">
    <FormItem
      v-for="(item, index) in formData.users"
      :key="index"
      :label="`用户 ${index + 1}`"
      :name="['users', index, 'name']"
      :rules="[{ required: true, message: '用户名必填' }]"
    >
      <Space>
        <Input v-model="item.name" />
        <Button theme="danger" @click="removeUser(index)">删除</Button>
      </Space>
    </FormItem>
    <FormItem>
      <Button @click="addUser">添加用户</Button>
    </FormItem>
  </Form>
</template>

<script setup lang="ts">
import { reactive } from "vue";
import { Form, FormItem, Input, Button, Space } from "tdesign-vue-next";

const formData = reactive({
  users: [{ name: "" }],
});

const rules = {};

const addUser = () => {
  formData.users.push({ name: "" });
};

const removeUser = (index: number) => {
  formData.users.splice(index, 1);
};
</script>
```

### 2. 跨字段联动校验

```vue
<template>
  <Form ref="formRef" :model="formData" :rules="rules">
    <FormItem label="密码" name="password">
      <Input v-model="formData.password" type="password" />
    </FormItem>
    <FormItem label="确认密码" name="confirmPassword">
      <Input v-model="formData.confirmPassword" type="password" />
    </FormItem>
  </Form>
</template>

<script setup lang="ts">
import { reactive, ref } from "vue";
import { Form, FormItem, Input } from "tdesign-vue-next";
import type {
  FormInstanceFunctions,
  FormRules,
  FormRule,
} from "tdesign-vue-next";

const formRef = ref<FormInstanceFunctions>();
const formData = reactive({
  password: "",
  confirmPassword: "",
});

const confirmPasswordValidator: FormRule["validator"] = (val) => {
  if (val !== formData.password) {
    return { result: false, message: "两次输入的密码不一致" };
  }
  return { result: true };
};

const rules: FormRules<typeof formData> = {
  password: [
    { required: true, message: "请输入密码" },
    { min: 6, message: "密码长度不能少于 6 位" },
  ],
  confirmPassword: [
    { required: true, message: "请确认密码" },
    { validator: confirmPasswordValidator },
  ],
};
</script>
```

### 3. 异步校验

```vue
<script setup lang="ts">
import type { FormRule } from "tdesign-vue-next";

const checkUsernameUnique: FormRule["validator"] = async (val) => {
  // 模拟 API 请求
  const response = await fetch(`/api/check-username?name=${val}`);
  const { available } = await response.json();

  if (!available) {
    return { result: false, message: "用户名已存在" };
  }
  return { result: true };
};

const rules = {
  username: [
    { required: true, message: "用户名必填" },
    { validator: checkUsernameUnique, trigger: "blur" },
  ],
};
</script>
```

### 4. 表单实例方法

```vue
<script setup lang="ts">
import { ref } from "vue";
import type { FormInstanceFunctions } from "tdesign-vue-next";

const formRef = ref<FormInstanceFunctions>();

// 校验整个表单
const validateForm = async () => {
  const result = await formRef.value?.validate();
  if (result === true) {
    console.log("校验通过");
  } else {
    console.log("校验失败", result);
  }
};

// 校验指定字段
const validateField = async (name: string) => {
  const result = await formRef.value?.validateOnly([name]);
  return result;
};

// 重置表单
const resetForm = () => {
  formRef.value?.reset();
};

// 清除校验状态
const clearValidate = (fields?: string[]) => {
  formRef.value?.clearValidate(fields);
};

// 设置字段值
const setFieldsValue = (values: Record<string, any>) => {
  formRef.value?.setFieldsValue(values);
};

// 获取字段值
const getFieldValue = (name: string) => {
  return formRef.value?.getFieldValue(name);
};
</script>
```

### 5. 条件渲染表单项

```vue
<template>
  <Form :model="formData" :rules="rules">
    <FormItem label="用户类型" name="userType">
      <RadioGroup v-model="formData.userType">
        <Radio value="personal">个人</Radio>
        <Radio value="enterprise">企业</Radio>
      </RadioGroup>
    </FormItem>

    <FormItem
      v-if="formData.userType === 'enterprise'"
      label="公司名称"
      name="companyName"
      :rules="[{ required: true, message: '公司名称必填' }]"
    >
      <Input v-model="formData.companyName" />
    </FormItem>
  </Form>
</template>

<script setup lang="ts">
import { reactive } from "vue";

const formData = reactive({
  userType: "personal",
  companyName: "",
});
</script>
```

### 6. 分步表单

```vue
<template>
  <Steps :current="currentStep">
    <StepItem title="基本信息" />
    <StepItem title="详细信息" />
    <StepItem title="完成" />
  </Steps>

  <Form ref="formRef" :model="formData" :rules="currentRules">
    <template v-if="currentStep === 0">
      <FormItem label="姓名" name="name">
        <Input v-model="formData.name" />
      </FormItem>
    </template>

    <template v-if="currentStep === 1">
      <FormItem label="邮箱" name="email">
        <Input v-model="formData.email" />
      </FormItem>
    </template>

    <Space>
      <Button v-if="currentStep > 0" @click="prevStep">上一步</Button>
      <Button v-if="currentStep < 2" theme="primary" @click="nextStep">
        下一步
      </Button>
      <Button v-if="currentStep === 2" theme="primary" @click="submit">
        提交
      </Button>
    </Space>
  </Form>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from "vue";
import type { FormInstanceFunctions, FormRules } from "tdesign-vue-next";

const formRef = ref<FormInstanceFunctions>();
const currentStep = ref(0);

const formData = reactive({
  name: "",
  email: "",
});

const stepRules: FormRules<typeof formData>[] = [
  { name: [{ required: true, message: "姓名必填" }] },
  { email: [{ required: true, message: "邮箱必填" }, { email: true }] },
  {},
];

const currentRules = computed(() => stepRules[currentStep.value]);

const nextStep = async () => {
  const result = await formRef.value?.validate();
  if (result === true) {
    currentStep.value++;
  }
};

const prevStep = () => {
  currentStep.value--;
};

const submit = () => {
  console.log("提交", formData);
};
</script>
```

## 常见问题与建议

### Q: 动态表单项校验不生效？

确保每个动态项有唯一的 `name`，使用数组形式 `['list', index, 'field']`。

### Q: 表单重置后校验状态未清除？

调用 `reset()` 方法会同时重置值和校验状态。如果只想清除校验状态，使用 `clearValidate()`。

### Q: 异步校验导致提交时机不对？

使用 `await formRef.value?.validate()` 等待所有校验（包括异步校验）完成。

### Q: 如何获取表单完整数据？

```ts
const allValues = formRef.value?.getAllFieldsValue();
```

### Q: 如何监听字段值变化？

```vue
<Form @valuesChange="onValuesChange">
  <!-- ... -->
</Form>

<script setup>
const onValuesChange = (changedValues: Record<string, any>, allValues: Record<string, any>) => {
  console.log('变化的字段', changedValues);
  console.log('所有字段', allValues);
};
</script>
```

## 最小示例

```vue
<template>
  <Form ref="formRef" :model="formData" :rules="rules" @submit="onSubmit">
    <FormItem label="用户名" name="username">
      <Input v-model="formData.username" />
    </FormItem>
    <FormItem label="邮箱" name="email">
      <Input v-model="formData.email" />
    </FormItem>
    <FormItem>
      <Space>
        <Button theme="primary" type="submit">提交</Button>
        <Button theme="default" @click="formRef?.reset()">重置</Button>
      </Space>
    </FormItem>
  </Form>
</template>

<script setup lang="ts">
import { reactive, ref } from "vue";
import { Form, FormItem, Input, Button, Space } from "tdesign-vue-next";
import type {
  FormInstanceFunctions,
  FormRules,
  SubmitContext,
} from "tdesign-vue-next";

const formRef = ref<FormInstanceFunctions>();

const formData = reactive({
  username: "",
  email: "",
});

const rules: FormRules<typeof formData> = {
  username: [{ required: true, message: "用户名必填" }],
  email: [
    { required: true, message: "邮箱必填" },
    { email: true, message: "请输入有效的邮箱地址" },
  ],
};

const onSubmit = ({ validateResult, firstError }: SubmitContext) => {
  if (validateResult === true) {
    console.log("提交成功", formData);
  } else {
    console.log("校验失败", firstError);
  }
};
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"是否使用 Form"或"基本校验规则"，回到主 Skill 的表单决策链
- 若涉及 Form 配合其他组件（如 Dialog 中的表单），查阅对应组件的 Reference

## 参考文档

- Form 组件文档：https://tdesign.tencent.com/vue-next/components/form
- FormItem 组件文档：https://tdesign.tencent.com/vue-next/components/form#formitem
