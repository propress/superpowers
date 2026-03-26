# 术语表

> 本表收录 Superpowers 框架中出现的所有核心术语，按英文字母排序。

## 前置阅读

本表可在任何阶段查阅，无需前置知识。

---

## 完整术语表

| 英文术语 | 中文翻译 | 定义 | 首次出现章节 |
|----------|----------|------|-------------|
| Agent | 智能体 | 执行编码任务的 AI（如 Claude、Gemini、Codex），Superpowers 的操作主体 | 00-overview/01 |
| Anthropic Best Practices | Anthropic 官方最佳实践 | Anthropic 发布的 Skill 编写指南，涵盖简洁性、自由度、测试策略 | 02-skills/13 |
| Authority | 权威原则 | Cialdini 说服原则之一：使用权威语言（"YOU MUST"）提高 Agent 合规率 | 02-skills/13 |
| Baseline Test | 基线测试 | 在没有 Skill 的情况下运行场景，记录 Agent 的失败行为作为参照 | 06-testing/02 |
| Brainstorming | 头脑风暴 | 将用户模糊想法转化为经过验证的设计文档（Spec）的结构化对话流程 | 02-skills/01 |
| Bulletproof | 防弹 | Skill 在最大压力下仍能让 Agent 正确遵守规则的状态 | 06-testing/02 |
| Checkpoint | 检查点 | 执行计划中标记阶段完成的节点，用于进度追踪和回滚 | 02-skills/03 |
| Claude Code | Claude Code | Anthropic 的 AI 编程工具，Superpowers 的主要目标平台 | 05-platform/02 |
| Claude Search Optimization (CSO) | Claude 搜索优化 | Skill 的 `description` 字段只写触发条件不写流程摘要的策略，防止 Agent 走捷径 | 07-advanced/02 |
| Code Quality Reviewer | 代码质量审查者 | Subagent-Driven Development 中双重审查的第二阶段，审查实现质量 | 02-skills/04 |
| Codex | Codex | OpenAI 的编程 Agent 平台，通过 symlink 安装 Superpowers | 05-platform/04 |
| Command | 命令 | 用户可手动触发的操作入口（如 `/brainstorm`），v5.0 已废弃 | 04-commands/01 |
| Commitment | 承诺原则 | Cialdini 说服原则之一：要求 Agent 公开声明后续行动以保持一致性 | 02-skills/13 |
| Compact | 压缩 | Claude Code 的上下文压缩事件，触发 Hook 重新注入 Skill 上下文 | 05-platform/01 |
| Context Isolation | 上下文隔离 | 每个 Subagent 只接收执行任务所需的最小上下文，防止上下文窗口污染 | 01-core-workflow/01 |
| Context Window | 上下文窗口 | Agent 可处理的文本总量限制，是共享的公共资源 | 07-advanced/03 |
| Controller | 调度者 | Subagent-Driven Development 中的核心角色，负责读取计划、分派任务、审查结果 | 02-skills/04 |
| Cursor | Cursor | AI 编程 IDE，通过 `.cursor-plugin/` 集成 Superpowers | 05-platform/03 |
| Defense in Depth | 纵深防御 | 多层错误检测策略：状态机层 → 集成层 → 端到端层 | 02-skills/06 |
| Degrees of Freedom | 自由度 | Skill 给予 Agent 的灵活程度（高/中/低），取决于任务的脆弱性和变异性 | 07-advanced/03 |
| Description | 描述字段 | SKILL.md YAML frontmatter 中的必填字段，决定 Skill 何时被发现和加载 | 07-advanced/02 |
| Dispatching Parallel Agents | 并行 Agent 调度 | 将多个独立问题分配给不同 Agent 同时解决的模式 | 02-skills/10 |
| Evidence Before Claims | 证据先于声明 | Superpowers 设计原则：任何"完成"声明必须附带刚运行的验证输出 | 01-core-workflow/01 |
| Executing Plans | 执行计划 | 按计划逐步实施代码变更的 Skill，适用于不支持 Subagent 的平台 | 02-skills/03 |
| Finishing Branch | 完成开发分支 | 在实现完成后选择 merge/PR/keep/discard 的分支收尾流程 | 02-skills/12 |
| Frontmatter | 前置元数据 | SKILL.md 文件头部的 YAML 块，包含 `name` 和 `description` 字段 | 07-advanced/01 |
| Gemini CLI | Gemini CLI | Google 的命令行 AI 工具，通过 `gemini-extension.json` 集成 | 05-platform/06 |
| Git Worktree | Git 工作树 | Git 功能，允许在独立目录中创建分支的工作副本，共享同一仓库 | 02-skills/11 |
| Hard Gate | 硬性门禁 | 流程中不可跳过的强制检查点，未通过则终止流程 | 02-skills/01 |
| Hook | 钩子 | 在会话生命周期事件时自动执行的脚本，用于注入 Skill 上下文 | 05-platform/01 |
| Idle Timeout | 空闲超时 | Brainstorm Server 30 分钟无客户端连接后自动退出的安全机制 | 02-skills/01 |
| Implementer | 执行者 | Subagent-Driven Development 中负责写代码、运行测试、提交的角色 | 02-skills/04 |
| Iron Law | 铁律 | 不可违反的工程规则，Superpowers 有三条：TDD、Debugging、Verification | 01-core-workflow/01 |
| JSONL | JSON Lines | Claude Code 会话记录格式，每行一个 JSON 对象，用于 Token 分析 | 06-testing/01 |
| Matcher | 匹配器 | hooks.json 中的正则表达式，决定 Hook 在哪些事件类型上触发 | 05-platform/01 |
| Meta-Testing | 元测试 | Agent 选错答案后追问"Skill 怎么写才能让你选对？"的反馈技术 | 06-testing/02 |
| OpenCode | OpenCode | 开源 AI 编程工具，通过 JS 插件集成 Superpowers | 05-platform/05 |
| Owner-PID | 属主进程 ID | Brainstorm Server 监控的父进程 PID，父进程死亡时 Server 自动退出 | 02-skills/01 |
| Parallel Agents | 并行 Agent | 同时执行独立任务的多个 Agent 实例 | 02-skills/10 |
| Plan Mode | 计划模式 | Agent 进入结构化计划编写状态的触发条件 | 02-skills/02 |
| Plugin | 插件 | Superpowers 在各平台上的安装载体（.claude-plugin/, .cursor-plugin/ 等） | 05-platform/02 |
| Pressure Scenario | 压力场景 | 模拟时间、沉没成本、权威等压力的测试场景，用于验证 Skill 的防弹程度 | 06-testing/02 |
| Process Before Shortcuts | 流程先于捷径 | 设计原则：即使任务简单，也必须触发对应 Skill | 01-core-workflow/01 |
| Progressive Disclosure | 渐进披露 | SKILL.md 作为概览指向详细材料的信息组织模式 | 07-advanced/01 |
| Prompt Caching | 提示缓存 | 重复使用的系统提示被缓存以减少 Token 消耗的机制 | 07-advanced/03 |
| Rationalization | 合理化 | Agent 为跳过 Skill 而自我辩解的思维模式（如"这只是个简单问题"） | 02-skills/00 |
| Receiving Code Review | 接收代码评审 | 处理和应对他人代码审查反馈的 Skill | 02-skills/09 |
| Red Flag | 红旗 | Skill 中标记的危险行为模式，出现即应立即停止 | 02-skills/05 |
| Red-Green-Refactor | 红-绿-重构 | TDD 的三阶段循环：Red（测试失败）→ Green（测试通过）→ Refactor（优化） | 02-skills/05 |
| Requesting Code Review | 请求代码评审 | 在实现完成后发起代码审查的 Skill | 02-skills/08 |
| Root Cause | 根因 | 问题的真正原因，而非表面症状。Debugging Iron Law 要求先找到根因再修复 | 02-skills/06 |
| Scarcity | 稀缺原则 | Cialdini 说服原则之一：通过时间限制创造紧迫感（"Before proceeding"） | 02-skills/13 |
| SessionStart | 会话启动 | Hook 触发事件：新会话开始、清除上下文、压缩上下文时触发 | 05-platform/01 |
| Skill | 技能 | 定义 Agent 在特定场景下必须遵循的流程文档（SKILL.md），Superpowers 的基本单元 | 00-overview/01 |
| Social Proof | 社会证明 | Cialdini 说服原则之一：通过普遍模式建立规范（"Every time"） | 02-skills/13 |
| Spec | 设计文档 | Brainstorming 的输出物，描述要构建什么及如何构建，保存在 `docs/superpowers/specs/` | 02-skills/01 |
| Spec Reviewer | 规格审查者 | 双重审查第一阶段，验证代码是否严格符合 Spec 要求（不多不少） | 02-skills/04 |
| Subagent | 子智能体 | 由 Controller 分派的、具有独立上下文窗口的 Agent 实例 | 02-skills/04 |
| Subagent-Driven Development (SDD) | 子智能体驱动开发 | 通过 Controller/Implementer/Reviewer 架构自主执行计划的开发模式 | 02-skills/04 |
| Systematic Debugging | 系统化调试 | 四阶段调试法：Reproduce → Hypothesize → Verify → Fix | 02-skills/06 |
| TDD | 测试驱动开发 | Test-Driven Development：先写失败测试 → 写最小实现 → 重构 | 02-skills/05 |
| TodoWrite | 任务追踪工具 | Claude Code 的待办事项工具，用于追踪 Checklist 进度 | 02-skills/03 |
| Token | 令牌 | LLM 处理文本的基本单位，Context Window 的计量单位 | 07-advanced/03 |
| Unity | 统一原则 | Cialdini 说服原则之一：通过共享身份建立合作（"我们是同事"） | 02-skills/13 |
| Verification | 验证 | 在声明完成前重新运行所有测试的强制步骤 | 02-skills/07 |
| Visual Companion | 可视化伴侣 | Brainstorming 过程中的浏览器辅助工具，展示 mockup、图表和选项 | 02-skills/01 |
| Worktree | 工作树 | Git Worktree 的简称，用于创建隔离的开发环境 | 02-skills/11 |
| Writing Plans | 编写计划 | 将设计文档分解为 2-5 分钟粒度任务的 Skill | 02-skills/02 |
| Writing Skills | 编写技能 | 以 TDD 方法论创建新 Skill 的元技能 | 02-skills/13 |
| YAGNI | 你不会需要它 | You Aren't Gonna Need It：只实现 Spec 要求的功能，防止过度实现 | 01-core-workflow/01 |

---

## 本章核心结论

1. **术语一致性**：在所有文档和讨论中使用相同的英文术语，首次出现时标注中文翻译，之后统一使用英文。
2. **查阅方式**：遇到不熟悉的缩写或术语时，先查本表确认准确含义，再阅读对应章节了解完整上下文。
3. **扩展规则**：项目中新增概念时，应同步更新本表，确保本表始终是术语的单一事实来源。
