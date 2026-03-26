# 第七章·第一节 Skill 设计模式

## 概述

本节系统梳理 Superpowers 中 Skill 的设计模式——从 SKILL.md 结构规范到目录组织，从 Progressive Disclosure(渐进式披露) 到 Feedback Loop(反馈循环)，涵盖所有经过验证的模式和需要避免的反模式。

---

## SKILL.md 结构规范

每个 Skill 的入口文件是 `SKILL.md`，由 YAML Frontmatter 和 Markdown Body 两部分组成。

### 标准骨架

```markdown
---
name: skill-name-with-hyphens
description: Use when [specific triggering conditions and symptoms]
---

# Skill Name

## Overview
Core principle in 1-2 sentences.

## When to Use
Bullet list with SYMPTOMS and use cases. When NOT to use.

## Core Pattern
Before/after code comparison (for techniques/patterns).

## Quick Reference
Table or bullets for scanning common operations.

## Implementation
Inline code for simple patterns. Link to file for heavy reference.

## Common Mistakes
What goes wrong + fixes.

## Real-World Impact (optional)
Concrete results.
```

---

## YAML Frontmatter 规范

| 字段 | 要求 | 限制 |
|------|------|------|
| `name` | 必填 | 64 字符，仅字母/数字/连字符 |
| `description` | 必填 | 1024 字符，第三人称，"Use when..." 开头 |
| 总 frontmatter | — | 最大 1024 字符 |

完整规范见 [agentskills.io/specification](https://agentskills.io/specification)。

**description 黄金规则**：只描述**何时使用**（触发条件），**绝不**总结 skill 的工作流程。

```yaml
# ✅ GOOD
description: Use when executing implementation plans with independent tasks

# ❌ BAD — 包含工作流摘要，Claude 会走捷径
description: Use when executing plans - dispatches subagent per task with code review between tasks
```

---

## Skill 类型三分法

| 类型 | 英文名 | 特征 | 示例 |
|------|--------|------|------|
| 技术型 | Technique | 具体步骤可遵循 | condition-based-waiting, root-cause-tracing |
| 模式型 | Pattern | 思考问题的方式 | flatten-with-flags, test-invariants |
| 参考型 | Reference | API 文档、语法指南 | office docs, tool references |

每种类型对测试、文档详细度和自由度有不同要求。

---

## 目录结构模式

### Simple（简单型）
```
defense-in-depth/
  SKILL.md          # 所有内容内联
```
适用：内容简短，无需外部参考。

### Medium（中等型）
```
condition-based-waiting/
  SKILL.md          # 概述 + 模式
  example.ts        # 可复用的工具代码
```
适用：包含可复用的脚本或工具。

### Complex（复杂型）
```
pptx/
  SKILL.md          # 概述 + 工作流
  pptxgenjs.md      # 600 行 API 参考
  ooxml.md          # 500 行 XML 结构
  scripts/          # 可执行工具
```
适用：参考资料过大无法内联。

---

## Progressive Disclosure 模式

SKILL.md 作为目录，按需加载详细内容。Claude 仅在需要时读取额外文件。

```markdown
# PDF Processing

## Quick start
[内联简短示例]

## Advanced features
**Form filling**: See [FORMS.md](FORMS.md)
**API reference**: See [REFERENCE.md](REFERENCE.md)
```

**关键规则**：引用保持**一层深度**。避免 SKILL.md → advanced.md → details.md 的嵌套。

### Domain-Specific Organization

```
bigquery-skill/
├── SKILL.md                 # 概述 + 导航
└── reference/
    ├── finance.md           # 财务指标
    ├── sales.md             # 销售数据
    └── product.md           # 产品指标
```

用户问销售指标时，Claude 只需读 `reference/sales.md`。

---

## Workflow 模式

将复杂操作分解为清晰步骤，提供可复制的 checklist：

```markdown
## Research synthesis workflow

Copy this checklist and track your progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

---

## Feedback Loop 模式

运行验证器 → 修复错误 → 重复。显著提高输出质量。

```markdown
1. Make edits to document.xml
2. **Validate immediately**: python validate.py unpacked_dir/
3. If validation fails:
   - Review error message
   - Fix issues
   - Run validation again
4. **Only proceed when validation passes**
```

---

## Template 模式

为输出格式提供模板，根据需要选择严格或灵活程度：

- **严格模板**："ALWAYS use this exact template structure"
- **灵活模板**："Here is a sensible default, use your best judgment"

---

## Conditional Workflow 模式

在决策点引导 Claude：

```markdown
1. Determine the modification type:
   **Creating new content?** → Follow "Creation workflow"
   **Editing existing content?** → Follow "Editing workflow"
```

---

## 文件组织模式

| 模式 | 结构 | 适用场景 |
|------|------|---------|
| Self-contained | 仅 SKILL.md | 内容简短 |
| With tool | SKILL.md + 工具脚本 | 包含可复用代码 |
| Heavy reference | SKILL.md + 多个参考文件 + scripts/ | 大量 API 文档 |

---

## Anti-patterns

| 反模式 | 问题 | 修复 |
|--------|------|------|
| Narrative example | 太具体，不可复用 | 提取通用模式 |
| Multi-language dilution | 质量差，维护负担 | 一个优秀示例足够 |
| Code in flowcharts | 无法复制粘贴 | 代码用代码块 |
| Generic labels | 无语义意义 | 使用描述性标签 |
| Windows-style paths | 跨平台失败 | 始终用正斜杠 |
| Too many options | 混淆选择 | 提供默认值 + 逃生口 |
| Deeply nested references | Claude 部分读取 | 保持一层深度 |

---

## 本章核心结论

- SKILL.md 是 skill 的唯一入口，frontmatter 只需 `name` + `description` 两个必填字段
- Skill 分三类：Technique / Pattern / Reference，各有不同的测试和文档策略
- Progressive Disclosure 是核心架构——SKILL.md 作目录，详细内容按需加载
- 引用必须保持一层深度，避免 Claude 部分读取嵌套文件
- Workflow checklist、Feedback Loop、Template 是三种最常用的内容模式
