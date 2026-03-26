# 第七章·第二节 CSO 策略

## 概述

CSO(Claude Search Optimization，Claude 搜索优化) 是让未来的 Claude 实例能够**发现并正确加载**你的 Skill 的一套策略。如果 Claude 找不到你的 skill，这个 skill 就等于不存在。CSO 的核心是 `description` 字段——它决定了 skill 是否会被选中。

---

## 什么是 CSO

启动时，所有 Skill 的 metadata（`name` + `description`）被预加载到 system prompt。当用户发送消息时，Claude 根据 description 决定加载哪个 skill 的完整内容。这意味着：

1. **description 是第一道门**——写不好，skill 永远不会被读取
2. **description 是唯一被始终加载的部分**——SKILL.md body 只在触发后才读

---

## "中间遗忘" 问题解释

测试发现了一个关键问题：当 description **总结了 skill 的工作流**时，Claude 会直接按 description 行事，而跳过阅读 SKILL.md 的详细内容。

**实际案例**：description 说 "code review between tasks"，Claude 做了**一次** review——尽管 SKILL.md 的流程图明确显示需要**两次** review（spec compliance 然后 code quality）。

把 description 改为纯触发条件 "Use when executing implementation plans with independent tasks" 后，Claude 正确读取了流程图并执行了两阶段 review。

**陷阱**：总结工作流的 description 创建了一条捷径——Claude 会走捷径。SKILL.md body 变成了被跳过的文档。

---

## description 字段黄金规则

**description = When to Use, NOT What the Skill Does**

| 规则 | 说明 |
|------|------|
| 以 "Use when..." 开头 | 聚焦触发条件 |
| 第三人称 | 注入 system prompt 时保持一致 |
| 只描述触发条件 | 症状、场景、上下文 |
| **绝不**总结工作流 | 不提步骤、不提流程 |
| 500 字符以内 | 保持简洁 |

---

## Good vs Bad Descriptions

```yaml
# ❌ BAD: 总结了工作流
description: Use when executing plans - dispatches subagent per task with code review between tasks

# ❌ BAD: 太多流程细节
description: Use for TDD - write test first, watch it fail, write minimal code, refactor

# ❌ BAD: 太抽象
description: For async testing

# ❌ BAD: 第一人称
description: I can help you with async tests when they're flaky

# ❌ BAD: 提到技术但 skill 不限于该技术
description: Use when tests use setTimeout/sleep and are flaky

# ✅ GOOD: 纯触发条件
description: Use when executing implementation plans with independent tasks in the current session

# ✅ GOOD: 描述问题而非技术
description: Use when tests have race conditions, timing dependencies, or pass/fail inconsistently

# ✅ GOOD: 特定技术的 skill 明确声明
description: Use when using React Router and handling authentication redirects

# ✅ GOOD: 包含违规症状
description: Use when implementing any feature or bugfix, before writing implementation code
```

---

## 关键词覆盖策略

使用 Claude 会搜索的词汇：

| 类别 | 示例 |
|------|------|
| 错误信息 | "Hook timed out", "ENOTEMPTY", "race condition" |
| 症状 | "flaky", "hanging", "zombie", "pollution" |
| 同义词 | "timeout/hang/freeze", "cleanup/teardown/afterEach" |
| 工具名 | 具体命令、库名、文件类型 |

---

## 动词优先命名

Skill 名称使用主动语态、动词优先：

| ✅ Good | ❌ Bad |
|---------|--------|
| `creating-skills` | `skill-creation` |
| `condition-based-waiting` | `async-test-helpers` |
| `root-cause-tracing` | `debugging-techniques` |
| `using-skills` | `skill-usage` |

Gerund（-ing 形式）适合描述过程：`creating-skills`, `testing-skills`, `debugging-with-logs`。

---

## 第三人称规则

description 会被注入 system prompt，因此必须使用第三人称：

```yaml
# ✅ GOOD
description: Processes Excel files and generates reports

# ❌ BAD
description: I can help you process Excel files

# ❌ BAD
description: You can use this to process Excel files
```

---

## Token 效率与 description 长度

description 在**每次会话**中都会被加载（作为 metadata 的一部分），因此应尽量简短。但更重要的是**信息密度**——500 字符以内包含足够的触发信号。

---

## 与 Skill 触发的关系

description 直接影响两个测试维度：

1. **Skill Triggering Test**：朴素提示（不提 skill 名称）能否通过 description 匹配触发 skill
2. **Explicit Skill Request Test**：显式请求后 Claude 是否正确加载而非 "I know what that means"

好的 description 让两条路径都畅通。

---

## 本章核心结论

- CSO 的核心是 `description` 字段——它决定 skill 是否被发现和加载
- **黄金规则**：description 只写触发条件（"Use when..."），绝不总结工作流
- 总结工作流的 description 会导致 Claude 走捷径跳过 SKILL.md 详细内容
- 使用第三人称、动词优先命名、关键词覆盖来最大化可发现性
- description 是每次会话必加载的内容，保持简洁但信息密度高
