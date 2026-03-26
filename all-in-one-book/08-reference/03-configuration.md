# 配置参考

> 本章列出 Superpowers 框架的所有配置文件，展示完整内容并逐字段解释。

---

## 1 Claude Code 插件配置

### .claude-plugin/plugin.json

```json
{
  "name": "superpowers",
  "description": "Core skills library for Claude Code — a complete software development workflow built on composable skills including brainstorming, test-driven development, systematic debugging, subagent-driven development, and more.",
  "version": "5.0.6",
  "author": {
    "name": "Jesse Vincent",
    "email": "jesse@fsck.com"
  },
  "homepage": "https://github.com/obra/superpowers",
  "repository": "https://github.com/obra/superpowers",
  "license": "MIT",
  "keywords": ["skills", "tdd", "debugging", "collaboration", "best-practices", "workflows"]
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 插件唯一标识符，用于安装和引用 |
| `description` | string | 插件描述，显示在 Marketplace 中 |
| `version` | string | 语义化版本号 |
| `author` | object | 作者信息（name + email） |
| `homepage` | string | 项目主页 URL |
| `repository` | string | Git 仓库 URL |
| `license` | string | 开源许可证类型 |
| `keywords` | array | 搜索关键词，帮助用户发现插件 |

### .claude-plugin/marketplace.json

此文件用于 Claude Code Community Marketplace 发布，包含 Marketplace 特定的元数据。

---

## 2 Cursor 插件配置

### .cursor-plugin/plugin.json

```json
{
  "name": "superpowers",
  "displayName": "Superpowers",
  "description": "Core skills library — a complete software development workflow built on composable skills.",
  "version": "5.0.6",
  "author": {
    "name": "Jesse Vincent",
    "email": "jesse@fsck.com"
  },
  "homepage": "https://github.com/obra/superpowers",
  "repository": "https://github.com/obra/superpowers",
  "license": "MIT",
  "keywords": ["skills", "tdd", "debugging", "collaboration", "best-practices", "workflows"],
  "skills": "./skills/",
  "agents": "./agents/",
  "commands": "./commands/",
  "hooks": "./hooks/hooks-cursor.json"
}
```

| 字段 | 与 Claude Code 的差异 | 说明 |
|------|---------------------|------|
| `displayName` | Cursor 专有 | 在 UI 中显示的友好名称 |
| `skills` | Cursor 专有 | Skill 文件目录路径 |
| `agents` | Cursor 专有 | Agent 文件目录路径 |
| `commands` | Cursor 专有 | Command 文件目录路径 |
| `hooks` | Cursor 专有 | Hook 配置文件路径（指向 camelCase 格式） |

---

## 3 Hook 配置

### hooks/hooks.json（Claude Code）

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

| 字段 | 说明 |
|------|------|
| `SessionStart` | 事件名称（PascalCase），在会话启动时触发 |
| `matcher` | 正则表达式，匹配 `startup`、`clear` 或 `compact` 事件 |
| `type` | Hook 类型，`command` 表示执行命令 |
| `command` | 要执行的命令，`${CLAUDE_PLUGIN_ROOT}` 指向插件安装目录 |
| `async` | `false` 表示同步执行，阻塞直到命令完成 |

**注意**：`matcher` 不包含 `resume`，避免在恢复会话时重复注入上下文。

### hooks/hooks-cursor.json（Cursor）

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      {
        "command": "./hooks/session-start"
      }
    ]
  }
}
```

| 字段 | 与 Claude Code 的差异 | 说明 |
|------|---------------------|------|
| `version` | Cursor 必需 | 配置格式版本号 |
| `sessionStart` | camelCase（非 PascalCase） | Cursor 使用驼峰命名 |
| `command` | 相对路径 | Cursor 使用相对于插件根目录的路径 |

**注意**：Cursor 格式没有 `matcher`、`type`、`async` 字段。

---

## 4 Gemini CLI 扩展配置

### gemini-extension.json

```json
{
  "name": "superpowers",
  "description": "Core skills library for Gemini CLI — a complete software development workflow built on composable skills.",
  "version": "5.0.6",
  "contextFileName": "GEMINI.md"
}
```

| 字段 | 说明 |
|------|------|
| `name` | 扩展标识符 |
| `description` | 扩展描述 |
| `version` | 版本号 |
| `contextFileName` | Gemini CLI 启动时读取的上下文文件名 |

### GEMINI.md

```markdown
@./skills/using-superpowers/SKILL.md
@./skills/using-superpowers/references/gemini-tools.md
```

`@` 语法是 Gemini CLI 的文件导入指令，启动时将 `using-superpowers` Skill 和工具映射表注入上下文。

---

## 5 OpenCode 插件配置

### .opencode/plugins/superpowers.js

OpenCode 使用 JavaScript 插件而非 JSON 配置。插件导出两个钩子：

```javascript
// config hook - 注册 Skill 目录
config: async (config) => {
  config.skills = config.skills || {};
  config.skills.paths = config.skills.paths || [];
  if (!config.skills.paths.includes(superpowersSkillsDir)) {
    config.skills.paths.push(superpowersSkillsDir);
  }
}

// system prompt hook - 注入启动上下文
'experimental.chat.system.transform': async (_input, output) => {
  const bootstrap = getBootstrapContent();
  if (bootstrap) {
    (output.system ||= []).push(bootstrap);
  }
}
```

| 钩子 | 作用 |
|------|------|
| `config` | 将 `skills/` 目录注册到 OpenCode 的 Skill 发现路径 |
| `experimental.chat.system.transform` | 在系统提示中注入 `using-superpowers` 内容 |

**用户配置（opencode.json）**：

```json
{
  "plugin": ["superpowers@git+https://github.com/obra/superpowers.git"]
}
```

固定版本：`"superpowers@git+https://github.com/obra/superpowers.git#v5.0.3"`

---

## 6 NPM 包配置

### package.json

```json
{
  "name": "superpowers",
  "version": "5.0.6",
  "description": "Core skills library for coding agents",
  "type": "module",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/obra/superpowers.git"
  }
}
```

| 字段 | 说明 |
|------|------|
| `type` | `"module"` — 启用 ESM 模式（这就是 server.js 需要改名为 server.cjs 的原因） |

---

## 7 Codex 配置

Codex 不使用配置文件，而是通过目录约定进行 Skill 发现：

```
~/.agents/skills/superpowers → symlink → ~/.codex/superpowers/skills
```

Codex 会自动扫描 `~/.agents/skills/` 下的所有子目录，发现并加载 SKILL.md 文件。

---

## 本章核心结论

1. **平台选择**：Claude Code 和 Cursor 使用 JSON 声明式配置；OpenCode 使用 JS 编程式配置；Gemini CLI 使用文件导入；Codex 使用目录约定。
2. **Hook 格式差异**：Claude Code 用 PascalCase + matcher + async；Cursor 用 camelCase + version，没有 matcher。
3. **版本同步**：所有平台配置文件中的 `version` 字段应保持一致。发布新版本时需同步更新所有配置文件。
