# 第一章 /brainstorm 命令

> **对应源文件**：`commands/brainstorm.md`

## 概述

`/brainstorm` 命令**已废弃**（Deprecated），将在下一个主要版本中移除。该命令的功能已被 `superpowers:brainstorming` Skill（技能）完全替代。如果你的工作流中仍在使用此命令，请立即迁移到对应的 Skill。

> ⚠️ **废弃通知**：当用户调用 `/brainstorm` 时，Agent 会直接告知用户该命令已废弃，并建议改用 `superpowers:brainstorming` Skill。

---

## 1 命令定义

### 原始用途

| 维度 | 说明 |
|------|------|
| 命令名 | `/brainstorm` |
| 作用 | 启动头脑风暴流程，将模糊想法转化为结构化的设计文档 |
| 现状 | **已废弃**——功能已迁移至 `superpowers:brainstorming` Skill |

### 当前行为

当用户调用 `/brainstorm` 时，Agent 会执行以下操作：

1. 告知用户该命令已废弃，将在下一个 Major Release（主要版本）中移除
2. 引导用户使用 `superpowers:brainstorming` Skill 替代

---

## 2 废弃原因

### Commands 到 Skills 的架构演进

Superpowers 从 Commands（命令）架构迁移到 Skills 架构，原因如下：

| 维度 | Commands 架构 | Skills 架构 |
|------|-------------|------------|
| 触发方式 | 需要用户输入特定命令 | Agent 根据上下文自动识别并使用 |
| 组合能力 | 单个命令独立运行 | Skill 之间可以自由组合和互相调用 |
| 上下文感知 | 无上下文 | 可以感知当前工作流状态 |
| 可扩展性 | 需要平台支持命令注册 | 基于文件系统，平台无关 |

**核心原因**：Skills 是**声明式**的能力描述，Agent 可以根据场景自动决定使用哪个 Skill；而 Commands 是**命令式**的，需要用户显式触发。这意味着 Skills 能更好地融入自然对话流。

---

## 3 迁移指南

### 从 `/brainstorm` 迁移到 `superpowers:brainstorming`

**旧方式**（已废弃）：

```
用户: /brainstorm
Agent: [启动头脑风暴流程]
```

**新方式**（推荐）：

```
用户: 我想设计一个用户认证系统，请帮我头脑风暴一下
Agent: [自动识别并使用 superpowers:brainstorming Skill]
```

或者显式请求：

```
用户: 请使用 superpowers brainstorming Skill 来讨论这个设计
Agent: [使用 superpowers:brainstorming Skill]
```

### `superpowers:brainstorming` Skill 的核心流程

1. 探索上下文、提出澄清问题
2. 提出 2-3 种候选方案
3. 呈现设计文档
4. 自我审查（Placeholder 扫描、一致性检查、范围检查）
5. 用户审核
6. 衔接 `superpowers:writing-plans` Skill 生成实施计划

> **硬门禁**：在设计呈现并获得用户批准之前，**不得编写任何代码**或采取实施行动。

---

## 4 原始命令内容

以下是 `commands/brainstorm.md` 的完整内容：

```yaml
---
description: "Deprecated - use the superpowers:brainstorming skill instead"
---
```

```markdown
Tell your human partner that this command is deprecated and will be
removed in the next major release. They should ask you to use the
"superpowers brainstorming" skill instead.
```

---

## 本章核心结论

1. **`/brainstorm` 已废弃**——将在下一个 Major Release 中移除，不应在新工作流中使用
2. **替代方案是 `superpowers:brainstorming` Skill**——功能更强大，支持自动触发和 Skill 间组合
3. **迁移方式是自然语言触发**——不再需要输入特定命令，直接描述需求即可
