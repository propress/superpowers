# 第二章 /write-plan 命令

> **对应源文件**：`commands/write-plan.md`

## 概述

`/write-plan` 命令**已废弃**（Deprecated），将在下一个主要版本中移除。该命令的功能已被 `superpowers:writing-plans` Skill（技能）完全替代。如果你的工作流中仍在使用此命令，请立即迁移到对应的 Skill。

> ⚠️ **废弃通知**：当用户调用 `/write-plan` 时，Agent 会直接告知用户该命令已废弃，并建议改用 `superpowers:writing-plans` Skill。

---

## 1 命令定义

### 原始用途

| 维度 | 说明 |
|------|------|
| 命令名 | `/write-plan` |
| 作用 | 根据设计文档生成结构化的实施计划（Implementation Plan） |
| 现状 | **已废弃**——功能已迁移至 `superpowers:writing-plans` Skill |

### 当前行为

当用户调用 `/write-plan` 时，Agent 会执行以下操作：

1. 告知用户该命令已废弃，将在下一个 Major Release（主要版本）中移除
2. 引导用户使用 `superpowers:writing-plans` Skill 替代

---

## 2 废弃原因

### Commands 到 Skills 的架构演进

与所有 Commands 的废弃原因一致——Superpowers 从 Commands 架构迁移到 Skills 架构：

| 维度 | Commands 架构 | Skills 架构 |
|------|-------------|------------|
| 触发方式 | 需要用户输入 `/write-plan` | Agent 在 Brainstorming 完成后自动衔接 |
| 工作流集成 | 孤立执行 | 与 `superpowers:brainstorming` 无缝衔接 |
| 输出规范 | 无统一格式 | 强制标准化的 Plan 文档格式 |
| 质量保障 | 无自检 | 内置 Self-Review（自检）流程 |

**核心原因**：`/write-plan` 是孤立的命令——它不知道 Brainstorming 阶段产出了什么，也不会自动引导到 Execution 阶段。而 `superpowers:writing-plans` Skill 天然地串联在 Brainstorming → Planning → Execution 工作流中。

---

## 3 迁移指南

### 从 `/write-plan` 迁移到 `superpowers:writing-plans`

**旧方式**（已废弃）：

```
用户: /write-plan
Agent: [生成计划文档]
```

**新方式**（推荐）：

```
用户: 设计已经确定了，请帮我写实施计划
Agent: [自动识别并使用 superpowers:writing-plans Skill]
```

或者在 Brainstorming 完成后自动触发：

```
Agent: 设计文档已完成，现在为你生成实施计划...
       [自动调用 superpowers:writing-plans Skill]
```

### `superpowers:writing-plans` Skill 的核心特性

- **零上下文假设**：Plan 假设工程师对代码库一无所知，每个步骤都是完整自包含的
- **Bite-sized Task 粒度**：每个步骤 2-5 分钟（写测试 → 运行失败 → 实现 → 运行通过 → Commit）
- **禁止 Placeholder**：不允许 "TBD"、"implement later" 等模糊步骤
- **Self-Review 内置**：Spec 覆盖检查、Placeholder 扫描、类型一致性验证
- **输出路径**：`docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- **执行衔接**：完成后提供 Subagent-Driven（推荐）或 Inline Execution 两种选择

---

## 4 原始命令内容

以下是 `commands/write-plan.md` 的完整内容：

```yaml
---
description: "Deprecated - use the superpowers:writing-plans skill instead"
---
```

```markdown
Tell your human partner that this command is deprecated and will be
removed in the next major release. They should ask you to use the
"superpowers writing-plans" skill instead.
```

---

## 本章核心结论

1. **`/write-plan` 已废弃**——将在下一个 Major Release 中移除，不应在新工作流中使用
2. **替代方案是 `superpowers:writing-plans` Skill**——提供标准化 Plan 格式、Self-Review 和自动工作流衔接
3. **迁移方式是自然语言触发或工作流自动衔接**——Brainstorming 完成后会自动引导到 Plan 编写阶段
