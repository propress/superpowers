# 第六章·第一节 测试基础设施

## 概述

Superpowers 的测试体系围绕 Claude Code CLI(命令行界面) 的 **headless mode(无头模式)** 构建——通过 `claude -p` 在无交互环境下执行真实 AI 会话，再对产生的 **session transcript(会话记录)** 进行自动化断言。这套体系不是传统的单元测试，而是 **行为验证**：它验证 skill 是否被正确加载、agent 是否遵循了 skill 的工作流、以及最终产物是否符合预期。

整个测试基础设施由三大组件构成：

| 组件 | 路径 | 职责 |
|------|------|------|
| test-helpers.sh | `tests/claude-code/test-helpers.sh` | 共享断言函数库 |
| run-skill-tests.sh | `tests/claude-code/run-skill-tests.sh` | 测试运行器 |
| analyze-token-usage.py | `tests/claude-code/analyze-token-usage.py` | Token 开销分析 |

---

## 测试目录结构

```
tests/
├── claude-code/
│   ├── test-helpers.sh                         # 共享测试工具函数
│   ├── run-skill-tests.sh                      # 测试运行器（协调所有测试）
│   ├── analyze-token-usage.py                  # Token 使用分析工具
│   ├── test-subagent-driven-development.sh     # 快速测试：skill 内容验证
│   └── test-subagent-driven-development-integration.sh  # 集成测试：完整工作流
├── skill-triggering/
│   ├── run-test.sh                             # 单个 skill 触发测试
│   ├── run-all.sh                              # 批量触发测试
│   └── prompts/                                # 朴素提示词（不提 skill 名称）
│       ├── test-driven-development.txt
│       ├── systematic-debugging.txt
│       ├── writing-plans.txt
│       ├── executing-plans.txt
│       ├── dispatching-parallel-agents.txt
│       └── requesting-code-review.txt
├── explicit-skill-requests/
│   ├── run-test.sh                             # 显式请求测试
│   ├── run-all.sh                              # 批量测试
│   ├── run-multiturn-test.sh                   # 多轮对话测试
│   ├── run-extended-multiturn-test.sh          # 扩展多轮测试
│   ├── run-haiku-test.sh                       # Haiku 模型测试
│   ├── run-claude-describes-sdd.sh             # SDD 描述测试
│   └── prompts/                                # 显式请求提示词
│       ├── subagent-driven-development-please.txt
│       ├── please-use-brainstorming.txt
│       ├── use-systematic-debugging.txt
│       └── ...
├── brainstorm-server/                          # 头脑风暴服务器测试
├── subagent-driven-dev/                        # SDD 端到端测试项目
│   ├── go-fractals/                            # Go CLI 工具（10 个任务）
│   └── svelte-todo/                            # Svelte CRUD 应用（12 个任务）
└── opencode/                                   # OpenCode 平台测试
```

---

## test-helpers.sh API

`test-helpers.sh` 是所有 Claude Code 测试共享的函数库，提供 6 个核心函数。

### run_claude

```bash
run_claude "prompt text" [timeout_seconds] [allowed_tools]
```

在 headless mode 下执行 Claude，捕获输出并返回。

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `prompt` | 必填 | 发送给 Claude 的提示文本 |
| `timeout` | `60` | 超时时间（秒） |
| `allowed_tools` | 空 | 允许使用的工具（逗号分隔） |

**内部实现**：创建临时文件接收输出，通过 `timeout` 命令控制超时，成功时返回 stdout，失败时输出到 stderr 并返回非零退出码。

```bash
# 典型用法
output=$(run_claude "What does the test-driven-development skill say?" 30)
```

### assert_contains

```bash
assert_contains "output" "pattern" "test name"
```

验证输出中**包含**指定模式。内部使用 `grep -q` 匹配。

- 成功时打印 `[PASS] test name`，返回 0
- 失败时打印 `[FAIL] test name`，显示期望模式和实际输出，返回 1

### assert_not_contains

```bash
assert_not_contains "output" "pattern" "test name"
```

验证输出中**不包含**指定模式。逻辑与 `assert_contains` 相反。

### assert_count

```bash
assert_count "output" "pattern" expected_count "test name"
```

验证模式在输出中出现的**精确次数**。内部使用 `grep -c` 统计。

```bash
# 验证 TodoWrite 被调用了 5 次
assert_count "$transcript" "TodoWrite" 5 "Task tracking calls"
```

### assert_order

```bash
assert_order "output" "pattern_a" "pattern_b" "test name"
```

验证 pattern_a 在输出中出现的行号**早于** pattern_b。用于验证工作流顺序。

```bash
# 验证 spec review 在 code review 之前
assert_order "$output" "spec compliance" "code quality" "Review order correct"
```

**实现细节**：通过 `grep -n` 获取两个模式的首次出现行号，比较行号大小。如果任一模式未找到，测试失败。

### create_test_project / create_test_plan

```bash
test_project=$(create_test_project)
plan_file=$(create_test_plan "$project_dir" "plan-name")
```

- `create_test_project`：创建临时目录并返回路径
- `create_test_plan`：在指定项目中创建标准 implementation plan 文件（含 Task 1: Hello Function 和 Task 2: Goodbye Function）
- `cleanup_test_project`：清理临时目录（建议配合 `trap` 使用）

```bash
TEST_PROJECT=$(create_test_project)
trap "cleanup_test_project $TEST_PROJECT" EXIT
create_test_plan "$TEST_PROJECT"
```

---

## 快速测试 vs 集成测试

Superpowers 区分两类测试，它们的目标和执行成本截然不同：

| 维度 | 快速测试（Fast Tests） | 集成测试（Integration Tests） |
|------|----------------------|------------------------------|
| **目标** | 验证 skill 内容和规则 | 验证完整工作流执行 |
| **执行时间** | ~2 分钟 | 10-30 分钟 |
| **默认运行** | ✅ 是 | ❌ 需要 `--integration` |
| **验证方式** | 询问 Claude skill 的规则 | 创建真实项目并执行 |
| **Token 成本** | 低 | 高（多个 subagent） |
| **典型检查** | skill 是否可加载、关键规则是否存在 | 文件是否创建、测试是否通过、commit 历史 |

### 快速测试示例

快速测试通过 `run_claude` 询问 skill 的内容并验证：

```bash
output=$(run_claude "What does subagent-driven-development require?" 120)
assert_contains "$output" "spec compliance" "Spec review documented"
assert_contains "$output" "self-review" "Self-review requirement"
assert_order "$output" "spec compliance" "code quality" "Review order"
```

### 集成测试示例

集成测试创建真实项目、执行完整工作流并解析 session transcript：

```bash
# 1. 创建 Node.js 测试项目
TEST_PROJECT=$(create_test_project)
cd "$TEST_PROJECT" && npm init -y

# 2. 创建 implementation plan
create_test_plan "$TEST_PROJECT"

# 3. 执行 Claude + skill
timeout 1800 claude -p "Execute the plan" \
  --permission-mode bypassPermissions \
  --add-dir "$TEST_PROJECT"

# 4. 解析 session transcript 验证行为
grep -q '"name":"Skill".*"skill":"subagent-driven-development"' "$SESSION_FILE"
```

---

## 运行测试

### 基本命令

```bash
cd tests/claude-code

# 运行所有快速测试（推荐）
./run-skill-tests.sh

# 运行集成测试（慢，10-30 分钟）
./run-skill-tests.sh --integration

# 运行指定测试
./run-skill-tests.sh --test test-subagent-driven-development.sh

# 详细输出
./run-skill-tests.sh --verbose

# 自定义超时
./run-skill-tests.sh --timeout 1800
```

### 命令行参数

| 参数 | 缩写 | 说明 |
|------|------|------|
| `--verbose` | `-v` | 显示完整 Claude 输出 |
| `--test NAME` | `-t NAME` | 仅运行指定测试 |
| `--timeout SECONDS` | — | 设置每个测试的超时时间（默认 300 秒） |
| `--integration` | `-i` | 包含集成测试 |
| `--help` | `-h` | 显示帮助信息 |

### 运行前提

1. Claude Code CLI 已安装且在 PATH 中（`claude --version` 可执行）
2. 必须从 **superpowers 插件目录** 运行（不能从临时目录）
3. 本地开发市场已启用：`~/.claude/settings.json` 中 `"superpowers@superpowers-dev": true`

### 输出格式

```
========================================
 Claude Code Skills Test Suite
========================================

Repository: /path/to/superpowers
Test time: Mon Mar 24 10:00:00 UTC 2026
Claude version: 2.1.x

----------------------------------------
Running: test-subagent-driven-development.sh
----------------------------------------
  [PASS] (45s)

========================================
 Test Results Summary
========================================

  Passed:  1
  Failed:  0
  Skipped: 0

STATUS: PASSED
```

---

## Token 分析工具

`analyze-token-usage.py` 用于分析任何 Claude Code session 的 token 消耗：

```bash
python3 tests/claude-code/analyze-token-usage.py \
  ~/.claude/projects/<project-dir>/<session-id>.jsonl
```

### 查找 Session 文件

Session transcript 存储在 `~/.claude/projects/` 中，路径编码规则为工作目录路径中的 `/` 替换为 `-`：

```bash
# 示例路径
SESSION_DIR="$HOME/.claude/projects/-Users-jesse-Documents-GitHub-superpowers-superpowers"

# 查找最近的 session
ls -lt "$SESSION_DIR"/*.jsonl | head -5

# 查找最近 60 分钟内的 session
find ~/.claude/projects -name "*.jsonl" -mmin -60
```

### 输出解读

```
=========================================
 Token Usage Analysis
=========================================

Usage Breakdown:
------------------------------------
Agent           Description              Msgs    Input   Output    Cache    Cost
------------------------------------
main            Main session               34       27    3,996  1,213,703  $4.09
3380c209        implementing Task 1         1        2      787     24,989  $0.09
34b00fde        implementing Task 2         1        4      644     25,114  $0.09
...
------------------------------------

TOTALS:
  Total messages:         41
  Input tokens:           62
  Output tokens:          8,419
  Estimated cost: $4.67
```

### 关键指标含义

| 指标 | 正常范围 | 含义 |
|------|---------|------|
| 高 cache read | 期望值 | Prompt caching 工作正常 |
| main 的 input 高 | 正常 | coordinator 有完整上下文 |
| subagent 成本接近 | 正常 | 各任务复杂度相似 |
| 每个 subagent 成本 | $0.05–$0.15 | 典型单任务范围 |

---

## Session Transcript 格式

Session transcript 是 JSONL（JSON Lines）文件，每行是一个 JSON 对象。

### 消息结构

```json
{
  "type": "assistant",
  "message": {
    "content": [{"type": "text", "text": "..."}],
    "usage": {
      "input_tokens": 27,
      "output_tokens": 3996,
      "cache_read_input_tokens": 1213703
    }
  }
}
```

### 工具调用结果

```json
{
  "type": "user",
  "toolUseResult": {
    "agentId": "3380c209",
    "usage": {
      "input_tokens": 2,
      "output_tokens": 787,
      "cache_read_input_tokens": 24989
    },
    "prompt": "You are implementing Task 1...",
    "content": [{"type": "text", "text": "..."}]
  }
}
```

- `agentId`：关联到 subagent session
- `usage`：该 subagent 调用的 token 统计
- `prompt`：发送给 subagent 的完整提示

### 常用 grep 模式

```bash
# 检查 skill 是否被调用
grep -q '"name":"Skill".*"skill":"your-skill-name"' "$SESSION_FILE"

# 统计 subagent 派遣次数
grep -c '"name":"Task"' "$SESSION_FILE"

# 提取所有 skill 调用
grep -o '"skill":"[^"]*"' "$SESSION_FILE" | sort -u

# 检查 TodoWrite 使用
grep -c '"name":"TodoWrite"' "$SESSION_FILE"
```

---

## 故障排查

### Skill 未加载

**症状**：headless 测试中 skill 未找到

**解决方案**：
1. 确保从 superpowers 目录运行：`cd /path/to/superpowers && tests/...`
2. 检查 `~/.claude/settings.json` 中 `"superpowers@superpowers-dev": true`
3. 验证 skill 存在于 `skills/` 目录

### 权限错误

**症状**：Claude 被阻止写文件或访问目录

**解决方案**：
1. 使用 `--permission-mode bypassPermissions` 标志
2. 使用 `--add-dir /path/to/dir` 授予目录访问权限
3. 检查测试目录的文件权限

### 测试超时

**症状**：测试运行时间过长

**解决方案**：
1. 增加超时：`--timeout 1800`（30 分钟）
2. 检查 skill 逻辑中是否存在无限循环
3. 审查 subagent 任务的复杂度

### Session 文件未找到

**症状**：测试运行后找不到 session transcript

**解决方案**：
1. 检查正确的 `~/.claude/projects/` 路径
2. `find ~/.claude/projects -name "*.jsonl" -mmin -60`
3. 确认测试实际运行（检查测试输出中的错误）

---

## 编写新测试

### 标准模板

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

echo "=== Test: My Skill ==="

# --- 快速测试模式：询问 skill 内容 ---
output=$(run_claude "What does the my-skill skill do?" 30)

# 验证关键规则
assert_contains "$output" "expected behavior" "Skill describes behavior"
assert_contains "$output" "key requirement" "Key requirement documented"
assert_not_contains "$output" "deprecated pattern" "No deprecated patterns"

echo "=== All tests passed ==="
```

### 集成测试模板

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
source "$SCRIPT_DIR/test-helpers.sh"

# 1. 创建测试项目
TEST_PROJECT=$(create_test_project)
trap "cleanup_test_project $TEST_PROJECT" EXIT

# 2. 设置测试文件
cd "$TEST_PROJECT"
npm init -y
create_test_plan "$TEST_PROJECT"

# 3. 运行 Claude
PROMPT="Execute the implementation plan in docs/superpowers/plans/"
cd "$SCRIPT_DIR/../.." && timeout 1800 claude -p "$PROMPT" \
  --allowed-tools=all \
  --add-dir "$TEST_PROJECT" \
  --permission-mode bypassPermissions \
  2>&1 | tee output.txt

# 4. 查找 session transcript
WORKING_DIR_ESCAPED=$(echo "$SCRIPT_DIR/../.." | sed 's/\//-/g' | sed 's/^-//')
SESSION_DIR="$HOME/.claude/projects/$WORKING_DIR_ESCAPED"
SESSION_FILE=$(find "$SESSION_DIR" -name "*.jsonl" -type f -mmin -60 | sort -r | head -1)

# 5. 验证行为
if grep -q '"name":"Skill".*"skill":"your-skill-name"' "$SESSION_FILE"; then
    echo "[PASS] Skill was invoked"
fi

# 6. Token 分析
python3 "$SCRIPT_DIR/analyze-token-usage.py" "$SESSION_FILE"
```

### 最佳实践

1. **始终清理**：使用 `trap` 在退出时清理临时目录
2. **解析 transcript**：不要 grep 用户输出——解析 `.jsonl` session 文件
3. **授予权限**：使用 `--permission-mode bypassPermissions` 和 `--add-dir`
4. **从插件目录运行**：skill 只有在从 superpowers 目录运行时才会加载
5. **显示 token 使用**：始终包含 token 分析以了解成本
6. **测试真实行为**：验证实际创建的文件、通过的测试、生成的 commit

---

## 本章核心结论

- Superpowers 测试不是传统单元测试，而是基于 **headless Claude session** 的行为验证
- `test-helpers.sh` 提供 6 个核心 API：`run_claude`、`assert_contains`、`assert_not_contains`、`assert_count`、`assert_order`、`create_test_project`
- **快速测试**（~2 分钟）验证 skill 内容；**集成测试**（10-30 分钟）验证完整工作流
- Session transcript（JSONL 格式）是所有验证的数据来源
- `analyze-token-usage.py` 帮助追踪和优化每次测试的 token 成本
- 测试必须从 superpowers 插件目录运行，否则 skill 无法加载
