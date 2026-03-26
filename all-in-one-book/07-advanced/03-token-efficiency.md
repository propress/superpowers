# 第七章·第三节 Token 效率

## 概述

Context Window(上下文窗口) 是公共资源。你的 Skill 与 system prompt、对话历史、其他 Skill 的 metadata 以及用户请求共享同一个 context window。每一个 token 都有成本——不是经济成本，而是注意力成本。本节介绍如何在保持 Skill 有效性的同时最大限度减少 token 消耗。

---

## Context Window 是公共资源

启动时，所有 Skill 的 metadata（name + description）被预加载。Claude 只在 Skill 变得相关时才读取 SKILL.md，只在需要时才读取额外文件。但是，**一旦 SKILL.md 被加载，每个 token 都与对话历史和其他上下文竞争**。

**默认假设**：Claude 已经非常聪明。只添加 Claude 不具备的上下文。

质疑每一段信息：
- "Claude 真的需要这个解释吗？"
- "我能假设 Claude 知道这个吗？"
- "这段话值得它的 token 成本吗？"

---

## Token 预算指南

| Skill 类别 | 目标字数 | 原因 |
|-----------|---------|------|
| getting-started workflow | < 150 words | 每次会话加载 |
| 频繁加载的 Skill | < 200 words | 高频使用，累积成本大 |
| 其他 Skill | < 500 words | 按需加载但仍需简洁 |

### 验证方法

```bash
wc -w skills/path/SKILL.md
# getting-started: 目标 < 150
# 频繁加载: 目标 < 200
# 其他: 目标 < 500
```

SKILL.md body 保持 **500 行以内**以获得最佳性能。超过时拆分到独立文件。

---

## Degrees of Freedom 模型

匹配具体度与任务的脆弱性和可变性。

### High Freedom（高自由度）

文本指令，多种方法都有效。

```markdown
## Code review process
1. Analyze the code structure
2. Check for potential bugs
3. Suggest improvements
4. Verify adherence to conventions
```

适用：决策依赖上下文、启发式方法引导。

### Medium Freedom（中自由度）

伪代码或带参数的脚本。

```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data, generate output, optionally include visualizations
```

适用：存在推荐模式但允许变化。

### Low Freedom（低自由度）

精确脚本，几乎没有参数。

```bash
python scripts/migrate.py --verify --backup
# Do not modify the command or add additional flags.
```

适用：操作脆弱易出错、一致性至关重要。

---

## Bridge vs Open Field 类比

把 Claude 想象成探索路径的机器人：

- **窄桥（两侧悬崖）**：只有一条安全路径。提供精确护栏和详细指令（低自由度）。例：数据库迁移。
- **开阔草地（无危险）**：多条路径通往成功。给出大方向，信任 Claude 找到最佳路线（高自由度）。例：代码审查。

---

## Good vs Bad Examples

### 简洁版 (~50 tokens) ✅

````markdown
## Extract PDF text
Use pdfplumber for text extraction:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

### 冗余版 (~150 tokens) ❌

```markdown
## Extract PDF text
PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip...
```

简洁版假设 Claude 知道 PDF 和库的概念——它确实知道。

---

## 多模型兼容

Skill 效果取决于底层模型。测试时需覆盖所有目标模型：

| 模型 | 特点 | 考量 |
|------|------|------|
| Claude Haiku | 快速、经济 | Skill 是否提供了足够的指导？ |
| Claude Sonnet | 平衡 | Skill 是否清晰高效？ |
| Claude Opus | 强推理 | Skill 是否避免了过度解释？ |

对 Opus 完美的 Skill 可能需要为 Haiku 增加更多细节。跨模型使用时，瞄准对所有模型都有效的指令。

---

## 压缩技巧

### 1. 细节移到 tool help

```bash
# ❌ 在 SKILL.md 中列出所有 flag
search-conversations supports --text, --both, --after DATE, --before DATE, --limit N

# ✅ 引用 --help
search-conversations supports multiple modes and filters. Run --help for details.
```

### 2. 使用交叉引用

```markdown
# ❌ 重复工作流细节
When searching, dispatch subagent with template...
[20 lines of repeated instructions]

# ✅ 引用其他 skill
Always use subagents (50-100x context savings). REQUIRED: Use [other-skill-name] for workflow.
```

### 3. 压缩示例

```markdown
# ❌ 42 words
your human partner: "How did we handle authentication errors in React Router before?"
You: I'll search past conversations for React Router authentication patterns.
[Dispatch subagent with search query: "React Router authentication error handling 401"]

# ✅ 20 words
Partner: "How did we handle auth errors in React Router?"
You: Searching...
[Dispatch subagent → synthesis]
```

### 4. 消除冗余

- 不重复交叉引用 skill 中已有的内容
- 不解释命令本身已明确的东西
- 不为同一模式提供多个示例

---

## 本章核心结论

- Context window 是公共资源——每个 token 都与对话历史竞争
- 严格遵守 token 预算：getting-started < 150 words，频繁加载 < 200，其他 < 500
- 使用 Degrees of Freedom 模型匹配指令具体度与任务脆弱性
- "Bridge vs Open Field"：脆弱操作用精确指令，灵活操作用大方向
- 压缩技巧：引用 `--help`、交叉引用 skill、压缩示例、消除冗余
- 跨模型测试（Haiku/Sonnet/Opus）确保 Skill 在不同能力水平下都有效
