# 上传（Upload）复杂场景参考

## 适用场景边界

- 受控文件列表（完全控制文件状态）
- 自定义上传请求（替换默认 XHR）
- 断点续传、分片上传
- 图片预览与裁剪
- 拖拽上传、粘贴上传
- 文件夹上传
- 上传前拦截与校验
- 进度展示与取消上传

## 推荐模式

### 1. 受控模式管理文件列表

使用 `v-model:files` 完全控制文件列表状态：

```vue
<Upload
  v-model:files="fileList"
  :request-method="customRequest"
  @success="onSuccess"
  @fail="onFail"
/>
```

### 2. 自定义请求方法

通过 `request-method` 完全接管上传逻辑：

```ts
const customRequest: UploadProps["requestMethod"] = (file) => {
  return new Promise((resolve, reject) => {
    const formData = new FormData();
    formData.append("file", file.raw!);

    axios
      .post("/api/upload", formData, {
        onUploadProgress: (e) => {
          file.percent = Math.round((e.loaded / e.total!) * 100);
        },
      })
      .then((res) => resolve({ status: "success", response: res.data }))
      .catch((err) => reject({ status: "fail", error: err }));
  });
};
```

## 必须避免的反模式

1. **不设置受控模式就期望获取文件列表**：必须使用 `v-model:files`
2. **`request-method` 不返回 Promise**：会导致上传状态异常
3. **忘记处理上传失败状态**：用户无法得知失败原因
4. **大文件不使用分片上传**：可能导致上传超时或内存问题

## 常见实践

### 1. 受控文件列表

```vue
<template>
  <Upload
    v-model:files="fileList"
    :action="uploadUrl"
    :max="5"
    @success="onSuccess"
    @fail="onFail"
    @remove="onRemove"
  >
    <Button>点击上传</Button>
  </Upload>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload, Button } from "tdesign-vue-next";
import type {
  UploadFile,
  SuccessContext,
  UploadRemoveContext,
} from "tdesign-vue-next";

const uploadUrl = "/api/upload";
const fileList = ref<UploadFile[]>([]);

const onSuccess = (context: SuccessContext) => {
  console.log("上传成功", context.file);
  // 可以在这里更新文件的 url
  const file = context.file;
  if (context.response?.url) {
    file.url = context.response.url;
  }
};

const onFail = (context: { file: UploadFile; error: Error }) => {
  console.error("上传失败", context.error);
};

const onRemove = (context: UploadRemoveContext) => {
  console.log("移除文件", context.file);
};
</script>
```

### 2. 自定义上传请求

```vue
<template>
  <Upload v-model:files="fileList" :request-method="customRequest">
    <Button>自定义上传</Button>
  </Upload>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload, Button } from "tdesign-vue-next";
import type {
  UploadFile,
  RequestMethodResponse,
  UploadProps,
} from "tdesign-vue-next";
import axios from "axios";

const fileList = ref<UploadFile[]>([]);

const customRequest: UploadProps["requestMethod"] = async (file) => {
  const formData = new FormData();
  formData.append("file", file.raw!);
  formData.append("name", file.name);

  try {
    const response = await axios.post("/api/upload", formData, {
      headers: {
        "Content-Type": "multipart/form-data",
        Authorization: `Bearer ${getToken()}`,
      },
      onUploadProgress: (progressEvent) => {
        if (progressEvent.total) {
          file.percent = Math.round(
            (progressEvent.loaded / progressEvent.total) * 100,
          );
        }
      },
    });

    return {
      status: "success",
      response: {
        url: response.data.url,
        ...response.data,
      },
    } as RequestMethodResponse;
  } catch (error) {
    return {
      status: "fail",
      error: error instanceof Error ? error.message : "上传失败",
    } as RequestMethodResponse;
  }
};

const getToken = () => localStorage.getItem("token") || "";
</script>
```

### 3. 上传前校验

```vue
<template>
  <Upload
    v-model:files="fileList"
    :action="uploadUrl"
    :before-upload="beforeUpload"
    :size-limit="{ size: 10, unit: 'MB' }"
    accept="image/*,.pdf"
  >
    <Button>上传文件</Button>
    <template #tips>
      <div class="tips">支持 jpg、png、pdf 格式，单个文件不超过 10MB</div>
    </template>
  </Upload>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload, Button, MessagePlugin } from "tdesign-vue-next";
import type { UploadFile, UploadProps } from "tdesign-vue-next";

const uploadUrl = "/api/upload";
const fileList = ref<UploadFile[]>([]);

const beforeUpload: UploadProps["beforeUpload"] = (file) => {
  // 检查文件类型
  const allowedTypes = ["image/jpeg", "image/png", "application/pdf"];
  if (!allowedTypes.includes(file.type)) {
    MessagePlugin.error("不支持的文件类型");
    return false;
  }

  // 检查文件名
  if (file.name.length > 100) {
    MessagePlugin.error("文件名过长");
    return false;
  }

  // 返回修改后的文件（可选）
  // return new File([file.raw], 'new-name.jpg', { type: file.type });

  return true; // 允许上传
  // return false; // 阻止上传
};
</script>
```

### 4. 图片上传与预览

```vue
<template>
  <Upload
    v-model:files="fileList"
    :action="uploadUrl"
    theme="image"
    accept="image/*"
    :max="9"
    multiple
    :before-upload="validateImage"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload, MessagePlugin } from "tdesign-vue-next";
import type { UploadFile, UploadProps } from "tdesign-vue-next";

const uploadUrl = "/api/upload";
const fileList = ref<UploadFile[]>([]);

const validateImage: UploadProps["beforeUpload"] = (file) => {
  return new Promise((resolve) => {
    const img = new Image();
    img.src = URL.createObjectURL(file.raw!);

    img.onload = () => {
      URL.revokeObjectURL(img.src);

      // 检查图片尺寸
      if (img.width < 200 || img.height < 200) {
        MessagePlugin.error("图片尺寸不能小于 200x200");
        resolve(false);
        return;
      }

      // 检查宽高比
      const ratio = img.width / img.height;
      if (ratio < 0.5 || ratio > 2) {
        MessagePlugin.error("图片宽高比需在 1:2 到 2:1 之间");
        resolve(false);
        return;
      }

      resolve(true);
    };

    img.onerror = () => {
      MessagePlugin.error("无法读取图片");
      resolve(false);
    };
  });
};
</script>
```

### 5. 拖拽上传

```vue
<template>
  <Upload
    v-model:files="fileList"
    :action="uploadUrl"
    theme="custom"
    draggable
    multiple
  >
    <template #dragContent="{ dragActive }">
      <div :class="['drag-area', { active: dragActive }]">
        <UploadIcon size="48" />
        <p v-if="dragActive">释放鼠标上传</p>
        <p v-else>点击或拖拽文件到此区域上传</p>
      </div>
    </template>
  </Upload>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload } from "tdesign-vue-next";
import { UploadIcon } from "tdesign-icons-vue-next";
import type { UploadFile } from "tdesign-vue-next";

const uploadUrl = "/api/upload";
const fileList = ref<UploadFile[]>([]);
</script>

<style scoped>
.drag-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 200px;
  border: 2px dashed var(--td-border-level-2-color);
  border-radius: 8px;
  cursor: pointer;
  transition: border-color 0.2s;
}

.drag-area:hover,
.drag-area.active {
  border-color: var(--td-brand-color);
}

.drag-area p {
  margin-top: 8px;
  color: var(--td-text-color-secondary);
}
</style>
```

### 6. 分片上传

```vue
<template>
  <Upload
    v-model:files="fileList"
    :request-method="chunkUpload"
    :size-limit="{ size: 500, unit: 'MB' }"
  >
    <Button>上传大文件</Button>
  </Upload>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload, Button } from "tdesign-vue-next";
import type {
  UploadFile,
  RequestMethodResponse,
  UploadProps,
} from "tdesign-vue-next";
import axios from "axios";

const fileList = ref<UploadFile[]>([]);

const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB per chunk

const chunkUpload: UploadProps["requestMethod"] = async (file) => {
  const raw = file.raw!;
  const totalChunks = Math.ceil(raw.size / CHUNK_SIZE);
  const fileHash = await calculateHash(raw);

  try {
    // 检查已上传的分片
    const { uploadedChunks } = await axios
      .get(`/api/upload/check?hash=${fileHash}`)
      .then((res) => res.data);

    // 上传未完成的分片
    for (let i = 0; i < totalChunks; i++) {
      if (uploadedChunks.includes(i)) continue;

      const start = i * CHUNK_SIZE;
      const end = Math.min(start + CHUNK_SIZE, raw.size);
      const chunk = raw.slice(start, end);

      const formData = new FormData();
      formData.append("chunk", chunk);
      formData.append("hash", fileHash);
      formData.append("index", String(i));
      formData.append("total", String(totalChunks));

      await axios.post("/api/upload/chunk", formData);

      // 更新进度
      file.percent = Math.round(((i + 1) / totalChunks) * 100);
    }

    // 合并分片
    const { url } = await axios
      .post("/api/upload/merge", {
        hash: fileHash,
        name: file.name,
        total: totalChunks,
      })
      .then((res) => res.data);

    return { status: "success", response: { url } } as RequestMethodResponse;
  } catch (error) {
    return {
      status: "fail",
      error: error instanceof Error ? error.message : "上传失败",
    } as RequestMethodResponse;
  }
};

// 计算文件 hash（使用 spark-md5 或 Web Crypto API）
const calculateHash = async (file: File): Promise<string> => {
  // 简化示例，实际应使用 Web Worker 计算
  return `${file.name}_${file.size}_${file.lastModified}`;
};
</script>
```

### 7. 文件列表展示

```vue
<template>
  <Upload
    v-model:files="fileList"
    :action="uploadUrl"
    theme="file-flow"
    multiple
  >
    <Button>上传文件</Button>
  </Upload>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload, Button } from "tdesign-vue-next";
import type { UploadFile } from "tdesign-vue-next";

const uploadUrl = "/api/upload";

// 初始化已有文件
const fileList = ref<UploadFile[]>([
  {
    name: "已有文件.pdf",
    url: "https://example.com/file.pdf",
    status: "success",
    size: 1024000,
  },
]);
</script>
```

### 8. 手动触发上传

```vue
<template>
  <Space direction="vertical">
    <Upload
      ref="uploadRef"
      v-model:files="fileList"
      :action="uploadUrl"
      :auto-upload="false"
      multiple
    >
      <Button>选择文件</Button>
    </Upload>
    <Button theme="primary" @click="startUpload">开始上传</Button>
  </Space>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload, Button, Space } from "tdesign-vue-next";
import type { UploadFile, UploadInstanceFunctions } from "tdesign-vue-next";

const uploadUrl = "/api/upload";
const fileList = ref<UploadFile[]>([]);
const uploadRef = ref<UploadInstanceFunctions>();

const startUpload = () => {
  uploadRef.value?.uploadFiles();
};
</script>
```

## 常见问题与建议

### Q: 如何取消正在上传的文件？

使用 Upload 实例的 `cancelUpload` 方法，或通过 AbortController 在自定义请求中取消。

### Q: 上传后如何获取服务端返回的文件 URL？

在 `onSuccess` 回调中从 `context.response` 获取，并更新到文件对象：

```ts
const onSuccess = (context: SuccessContext) => {
  context.file.url = context.response?.url;
};
```

### Q: 如何限制同时上传的文件数？

使用 `max` 属性限制总文件数，使用 `uploadAllFilesInOneRequest` 决定是否批量上传。

### Q: 如何显示上传进度？

默认会显示进度条，自定义请求时需要更新 `file.percent` 属性。

## 最小示例

```vue
<template>
  <Upload v-model:files="fileList" :action="uploadUrl">
    <Button>点击上传</Button>
  </Upload>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Upload, Button } from "tdesign-vue-next";
import type { UploadFile } from "tdesign-vue-next";

const uploadUrl = "/api/upload";
const fileList = ref<UploadFile[]>([]);
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"基本文件上传"，回到主 Skill 的组件选型规则
- 若涉及 Form 中的上传校验，查阅 `form-advanced.md`

## 参考文档

- Upload 组件文档：https://tdesign.tencent.com/vue-next/components/upload
