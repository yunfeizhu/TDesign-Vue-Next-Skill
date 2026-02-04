# AGENTS.md - TDesign Skill Development Guide

This document provides guidance for AI Agents and developers on using TDesign Vue Next Skill.

## Skill Structure

```
tdesign-skill/
├── AGENTS.md                           # This document
├── LICENSE                             # MIT License
├── README.md                           # English documentation
├── README.zh.md                        # Chinese documentation
└── skills/
    └── tdesign-vue-next/
        ├── LICENSE                     # Skill license
        ├── SKILL.md                    # Main skill file (SPO format)
        └── references/                 # Advanced scenario references
            ├── tdesign-v1.md           # Version reference
            ├── form-advanced.md        # Form advanced usage
            ├── table-advanced.md       # Table advanced usage
            ├── select-advanced.md      # Select advanced usage
            ├── upload-advanced.md      # Upload advanced usage
            ├── tree-advanced.md        # Tree advanced usage
            ├── cascader-advanced.md    # Cascader advanced usage
            ├── dialog-drawer-advanced.md # Dialog & Drawer advanced usage
            ├── theming-advanced.md     # Theme customization
            ├── dark-mode.md            # Dark mode
            └── chat-advanced.md        # AI Chat component
```

## How to Use This Skill

### For AI Agents

1. **Read SKILL.md first**: Understand the Scope, Process, and Output specifications

2. **Identify complex scenarios**: Consult the corresponding Reference documents when encountering:
   - Dynamic forms, cross-field validation → `form-advanced.md`
   - Server-side pagination, virtual scrolling → `table-advanced.md`
   - Remote search, large dataset selection → `select-advanced.md`
   - Controlled upload, custom requests → `upload-advanced.md`
   - Async tree node loading → `tree-advanced.md`
   - Async cascader loading → `cascader-advanced.md`
   - Nested dialogs, imperative calls → `dialog-drawer-advanced.md`
   - Deep theme customization → `theming-advanced.md`
   - Dark mode toggle → `dark-mode.md`
   - AI chat interface → `chat-advanced.md`

3. **Follow output specifications**:
   - Use TypeScript + `<script setup>` syntax
   - Provide directly runnable code
   - Include necessary import statements
   - Add performance and accessibility tips

### For Developers

1. **Install Skill** (if skills CLI is supported):

   ```bash
   npx skills add tdesign-skill
   ```

2. **Manual usage**:
   - Add this Skill to your AI tool's context
   - Or read the documentation directly for best practices

## Skill Design Principles

### SPO Format

Each Skill follows the **S**cope-**P**rocess-**O**utput format:

- **Scope**: Defines applicable scenarios and boundaries
- **Process**: Provides decision trees and processing steps
- **Output**: Specifies output format and checklists

### Reference Document Format

Each Reference document contains:

1. **Applicable scenario boundaries**: When to consult this document
2. **Recommended patterns**: Best practices (1-2 types)
3. **Anti-patterns to avoid**: Common mistakes
4. **Common practices**: Detailed code examples
5. **FAQ and suggestions**: Frequently asked questions
6. **Minimal examples**: Runnable minimal code
7. **Return to main Skill**: When to return to the main Skill
8. **Reference documentation**: Official documentation links

## Contributing Guide

### Adding New References

1. Create a Markdown file in the `references/` directory
2. Follow the Reference document format above
3. Add an index in SKILL.md's Complex triggers and Reference index
4. Ensure example code can be run directly

### Updating Existing Documents

1. Ensure compatibility with the latest TDesign Vue Next version
2. Add new APIs and best practices
3. Remove deprecated content

### Code Example Requirements

- Use TypeScript
- Use `<script setup>` syntax
- Include complete import statements
- Add necessary type annotations
- Ensure code can be copied and run directly

## Version Compatibility

- **TDesign Vue Next**: 1.x
- **Vue**: 3.3+
- **TypeScript**: 5.x recommended

## License

MIT License
