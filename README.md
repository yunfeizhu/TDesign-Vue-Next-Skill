# TDesign Vue Next Skill

> **Inspired by [antd-skill](https://github.com/ant-design/antd-skill)** - A similar skill project for Ant Design.

AI-powered guidance for [TDesign Vue Next](https://github.com/Tencent/tdesign-vue-next) component library development.

> ⚠️ **Note**: This skill is specifically designed for **TDesign Vue Next** (Vue 3). For other TDesign versions (React, Miniprogram, Mobile), please refer to their respective documentation.

## Overview

This skill provides comprehensive guidance for AI agents and developers working with TDesign Vue Next (Vue 3), covering:

- Component selection and usage patterns
- Complex scenarios (forms, tables, uploads, etc.)
- Theme customization and dark mode
- Best practices and anti-patterns

## Installation

```bash
npx skills add yunfeizhu/tdesign-vue-next-skill
```

Or manually copy the `skills/tdesign-vue-next/` directory to your project's agent skills folder.

## Structure

```
tdesign-vue-next-skill/
├── AGENTS.md                    # Development guide
├── README.md                    # This file
├── README.zh.md                 # Chinese documentation
└── skills/
    └── tdesign-vue-next/
        ├── SKILL.md             # Main skill file (SPO format)
        └── references/          # Advanced topic references
            ├── form-advanced.md
            ├── table-advanced.md
            ├── select-advanced.md
            ├── upload-advanced.md
            ├── tree-advanced.md
            ├── cascader-advanced.md
            ├── dialog-drawer-advanced.md
            ├── theming-advanced.md
            ├── dark-mode.md
            └── chat-advanced.md
```

## Usage

### For AI Agents

1. Read `SKILL.md` to understand the scope, process, and output format
2. When encountering complex scenarios, refer to the appropriate reference document
3. Follow the output specifications for consistent responses

### For Developers

1. Use the skill with your AI coding assistant
2. Reference the documents for best practices
3. Copy code examples directly into your project

## Coverage

### Components

- Basic: Button, Icon, Link, Typography
- Layout: Grid, Layout, Space, Divider
- Navigation: Menu, Tabs, Breadcrumb, Steps, Pagination
- Input: Form, Input, Select, DatePicker, Upload, etc.
- Display: Table, List, Tree, Card, etc.
- Feedback: Message, Notification, Dialog, Drawer, etc.
- Advanced: Chat (AI conversation)

### Topics

- Theme customization with CSS variables
- Dark mode implementation
- Form validation and dynamic fields
- Table with virtual scroll and server-side operations
- File upload with custom requests
- Tree and cascader with async loading

## Compatibility

- **TDesign Vue Next**: 1.x
- **Vue**: 3.3+
- **TypeScript**: 5.x recommended

## Resources

- [TDesign Official Documentation](https://tdesign.tencent.com/vue-next/overview)
- [TDesign GitHub Repository](https://github.com/Tencent/tdesign-vue-next)
- [TDesign Design Guidelines](https://tdesign.tencent.com/design/values)

## License

MIT License

## Contributing

Contributions are welcome! Please read [AGENTS.md](./AGENTS.md) for guidelines.
