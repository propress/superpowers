# 第七章·第五节 扩展 Superpowers

## 概述

Superpowers 由四类可扩展组件构成：Skill(技能)、Agent(代理)、Command(命令) 和 Hook(钩子)。每类组件都支持自定义创建，让你可以将自己的工作流和最佳实践融入 Superpowers 体系。

---

## 创建自定义 Skill

### 目录结构

在 `skills/` 下创建新目录：

```
skills/
  my-custom-skill/
    SKILL.md              # 主文件（必须）
    reference.md          # 可选：参考资料
    scripts/              # 可选：工具脚本
```

### SKILL.md 模板

```markdown
---
name: my-custom-skill
description: Use when [specific triggering conditions]
---

# My Custom Skill

## Overview
Core principle in 1-2 sentences.

## When to Use
- Symptom or situation 1
- Symptom or situation 2

## Core Pattern
[Technique or pattern description with code examples]

## Quick Reference
| Operation | Command/Method |
|-----------|---------------|
| ... | ... |

## Common Mistakes
- ❌ Mistake → ✅ Fix
```

### 命名规范

- 仅使用小写字母、数字、连字符：`my-skill-name`
- 动词优先（gerund 形式）：`debugging-with-logs` > `log-debugging`
- 描述行为而非工具：`condition-based-waiting` > `async-helpers`

### 必须遵循的流程

**Iron Law: No Skill Without a Failing Test First**

1. 创建 pressure scenario（RED）
2. 不带 skill 运行，记录 agent 失败（RED）
3. 编写 SKILL.md（GREEN）
4. 带 skill 运行，验证通过（GREEN）
5. 堵住漏洞（REFACTOR）

---

## 创建自定义 Agent

Agent 定义文件放在 `agents/` 目录中：

```
agents/
  my-agent.md
```

### Agent 模板

```markdown
# My Agent

You are a [role description]. Your job is to [specific task].

## Checklist
- [ ] Step 1
- [ ] Step 2
- [ ] Step 3

## Output Format
[Expected output format description]
```

Superpowers 内置了 `code-reviewer.md` agent，用于代码审查工作流。自定义 agent 可以用 `superpowers:agent-name` 命名空间引用。

---

## 创建自定义 Command

Command 是用户可手动调用的斜杠命令，放在 `commands/` 目录中：

```
commands/
  my-command.md
```

### Command 模板

```markdown
Use superpowers:my-related-skill to [action].
```

> **注意**：v5.0.0 起，斜杠命令已弃用。推荐直接使用 skill 而非命令。现有命令 `/brainstorm`、`/write-plan`、`/execute-plan` 仅作为 skill 的重定向。

---

## 创建自定义 Hook

Hook 在会话生命周期事件时触发。目前主要使用 `SessionStart` hook。

### Claude Code 格式

在 `hooks/hooks.json` 中添加：

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "your-command-here",
        "async": false
      }]
    }]
  }
}
```

### Cursor 格式

在 `hooks/hooks-cursor.json` 中添加：

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [{
      "command": "./hooks/your-script"
    }]
  }
}
```

### Hook 脚本注意事项

- 使用 `#!/usr/bin/env bash` 而非 `#!/bin/bash`（跨平台兼容）
- 避免 heredoc 中的大变量展开（Bash 5.3+ 回归 bug）
- 使用 `$0` 而非 `${BASH_SOURCE[0]:-$0}`（POSIX 兼容）
- `async: false` 确保 hook 在第一轮对话前完成

---

## 发布到社区

### 贡献到核心库

1. Fork `obra/superpowers` 仓库
2. 创建功能分支
3. 遵循 `writing-skills` skill 创建和测试新 skill
4. 提交 PR（遵循 PR 模板）

### PR 要求

- 描述解决的具体问题
- 说明为何适合核心库（通用性）
- 进行对抗性测试（不仅是 happy path）
- 人类审查了完整 diff
- 已在至少一个平台上测试

### 发布为独立插件

如果 skill 是特定领域或第三方集成，应作为独立插件发布而非贡献到核心。

---

## 版本管理

Superpowers 使用语义化版本（SemVer）：

- `package.json` 中的 `version` 字段
- 各平台配置文件中同步版本号
- 通过 `/plugin update superpowers` 更新

当前版本：**5.0.6**

---

## 本章核心结论

- 四类可扩展组件：Skill、Agent、Command、Hook
- 创建 Skill 必须遵循 TDD 流程——Iron Law: No Skill Without a Failing Test First
- Command 已弃用，推荐直接使用 Skill
- Hook 脚本需保持 POSIX 兼容和跨平台可用
- 通用 skill 贡献到核心库，特定领域 skill 作独立插件发布
