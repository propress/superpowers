# 第七章·第四节 多平台支持

## 概述

Superpowers 支持多个 AI coding agent 平台：Claude Code、Cursor、Codex、OpenCode 和 Gemini CLI。虽然核心 skill 内容跨平台共享，但每个平台的安装方式、hook 格式、tool 名称和 skill 发现机制各有不同。

---

## 平台对比表

| 特性 | Claude Code | Cursor | Codex | OpenCode | Gemini CLI |
|------|-----------|--------|-------|----------|------------|
| **安装方式** | Plugin marketplace | Plugin marketplace | Clone + 指令 | Clone + 插件配置 | `gemini extensions install` |
| **Skill 发现** | Skill tool | Skill tool | 原生 skill discovery | 原生 `skill` tool | `activate_skill` |
| **Subagent 支持** | ✅ Task tool | ✅ | ✅ spawn_agent | ✅ @mention | ❌ 回退到 executing-plans |
| **Hook 系统** | hooks.json | hooks-cursor.json | N/A | 插件 hook | N/A |
| **配置文件** | `.claude-plugin/plugin.json` | `.cursor-plugin/plugin.json` | `.codex/INSTALL.md` | `.opencode/plugins/superpowers.js` | `gemini-extension.json` |
| **Context 注入** | SessionStart hook | sessionStart hook | AGENTS.md | system.transform | GEMINI.md @import |

---

## 共同架构

所有平台共享同一个目录结构：

```
superpowers/
├── skills/                  # 14 个 skill（所有平台共享）
│   ├── brainstorming/
│   ├── test-driven-development/
│   ├── systematic-debugging/
│   ├── using-superpowers/
│   └── ...
├── agents/                  # Agent 定义
│   └── code-reviewer.md
├── commands/                # 斜杠命令（已弃用）
│   ├── brainstorm.md
│   ├── write-plan.md
│   └── execute-plan.md
└── hooks/                   # 会话启动 hook
    ├── hooks.json           # Claude Code 格式
    ├── hooks-cursor.json    # Cursor 格式
    ├── run-hook.cmd         # 跨平台 hook 启动器
    └── session-start        # Hook 脚本
```

---

## 平台差异点

### Claude Code

- 主配置：`.claude-plugin/plugin.json`
- Hook 格式：`SessionStart`（PascalCase），`async: false`
- Skill 调用：`Skill` tool，自动加载内容
- 命名空间：`superpowers:skill-name`

### Cursor

- 主配置：`.cursor-plugin/plugin.json`（含 `displayName` 和 `skills`/`agents`/`commands` 路径）
- Hook 格式：`sessionStart`（camelCase），`version: 1`
- 安装：`/add-plugin superpowers`

### Codex

- 无配置文件，通过指令安装
- Skill 发现：`~/.agents/skills/superpowers/` symlink
- Tool 映射：`TodoWrite` → `update_plan`，Task → `spawn_agent`
- 无 subagent → 回退到手动工作流

### OpenCode

- 配置：`.opencode/plugins/superpowers.js`（ES module 插件）
- 通过 `config` hook 自动注册 skills 目录
- 通过 `experimental.chat.system.transform` 注入 bootstrap context
- Tool 映射：`TodoWrite` → `todowrite`，Task → @mention

### Gemini CLI

- 配置：`gemini-extension.json` + `GEMINI.md`
- `GEMINI.md` 通过 `@import` 加载 `using-superpowers` skill
- 无 subagent 支持——skill 回退到 `executing-plans`
- Tool 映射参考：`skills/using-superpowers/references/gemini-tools.md`

---

## Hook 格式差异

### Claude Code (`hooks/hooks.json`)

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
        "async": false
      }]
    }]
  }
}
```

### Cursor (`hooks/hooks-cursor.json`)

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [{
      "command": "./hooks/session-start"
    }]
  }
}
```

**关键差异**：Claude Code 用 PascalCase + matcher；Cursor 用 camelCase + version 字段。

---

## Skill 发现差异

| 平台 | 机制 | 说明 |
|------|------|------|
| Claude Code | `Skill` tool | 自动列出可用 skill，选中后加载 SKILL.md |
| Cursor | `Skill` tool | 类似 Claude Code |
| Codex | 原生 discovery | 通过 `~/.agents/skills/` symlink 发现 |
| OpenCode | `skill` tool | 插件通过 `config.skills.paths` 注册目录 |
| Gemini CLI | `activate_skill` | 通过 `gemini-extension.json` 注册 |

---

## Tool 映射差异

Skills 使用 Claude Code tool 名称，其他平台需要替换：

| Claude Code Tool | Codex | OpenCode | Gemini CLI |
|-----------------|-------|----------|------------|
| `Read` | read | read | `read_file` |
| `Write` | write | write | `write_file` |
| `Edit` | edit | edit | `replace` |
| `Bash` | exec | bash | `run_bash_command` |
| `Task` (subagent) | `spawn_agent` | @mention | ❌ 不支持 |
| `TodoWrite` | `update_plan` | `todowrite` | ❌ |
| `Skill` | skill | skill | `activate_skill` |

---

## 跨平台开发建议

1. **Skill 内容保持平台无关**——使用 Claude Code tool 名称，平台适配在外部处理
2. **测试覆盖多平台**——至少在 Claude Code 和一个非 Claude Code 平台上验证
3. **注意 subagent 限制**——Gemini CLI 无 subagent 支持，skill 应有回退路径
4. **Hook 脚本保持 POSIX 兼容**——使用 `#!/usr/bin/env bash`，避免 bash-specific 语法
5. **路径使用正斜杠**——Windows 也用 `/`，不用 `\`

---

## 本章核心结论

- Superpowers 支持 5 个平台，核心 skill 内容共享，平台差异在配置层处理
- Hook 格式是最大差异点：Claude Code 用 PascalCase，Cursor 用 camelCase
- Tool 名称需要平台间映射，参考 `references/codex-tools.md` 和 `references/gemini-tools.md`
- Gemini CLI 无 subagent 支持，所有涉及 Task tool 的 skill 需回退到 executing-plans
- 跨平台 skill 应保持 POSIX 兼容和平台无关的内容
