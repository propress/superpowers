# 第三章 /execute-plan 命令

> **对应源文件**：`commands/execute-plan.md`

## 概述

`/execute-plan` 命令**已废弃**（Deprecated），将在下一个主要版本中移除。该命令的功能已被 `superpowers:executing-plans` Skill（技能）完全替代。如果你的工作流中仍在使用此命令，请立即迁移到对应的 Skill。

> ⚠️ **废弃通知**：当用户调用 `/execute-plan` 时，Agent 会直接告知用户该命令已废弃，并建议改用 `superpowers:executing-plans` Skill。

---

## 1 命令定义

### 原始用途

| 维度 | 说明 |
|------|------|
| 命令名 | `/execute-plan` |
| 作用 | 按照实施计划逐步执行开发任务 |
| 现状 | **已废弃**——功能已迁移至 `superpowers:executing-plans` Skill |

### 当前行为

当用户调用 `/execute-plan` 时，Agent 会执行以下操作：

1. 告知用户该命令已废弃，将在下一个 Major Release（主要版本）中移除
2. 引导用户使用 `superpowers:executing-plans` Skill 替代

---

## 2 废弃原因

### Commands 到 Skills 的架构演进

与所有 Commands 的废弃原因一致——Superpowers 从 Commands 架构迁移到 Skills 架构：

| 维度 | Commands 架构 | Skills 架构 |
|------|-------------|------------|
| 触发方式 | 需要用户输入 `/execute-plan` | Agent 在 Plan 编写完成后自动衔接 |
| 审查机制 | 无内置审查 | 内置 Verification Checkpoint（验证检查点） |
| 分支管理 | 无分支感知 | 集成 Git Worktree 管理 |
| 推荐替代 | — | `superpowers:subagent-driven-development` 效果更佳 |

**核心原因**：`/execute-plan` 是一个简单的串行执行器——它不理解分支管理、不集成代码审查、不支持 Subagent 并行。而 `superpowers:executing-plans` Skill 与 Git Worktrees、Verification、Code Review 等 Skill 深度集成。

> **补充说明**：对于复杂项目，Superpowers 推荐使用 `superpowers:subagent-driven-development` Skill 代替 `superpowers:executing-plans`——前者通过独立 Subagent 执行每个 Task，具有更好的隔离性和审查能力。

---

## 3 迁移指南

### 从 `/execute-plan` 迁移到 Skill 体系

**旧方式**（已废弃）：

```
用户: /execute-plan
Agent: [开始执行计划]
```

**新方式 A** — 使用 `superpowers:executing-plans`（Inline Execution）：

```
用户: 计划已经写好了，请开始执行
Agent: [自动识别并使用 superpowers:executing-plans Skill]
```

**新方式 B** — 使用 `superpowers:subagent-driven-development`（推荐）：

```
用户: 请用 Subagent 驱动的方式执行这个计划
Agent: [使用 superpowers:subagent-driven-development Skill]
```

### 两种执行 Skill 的对比

| 维度 | `executing-plans` | `subagent-driven-development` |
|------|------------------|-------------------------------|
| 执行方式 | 在当前会话中串行执行 | 每个 Task 派遣独立 Subagent |
| 上下文隔离 | 共享上下文 | 每个 Task 独立上下文 |
| 审查集成 | 批量审查（每 3 个 Task） | 每个 Task 后两阶段审查 |
| 适用场景 | 简单、线性的执行任务 | 复杂、多步骤的开发项目 |
| 推荐程度 | 可用 | **推荐** |

### `superpowers:executing-plans` Skill 的核心流程

1. **Load and Review Plan** — 加载 Plan，批判性审查，发现问题先与人类讨论
2. **Execute Tasks** — 逐步执行：标记 in_progress → 执行步骤 → 运行验证 → 标记 completed
3. **Complete Development** — 所有 Task 完成后调用 `superpowers:finishing-a-development-branch`

> **安全规则**：永远不在 main/master 分支上直接开始执行，除非用户明确同意。

---

## 4 原始命令内容

以下是 `commands/execute-plan.md` 的完整内容：

```yaml
---
description: "Deprecated - use the superpowers:executing-plans skill instead"
---
```

```markdown
Tell your human partner that this command is deprecated and will be
removed in the next major release. They should ask you to use the
"superpowers executing-plans" skill instead.
```

---

## 本章核心结论

1. **`/execute-plan` 已废弃**——将在下一个 Major Release 中移除，不应在新工作流中使用
2. **替代方案有两个**——`superpowers:executing-plans`（Inline 执行）和 `superpowers:subagent-driven-development`（Subagent 驱动，推荐）
3. **迁移方式是自然语言触发或工作流自动衔接**——Plan 编写完成后会自动提供执行选项
