# 第一章 Hooks 系统

> **对应源文件**：`hooks/hooks.json`、`hooks/hooks-cursor.json`、`hooks/session-start`、`hooks/run-hook.cmd`

## 概述

Hooks(钩子) 是 Superpowers 与宿主平台之间的桥梁。它们在 Session(会话) 生命周期的关键节点自动执行，将 Skill(技能) 上下文注入到 AI Agent(智能代理) 的工作环境中。没有 Hooks，Agent 不会知道自己拥有 Superpowers——Hooks 使"开箱即用"成为可能。

## 前置阅读

- 架构总览章节（了解 Plugin 目录结构与 Skill 发现机制）

---

## 1 Hooks 的作用

Hooks 解决一个核心问题：**如何在 Agent 会话启动时，自动将 Superpowers 的引导上下文（bootstrap context）注入到对话中？**

没有 Hooks 的情况下，用户必须手动告诉 Agent "你有超能力"——这显然不可接受。Hooks 实现了以下目标：

1. **零配置激活**：插件安装后，下次会话自动获得 Superpowers 上下文
2. **平台适配**：同一套逻辑同时支持 Claude Code 和 Cursor
3. **跨操作系统**：通过 Polyglot Wrapper（多语言包装器），Windows、macOS、Linux 均可运行
4. **Legacy 兼容**：检测旧版目录结构并给出迁移提示

---

## 2 事件类型

Superpowers 当前使用的 Hook 事件类型：

| 事件名称 | 触发时机 | 说明 |
|----------|---------|------|
| `SessionStart` / `sessionStart` | 新会话启动 | 注入 `using-superpowers` Skill 的完整内容 |
| `startup` | 首次启动会话 | Claude Code matcher 匹配项之一 |
| `clear` | 用户执行清屏/重置 | 会话重置后重新注入上下文 |
| `compact` | 上下文压缩时 | 确保压缩后仍保留 Superpowers 上下文 |

Claude Code 通过 `matcher` 正则表达式匹配具体的子事件；Cursor 则统一在 `sessionStart` 时触发，不区分子事件。

---

## 3 Claude Code 的 hooks.json

文件路径：`hooks/hooks.json`

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

### 字段详解

| 字段 | 类型 | 说明 |
|------|------|------|
| `hooks` | Object | 顶层容器，按事件类型组织 |
| `SessionStart` | Array | 事件名称，**PascalCase**，Claude Code 规范 |
| `matcher` | String | 正则表达式，匹配 `startup`、`clear`、`compact` 三种子事件 |
| `type` | String | 固定为 `"command"`，表示执行 Shell 命令 |
| `command` | String | 要执行的命令。`${CLAUDE_PLUGIN_ROOT}` 是 Claude Code 提供的环境变量，指向插件根目录 |
| `async` | Boolean | `false` 表示同步执行——Agent 必须等待 Hook 完成后才开始回复 |

**关键设计决策**：

- 使用 `run-hook.cmd` 而非直接调用 `session-start`，因为 Windows 上 CMD.exe 无法直接执行 Bash 脚本
- 路径用双引号包裹，因为 `${CLAUDE_PLUGIN_ROOT}` 在 Windows 上可能包含空格（如 `C:\Program Files\...`）
- `async: false` 确保上下文在 Agent 回复前已就绪

---

## 4 Cursor 的 hooks-cursor.json

文件路径：`hooks/hooks-cursor.json`

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

### 与 Claude Code 格式的差异

| 差异点 | Claude Code | Cursor |
|--------|------------|--------|
| 事件命名 | `SessionStart`（PascalCase） | `sessionStart`（camelCase） |
| `version` 字段 | 无 | 必填，当前值为 `1` |
| `matcher` | 支持正则匹配子事件 | 不支持，无此字段 |
| 路径风格 | `${CLAUDE_PLUGIN_ROOT}` 绝对路径 | `./` 相对路径（相对于插件根目录） |
| Wrapper | 需要 `run-hook.cmd` | 直接调用脚本 |
| `async` 字段 | 显式设置 | 无此字段 |

Cursor 的 `plugin.json` 通过 `"hooks": "./hooks/hooks-cursor.json"` 字段指向此配置文件。

---

## 5 session-start 脚本解析

文件路径：`hooks/session-start`

这是 Hooks 系统的核心——一个 Bash 脚本，负责构建并输出 JSON 格式的上下文注入数据。

### 5.1 初始化与安全设置

```bash
#!/usr/bin/env bash
set -euo pipefail
```

- `set -e`：任何命令失败立即退出
- `set -u`：使用未定义变量时报错
- `set -o pipefail`：管道中任何命令失败则整个管道失败

### 5.2 确定插件根目录

```bash
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"
```

通过脚本自身位置（`hooks/` 目录）向上一级推导出插件根目录。这确保无论从哪里调用脚本，路径都是正确的。

### 5.3 Legacy 目录检测

```bash
legacy_skills_dir="${HOME}/.config/superpowers/skills"
if [ -d "$legacy_skills_dir" ]; then
    warning_message="\n\n<important-reminder>..."
fi
```

检测旧版 Skill 存放目录 `~/.config/superpowers/skills`。如果存在，生成一条警告消息，告诉用户迁移到 `~/.claude/skills`。这条警告会被注入到 Agent 的上下文中，Agent 在第一次回复时会主动告知用户。

### 5.4 加载 SKILL.md 内容

```bash
using_superpowers_content=$(cat "${PLUGIN_ROOT}/skills/using-superpowers/SKILL.md" 2>&1 \
    || echo "Error reading using-superpowers skill")
```

读取 `using-superpowers` Skill 的完整 Markdown 内容。这是 Superpowers 的入口 Skill，告诉 Agent 如何发现和使用所有其他 Skill。

### 5.5 JSON 转义技术

```bash
escape_for_json() {
    local s="$1"
    s="${s//\\/\\\\}"    # 反斜杠 → \\
    s="${s//\"/\\\"}"    # 双引号 → \"
    s="${s//$'\n'/\\n}"  # 换行符 → \n
    s="${s//$'\r'/\\r}"  # 回车符 → \r
    s="${s//$'\t'/\\t}"  # 制表符 → \t
    printf '%s' "$s"
}
```

**为什么不用 `sed`/`awk`？** 这是一个关键的跨平台设计决策：

- Windows 上通过 Git Bash 运行时，`sed`/`awk` 可能不在 PATH 中
- Bash 内建的 Parameter Substitution（参数替换）在 C 层面执行，比逐字符循环快几个数量级
- 纯 Bash 实现，零外部依赖

### 5.6 构建注入上下文

```bash
session_context="<EXTREMELY_IMPORTANT>\nYou have superpowers.\n\n..."
```

将 Skill 内容包裹在 `<EXTREMELY_IMPORTANT>` 标签中，确保 Agent 高度重视此上下文。

### 5.7 平台检测与输出格式

```bash
if [ -n "${CURSOR_PLUGIN_ROOT:-}" ]; then
  printf '{\n  "additional_context": "%s"\n}\n' "$session_context"
elif [ -n "${CLAUDE_PLUGIN_ROOT:-}" ]; then
  printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$session_context"
else
  printf '{\n  "additional_context": "%s"\n}\n' "$session_context"
fi
```

**平台判断逻辑**：

```
CURSOR_PLUGIN_ROOT 已设置?
  ├─ 是 → Cursor 格式：{ "additional_context": "..." }
  └─ 否 → CLAUDE_PLUGIN_ROOT 已设置?
           ├─ 是 → Claude Code 格式：{ "hookSpecificOutput": { ... } }
           └─ 否 → Fallback：同 Cursor 格式
```

**为什么用 `printf` 而非 Heredoc？** 注释中说明了原因：Bash 5.3+ 存在一个 Bug，当 Heredoc 中的变量展开内容超过约 512 字节时会挂起。`printf` 可以避免此问题。

**为什么不同时输出两种格式？** Claude Code 会读取 **两个** 字段但不去重，同时输出会导致上下文被注入两次。

---

## 6 Windows 支持

### 6.1 run-hook.cmd 多语言包装器

文件路径：`hooks/run-hook.cmd`

```cmd
: << 'CMDBLOCK'
@echo off
REM Windows 批处理部分

if "%~1"=="" (
    echo run-hook.cmd: missing script name >&2
    exit /b 1
)

set "HOOK_DIR=%~dp0"

REM 依次尝试标准安装路径的 Git Bash
if exist "C:\Program Files\Git\bin\bash.exe" (
    "C:\Program Files\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)
if exist "C:\Program Files (x86)\Git\bin\bash.exe" (
    "C:\Program Files (x86)\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)

REM 尝试 PATH 中的 bash
where bash >nul 2>nul
if %ERRORLEVEL% equ 0 (
    bash "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)

REM 未找到 bash，静默退出（插件仍可工作，仅缺少 SessionStart 上下文注入）
exit /b 0
CMDBLOCK

# Unix 部分：直接执行指定脚本
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

### 6.2 Polyglot 原理

这个文件同时是合法的 CMD 批处理和 Bash 脚本：

**Windows CMD 执行路径**：
1. `: << 'CMDBLOCK'` — CMD 将 `:` 视为标签（类似 `:label`），忽略其余部分
2. `@echo off` — 关闭命令回显
3. 在三个标准位置查找 `bash.exe`
4. `exit /b` — 退出批处理，不会执行后面的 Unix 代码

**Unix Bash 执行路径**：
1. `: << 'CMDBLOCK'` — `:` 是 Bash 的空操作，`<< 'CMDBLOCK'` 开始 Heredoc
2. 整个 Windows 批处理部分被 Heredoc 吞掉（忽略）
3. `CMDBLOCK` 结束 Heredoc
4. 执行 Unix 代码部分

### 6.3 Windows 前置要求

- 必须安装 **Git for Windows**（提供 `bash.exe` 和 `cygpath`）
- 默认安装路径：`C:\Program Files\Git\bin\bash.exe`
- Hook 脚本使用无扩展名（如 `session-start` 而非 `session-start.sh`），避免 Claude Code 的 Windows `.sh` 自动检测干扰

### 6.4 常见问题排查

| 症状 | 原因 | 解决方案 |
|------|------|---------|
| "bash is not recognized" | CMD 找不到 bash | 安装 Git for Windows 或检查安装路径 |
| 脚本在文本编辑器中打开 | hooks.json 指向了 `.sh` 文件 | 改为指向 `.cmd` 包装器 |
| 路径中出现 `\/` | Windows 路径与 Unix 路径拼接 | 使用 `cygpath` 转换 |

---

## 7 自定义 Hook 开发

### 7.1 创建新 Hook 的步骤

1. **编写 Bash 脚本**：在 `hooks/` 目录下创建无扩展名的脚本文件
2. **设置执行权限**：`chmod +x hooks/my-hook`
3. **注册到 hooks.json**：添加对应的事件和 matcher
4. **同步更新 hooks-cursor.json**（如果需要支持 Cursor）

### 7.2 Hook 脚本规范

- 使用 `#!/usr/bin/env bash` 和 `set -euo pipefail`
- 仅使用 Bash 内建命令，避免 `sed`、`awk`、`grep` 等外部工具
- 输出必须是合法的 JSON
- 使用 `printf` 而非 Heredoc 输出大内容

### 7.3 输出格式示例

**Claude Code**：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "additionalContext": "你的上下文内容"
  }
}
```

**Cursor / Fallback**：

```json
{
  "additional_context": "你的上下文内容"
}
```

### 7.4 多 Hook 配置示例

使用 `run-hook.cmd` 可复用包装器支持多个 Hook：

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" validate-bash"
          }
        ]
      }
    ]
  }
}
```

---

## 本章核心结论

1. **Hooks 是 Superpowers 自动激活的关键**——它们在会话启动时将 `using-superpowers` Skill 注入 Agent 上下文
2. **Claude Code 和 Cursor 使用不同的 JSON 格式**——事件命名（PascalCase vs camelCase）、输出字段（`hookSpecificOutput` vs `additional_context`）均有差异
3. **session-start 脚本是平台感知的**——通过环境变量自动选择输出格式
4. **跨平台支持通过 Polyglot Wrapper 实现**——`run-hook.cmd` 同时是合法的 CMD 和 Bash 脚本
5. **纯 Bash 实现**——不依赖 `sed`/`awk` 等外部工具，确保在 Git Bash 环境下也能正常运行
