# Chat 组件（AI 对话）复杂场景参考

## 适用场景边界

- AI 对话界面构建
- 流式响应展示
- 自定义消息渲染
- 消息列表管理
- 多模态内容（图片、文件、代码）
- 打字机效果
- 消息操作（复制、重新生成、点赞）

## 推荐模式

### 1. 使用受控消息列表

通过 `v-model:messages` 完全控制消息列表：

```vue
<Chat v-model:messages="messages" @send="onSend" />
```

### 2. 流式更新使用追加模式

对于流式响应，推荐追加更新而非替换：

```ts
const appendContent = (content: string) => {
  const lastMessage = messages.value[messages.value.length - 1];
  lastMessage.content += content;
};
```

## 必须避免的反模式

1. **频繁替换整个消息数组**：流式更新时会导致性能问题
2. **不处理 loading 状态**：用户无法知道 AI 正在响应
3. **忽略错误处理**：网络错误时需要给用户反馈
4. **不设置消息 ID**：会导致列表渲染异常

## 常见实践

### 1. 基础 AI 对话

```vue
<template>
  <div class="chat-container">
    <Chat v-model:messages="messages" :loading="loading" @send="handleSend">
      <template #empty>
        <div class="empty-state">
          <RobotIcon size="48" />
          <p>你好！我是 AI 助手，有什么可以帮助你的？</p>
        </div>
      </template>
    </Chat>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Chat } from "tdesign-vue-next";
import { RobotIcon } from "tdesign-icons-vue-next";
import type { ChatMessage, ChatSendContext } from "tdesign-vue-next";

const messages = ref<ChatMessage[]>([]);
const loading = ref(false);

const handleSend = async (context: ChatSendContext) => {
  const { content } = context;

  // 添加用户消息
  const userMessage: ChatMessage = {
    id: Date.now().toString(),
    role: "user",
    content,
    timestamp: Date.now(),
  };
  messages.value.push(userMessage);

  // 添加 AI 响应占位
  const aiMessage: ChatMessage = {
    id: (Date.now() + 1).toString(),
    role: "assistant",
    content: "",
    timestamp: Date.now(),
  };
  messages.value.push(aiMessage);

  loading.value = true;

  try {
    // 调用 AI API
    const response = await fetch("/api/chat", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ message: content }),
    });

    const data = await response.json();
    aiMessage.content = data.reply;
  } catch (error) {
    aiMessage.content = "抱歉，发生了错误，请稍后重试。";
    aiMessage.status = "error";
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.chat-container {
  height: 600px;
  border: 1px solid var(--td-border-level-2-color);
  border-radius: var(--td-radius-default);
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: var(--td-text-color-secondary);
}

.empty-state p {
  margin-top: 16px;
}
</style>
```

### 2. 流式响应（SSE）

```vue
<template>
  <Chat
    v-model:messages="messages"
    :loading="loading"
    @send="handleStreamSend"
  />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Chat } from "tdesign-vue-next";
import type { ChatMessage, ChatSendContext } from "tdesign-vue-next";

const messages = ref<ChatMessage[]>([]);
const loading = ref(false);
let abortController: AbortController | null = null;

const handleStreamSend = async (context: ChatSendContext) => {
  const { content } = context;

  // 添加用户消息
  messages.value.push({
    id: Date.now().toString(),
    role: "user",
    content,
    timestamp: Date.now(),
  });

  // 添加 AI 响应占位
  const aiMessageId = (Date.now() + 1).toString();
  messages.value.push({
    id: aiMessageId,
    role: "assistant",
    content: "",
    timestamp: Date.now(),
  });

  loading.value = true;
  abortController = new AbortController();

  try {
    const response = await fetch("/api/chat/stream", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ message: content }),
      signal: abortController.signal,
    });

    const reader = response.body?.getReader();
    const decoder = new TextDecoder();

    if (!reader) throw new Error("No reader available");

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      const chunk = decoder.decode(value);
      const lines = chunk.split("\n").filter(Boolean);

      for (const line of lines) {
        if (line.startsWith("data: ")) {
          const data = line.slice(6);
          if (data === "[DONE]") continue;

          try {
            const parsed = JSON.parse(data);
            const aiMessage = messages.value.find((m) => m.id === aiMessageId);
            if (aiMessage && parsed.content) {
              aiMessage.content += parsed.content;
            }
          } catch (e) {
            // 忽略解析错误
          }
        }
      }
    }
  } catch (error) {
    if ((error as Error).name === "AbortError") {
      console.log("请求已取消");
    } else {
      const aiMessage = messages.value.find((m) => m.id === aiMessageId);
      if (aiMessage) {
        aiMessage.content = "抱歉，发生了错误。";
        aiMessage.status = "error";
      }
    }
  } finally {
    loading.value = false;
    abortController = null;
  }
};

// 取消正在进行的请求
const cancelRequest = () => {
  abortController?.abort();
};
</script>
```

### 3. 自定义消息渲染

```vue
<template>
  <Chat v-model:messages="messages" @send="handleSend">
    <!-- 自定义消息内容 -->
    <template #message="{ message }">
      <div :class="['custom-message', `role-${message.role}`]">
        <Avatar v-if="message.role === 'assistant'" :image="botAvatar" />
        <Avatar v-else :image="userAvatar" />

        <div class="message-content">
          <!-- Markdown 渲染 -->
          <div
            v-if="message.role === 'assistant'"
            v-html="renderMarkdown(message.content)"
          />
          <div v-else>{{ message.content }}</div>

          <!-- 消息操作 -->
          <div class="message-actions">
            <Button
              variant="text"
              size="small"
              @click="copyMessage(message.content)"
            >
              <FileCopyIcon />
            </Button>
            <Button
              v-if="message.role === 'assistant'"
              variant="text"
              size="small"
              @click="regenerate(message)"
            >
              <RefreshIcon />
            </Button>
          </div>
        </div>
      </div>
    </template>
  </Chat>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Chat, Avatar, Button, MessagePlugin } from "tdesign-vue-next";
import { FileCopyIcon, RefreshIcon } from "tdesign-icons-vue-next";
import { marked } from "marked";
import type { ChatMessage } from "tdesign-vue-next";

const messages = ref<ChatMessage[]>([]);
const botAvatar = "/bot-avatar.png";
const userAvatar = "/user-avatar.png";

const renderMarkdown = (content: string) => {
  return marked(content);
};

const copyMessage = async (content: string) => {
  await navigator.clipboard.writeText(content);
  MessagePlugin.success("已复制到剪贴板");
};

const regenerate = (message: ChatMessage) => {
  // 找到对应的用户消息并重新发送
  const index = messages.value.findIndex((m) => m.id === message.id);
  if (index > 0) {
    const userMessage = messages.value[index - 1];
    // 移除当前 AI 回复
    messages.value.splice(index, 1);
    // 重新发送
    handleSend({ content: userMessage.content });
  }
};

const handleSend = async (context: { content: string }) => {
  // 发送逻辑...
};
</script>

<style scoped>
.custom-message {
  display: flex;
  gap: 12px;
  margin: 16px 0;
}

.custom-message.role-user {
  flex-direction: row-reverse;
}

.message-content {
  max-width: 70%;
  padding: 12px 16px;
  border-radius: var(--td-radius-default);
  background-color: var(--td-bg-color-container);
}

.role-user .message-content {
  background-color: var(--td-brand-color-light);
}

.message-actions {
  display: flex;
  gap: 4px;
  margin-top: 8px;
  opacity: 0;
  transition: opacity 0.2s;
}

.message-content:hover .message-actions {
  opacity: 1;
}
</style>
```

### 4. 多模态消息（图片、文件）

```vue
<template>
  <Chat v-model:messages="messages" @send="handleSend">
    <template #input>
      <div class="custom-input">
        <div v-if="attachments.length" class="attachments">
          <div
            v-for="(file, index) in attachments"
            :key="index"
            class="attachment-item"
          >
            <img v-if="isImage(file)" :src="getPreviewUrl(file)" />
            <span v-else>{{ file.name }}</span>
            <Button
              variant="text"
              size="small"
              @click="removeAttachment(index)"
            >
              <CloseIcon />
            </Button>
          </div>
        </div>

        <div class="input-row">
          <Upload
            :request-method="() => Promise.resolve({ status: 'success' })"
            :show-upload-progress="false"
            theme="custom"
            @success="onFileSelect"
          >
            <Button variant="outline">
              <AttachIcon />
            </Button>
          </Upload>

          <Textarea
            v-model="inputValue"
            :autosize="{ minRows: 1, maxRows: 5 }"
            placeholder="输入消息..."
            @keydown.enter.exact.prevent="sendMessage"
          />

          <Button theme="primary" :disabled="!canSend" @click="sendMessage">
            <SendIcon />
          </Button>
        </div>
      </div>
    </template>

    <template #message="{ message }">
      <div class="message-with-attachments">
        <div v-if="message.attachments?.length" class="message-attachments">
          <template v-for="(att, i) in message.attachments" :key="i">
            <Image
              v-if="att.type === 'image'"
              :src="att.url"
              :style="{ maxWidth: '200px' }"
            />
            <div v-else class="file-attachment">
              <FileIcon />
              <span>{{ att.name }}</span>
            </div>
          </template>
        </div>
        <p>{{ message.content }}</p>
      </div>
    </template>
  </Chat>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import { Chat, Button, Textarea, Upload, Image } from "tdesign-vue-next";
import {
  AttachIcon,
  SendIcon,
  CloseIcon,
  FileIcon,
} from "tdesign-icons-vue-next";
import type { ChatMessage, UploadFile } from "tdesign-vue-next";

interface Attachment {
  type: "image" | "file";
  name: string;
  url: string;
  file?: File;
}

interface ExtendedMessage extends ChatMessage {
  attachments?: Attachment[];
}

const messages = ref<ExtendedMessage[]>([]);
const inputValue = ref("");
const attachments = ref<File[]>([]);

const canSend = computed(() => {
  return inputValue.value.trim() || attachments.value.length > 0;
});

const isImage = (file: File) => file.type.startsWith("image/");

const getPreviewUrl = (file: File) => URL.createObjectURL(file);

const onFileSelect = (context: { file: UploadFile }) => {
  if (context.file.raw) {
    attachments.value.push(context.file.raw);
  }
};

const removeAttachment = (index: number) => {
  attachments.value.splice(index, 1);
};

const sendMessage = async () => {
  if (!canSend.value) return;

  const messageAttachments: Attachment[] = await Promise.all(
    attachments.value.map(async (file) => {
      // 上传文件并获取 URL
      const url = await uploadFile(file);
      return {
        type: isImage(file) ? "image" : "file",
        name: file.name,
        url,
      } as Attachment;
    }),
  );

  const userMessage: ExtendedMessage = {
    id: Date.now().toString(),
    role: "user",
    content: inputValue.value,
    attachments: messageAttachments,
    timestamp: Date.now(),
  };

  messages.value.push(userMessage);
  inputValue.value = "";
  attachments.value = [];

  // 发送到 AI...
};

const uploadFile = async (file: File): Promise<string> => {
  // 实际上传逻辑
  return URL.createObjectURL(file);
};

const handleSend = (context: { content: string }) => {
  inputValue.value = context.content;
  sendMessage();
};
</script>
```

### 5. 代码块高亮

```vue
<template>
  <Chat v-model:messages="messages" @send="handleSend">
    <template #message="{ message }">
      <div class="message-content" v-html="renderContent(message.content)" />
    </template>
  </Chat>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Chat } from "tdesign-vue-next";
import { marked } from "marked";
import hljs from "highlight.js";
import "highlight.js/styles/github-dark.css";
import type { ChatMessage } from "tdesign-vue-next";

// 配置 marked 使用 highlight.js
marked.setOptions({
  highlight: (code, lang) => {
    if (lang && hljs.getLanguage(lang)) {
      return hljs.highlight(code, { language: lang }).value;
    }
    return hljs.highlightAuto(code).value;
  },
});

const messages = ref<ChatMessage[]>([]);

const renderContent = (content: string) => {
  return marked(content);
};

const handleSend = (context: { content: string }) => {
  // 发送逻辑...
};
</script>

<style>
/* 代码块样式 */
.message-content pre {
  position: relative;
  background-color: #1e1e1e;
  border-radius: var(--td-radius-default);
  overflow: hidden;
}

.message-content pre code {
  display: block;
  padding: 16px;
  overflow-x: auto;
}

/* 添加复制按钮 */
.message-content pre::before {
  content: attr(data-language);
  position: absolute;
  top: 8px;
  right: 8px;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.5);
}
</style>
```

### 6. 打字机效果

```vue
<script setup lang="ts">
import { ref, nextTick } from "vue";
import type { ChatMessage } from "tdesign-vue-next";

const messages = ref<ChatMessage[]>([]);
const typingSpeed = 30; // 每个字符的延迟（毫秒）

const typeMessage = async (messageId: string, fullContent: string) => {
  const message = messages.value.find((m) => m.id === messageId);
  if (!message) return;

  message.content = "";

  for (let i = 0; i < fullContent.length; i++) {
    await new Promise((resolve) => setTimeout(resolve, typingSpeed));
    message.content += fullContent[i];
    await nextTick();
  }
};

const handleSend = async (context: { content: string }) => {
  // 添加用户消息
  messages.value.push({
    id: Date.now().toString(),
    role: "user",
    content: context.content,
    timestamp: Date.now(),
  });

  // 获取 AI 回复
  const response = await fetchAIResponse(context.content);

  // 添加 AI 消息占位
  const aiMessageId = (Date.now() + 1).toString();
  messages.value.push({
    id: aiMessageId,
    role: "assistant",
    content: "",
    timestamp: Date.now(),
  });

  // 打字机效果展示
  await typeMessage(aiMessageId, response);
};

const fetchAIResponse = async (message: string): Promise<string> => {
  // 模拟 API 调用
  await new Promise((resolve) => setTimeout(resolve, 500));
  return "这是 AI 的回复内容，会以打字机效果逐字显示。";
};
</script>
```

### 7. 消息状态管理

```vue
<template>
  <Chat v-model:messages="messages" @send="handleSend">
    <template #message="{ message }">
      <div class="message-wrapper">
        <div class="message-content">{{ message.content }}</div>

        <!-- 消息状态指示 -->
        <div class="message-status">
          <template v-if="message.status === 'sending'">
            <LoadingIcon class="spinning" />
            <span>发送中</span>
          </template>
          <template v-else-if="message.status === 'error'">
            <ErrorCircleIcon style="color: var(--td-error-color)" />
            <Button variant="text" size="small" @click="retry(message)">
              重试
            </Button>
          </template>
          <template v-else-if="message.status === 'success'">
            <CheckCircleIcon style="color: var(--td-success-color)" />
          </template>
        </div>
      </div>
    </template>
  </Chat>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Chat, Button } from "tdesign-vue-next";
import {
  LoadingIcon,
  ErrorCircleIcon,
  CheckCircleIcon,
} from "tdesign-icons-vue-next";
import type { ChatMessage } from "tdesign-vue-next";

interface MessageWithStatus extends ChatMessage {
  status?: "sending" | "success" | "error";
}

const messages = ref<MessageWithStatus[]>([]);

const handleSend = async (context: { content: string }) => {
  const message: MessageWithStatus = {
    id: Date.now().toString(),
    role: "user",
    content: context.content,
    status: "sending",
    timestamp: Date.now(),
  };

  messages.value.push(message);

  try {
    await sendToServer(context.content);
    message.status = "success";
  } catch (error) {
    message.status = "error";
  }
};

const retry = async (message: MessageWithStatus) => {
  message.status = "sending";
  try {
    await sendToServer(message.content);
    message.status = "success";
  } catch (error) {
    message.status = "error";
  }
};

const sendToServer = async (content: string) => {
  // 发送到服务器的逻辑
};
</script>

<style scoped>
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.spinning {
  animation: spin 1s linear infinite;
}

.message-status {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 12px;
  color: var(--td-text-color-secondary);
  margin-top: 4px;
}
</style>
```

## 常见问题与建议

### Q: 流式响应时滚动条跳动？

在更新消息内容后，使用 `nextTick` 确保 DOM 更新后再滚动：

```ts
await nextTick();
chatRef.value?.scrollToBottom();
```

### Q: 如何处理超长对话的性能问题？

1. 使用虚拟滚动
2. 限制保留的消息数量
3. 懒加载历史消息

### Q: 如何实现消息搜索？

维护消息索引，支持全文搜索或使用搜索引擎。

### Q: 如何保存对话历史？

使用 localStorage、IndexedDB 或服务端存储，在组件挂载时恢复。

## 最小示例

```vue
<template>
  <Chat v-model:messages="messages" :loading="loading" @send="handleSend" />
</template>

<script setup lang="ts">
import { ref } from "vue";
import { Chat } from "tdesign-vue-next";
import type { ChatMessage, ChatSendContext } from "tdesign-vue-next";

const messages = ref<ChatMessage[]>([]);
const loading = ref(false);

const handleSend = async (context: ChatSendContext) => {
  messages.value.push({
    id: Date.now().toString(),
    role: "user",
    content: context.content,
    timestamp: Date.now(),
  });

  loading.value = true;
  // 获取 AI 回复...
  loading.value = false;
};
</script>
```

## 与主 Skill 的回跳说明

- 若问题只涉及"简单消息列表展示"，可以使用 List 组件
- 若涉及消息中的表单元素，查阅 `form-advanced.md`

## 参考文档

- Chat 组件文档：https://tdesign.tencent.com/vue-next/components/chat
