# TDesign Vue Next Skill

简体中文 | [English](./README.md)

> **灵感来源于 [antd-skill](https://github.com/ant-design/antd-skill)** - Ant Design 的同类 Skill 项目。

为 [TDesign Vue Next](https://github.com/Tencent/tdesign-vue-next) 组件库开发提供 AI 驱动的指导。

> ⚠️ **注意**：本 Skill 专门为 **TDesign Vue Next**（Vue 3）设计。如需其他 TDesign 版本（React、小程序、移动端），请参考对应官方文档。

## 概述

本 Skill 为 AI 代理和开发者提供使用 TDesign Vue Next（Vue 3）的全面指导，涵盖：

- 组件选择和使用模式
- 复杂场景（表单、表格、上传等）
- 主题定制和暗黑模式
- 最佳实践和反模式

## 安装

```bash
npx skills add yunfeizhu/tdesign-vue-next-skill
```

或手动复制 `skills/tdesign-vue-next/` 目录到你项目的 agent skills 文件夹。

## 目录结构

```
tdesign-vue-next-skill/
├── AGENTS.md                    # 开发指南
├── README.md                    # 英文文档
├── README.zh.md                 # 本文档
└── skills/
    └── tdesign-vue-next/
        ├── SKILL.md             # 主 Skill 文件（SPO 格式）
        └── references/          # 高级主题参考文档
            ├── tdesign-v1.md    # 版本参考
            ├── form-advanced.md # 表单高级用法
            ├── table-advanced.md # 表格高级用法
            ├── select-advanced.md # 选择器高级用法
            ├── upload-advanced.md # 上传高级用法
            ├── tree-advanced.md # 树组件高级用法
            ├── cascader-advanced.md # 级联选择高级用法
            ├── dialog-drawer-advanced.md # 弹窗抽屉高级用法
            ├── theming-advanced.md # 主题定制
            ├── dark-mode.md     # 暗黑模式
            └── chat-advanced.md # AI 对话组件
```

## 使用方法

### 对于 AI 代理

1. 阅读 `SKILL.md` 了解范围、流程和输出格式
2. 遇到复杂场景时，参考对应的 reference 文档
3. 遵循输出规范以保持一致的响应

### 对于开发者

1. 将此 Skill 与你的 AI 编程助手配合使用
2. 参考文档获取最佳实践
3. 直接复制代码示例到你的项目中

## 覆盖范围

### 组件

- 基础：Button、Icon、Link、Typography
- 布局：Grid、Layout、Space、Divider
- 导航：Menu、Tabs、Breadcrumb、Steps、Pagination
- 输入：Form、Input、Select、DatePicker、Upload 等
- 展示：Table、List、Tree、Card 等
- 反馈：Message、Notification、Dialog、Drawer 等
- 高级：Chat（AI 对话）

### 主题

- 使用 CSS 变量的主题定制
- 暗黑模式实现
- 表单校验和动态字段
- 虚拟滚动和服务端操作的表格
- 自定义请求的文件上传
- 异步加载的树和级联选择

## 兼容性

- **TDesign Vue Next**：1.x
- **Vue**：3.3+
- **TypeScript**：推荐 5.x

## 资源

- [TDesign 官方文档](https://tdesign.tencent.com/vue-next/overview)
- [TDesign GitHub 仓库](https://github.com/Tencent/tdesign-vue-next)
- [TDesign 设计规范](https://tdesign.tencent.com/design/values)

## 许可证

MIT License

## 贡献

欢迎贡献！请阅读 [AGENTS.md](./AGENTS.md) 了解指南。
