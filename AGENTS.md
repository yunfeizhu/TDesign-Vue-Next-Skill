# AGENTS.md - TDesign Skill 开发指南

本文档为 AI Agent 和开发者提供使用 TDesign Vue Next Skill 的指导。

## Skill 结构

```
tdesign-skill/
├── AGENTS.md                           # 本文档
├── LICENSE                             # MIT 许可证
├── README.md                           # 英文说明
├── README.zh.md                        # 中文说明
└── skills/
    └── tdesign-vue-next/
        ├── LICENSE                     # Skill 许可证
        ├── SKILL.md                    # 主 Skill 文件（SPO 格式）
        └── references/                 # 复杂场景参考文档
            ├── tdesign-v1.md           # 版本参考
            ├── form-advanced.md        # 表单高级用法
            ├── table-advanced.md       # 表格高级用法
            ├── select-advanced.md      # 选择器高级用法
            ├── upload-advanced.md      # 上传高级用法
            ├── tree-advanced.md        # 树组件高级用法
            ├── cascader-advanced.md    # 级联选择高级用法
            ├── dialog-drawer-advanced.md # 弹窗抽屉高级用法
            ├── theming-advanced.md     # 主题定制
            ├── dark-mode.md            # 暗黑模式
            └── chat-advanced.md        # AI 对话组件
```

## 如何使用此 Skill

### 对于 AI Agent

1. **先阅读 SKILL.md**：了解 Scope（适用范围）、Process（处理流程）和 Output（输出规范）

2. **识别复杂场景**：当遇到以下情况时，查阅对应的 Reference 文档：
   - 动态表单、跨字段联动 → `form-advanced.md`
   - 服务端分页、虚拟滚动 → `table-advanced.md`
   - 远程搜索、大数据量选择 → `select-advanced.md`
   - 受控上传、自定义请求 → `upload-advanced.md`
   - 异步加载树节点 → `tree-advanced.md`
   - 异步级联加载 → `cascader-advanced.md`
   - 嵌套弹窗、命令式调用 → `dialog-drawer-advanced.md`
   - 深度主题定制 → `theming-advanced.md`
   - 暗黑模式切换 → `dark-mode.md`
   - AI 对话界面 → `chat-advanced.md`

3. **遵循输出规范**：
   - 使用 TypeScript + `<script setup>` 语法
   - 提供可直接运行的代码
   - 包含必要的 import 语句
   - 添加性能和可访问性提示

### 对于开发者

1. **安装 Skill**（如果支持 skills CLI）：

   ```bash
   npx skills add tdesign-skill
   ```

2. **手动使用**：
   - 将此 Skill 添加到你的 AI 工具的上下文中
   - 或者直接阅读文档获取最佳实践

## Skill 设计原则

### SPO 格式

每个 Skill 遵循 **S**cope-**P**rocess-**O**utput 格式：

- **Scope（范围）**：定义适用场景和边界
- **Process（流程）**：提供决策树和处理步骤
- **Output（输出）**：规范输出格式和检查清单

### Reference 文档格式

每个 Reference 文档包含：

1. **适用场景边界**：何时应该查阅此文档
2. **推荐模式**：最佳实践（1-2 种）
3. **必须避免的反模式**：常见错误
4. **常见实践**：详细的代码示例
5. **常见问题与建议**：FAQ
6. **最小示例**：可运行的最小代码
7. **与主 Skill 的回跳说明**：何时返回主 Skill
8. **参考文档**：官方文档链接

## 贡献指南

### 添加新的 Reference

1. 在 `references/` 目录下创建 Markdown 文件
2. 遵循上述 Reference 文档格式
3. 在 `SKILL.md` 的 Complex triggers 和 Reference index 中添加索引
4. 确保示例代码可以直接运行

### 更新现有文档

1. 确保与 TDesign Vue Next 最新版本兼容
2. 添加新的 API 和最佳实践
3. 移除已废弃的内容

### 代码示例要求

- 使用 TypeScript
- 使用 `<script setup>` 语法
- 包含完整的 import 语句
- 添加必要的类型注解
- 确保可以直接复制运行

## 版本兼容性

- **TDesign Vue Next**：1.x
- **Vue**：3.3+
- **TypeScript**：5.x 推荐

## 许可证

MIT License
