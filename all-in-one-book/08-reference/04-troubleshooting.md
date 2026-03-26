# 故障排查

> 本章以 FAQ 格式列出 Superpowers 使用中的常见问题、症状、解决方案和预防措施。

---

## 问题 1：Skill 不加载

**症状**：Agent 不遵循 Skill 流程，直接写代码或跳过步骤。

**可能原因**：

| 原因 | 诊断方法 | 解决方案 |
|------|---------|---------|
| 插件未安装 | 询问 Agent "Tell me about your superpowers" | 按平台安装指南重新安装 |
| Hook 未触发 | 检查会话开始时是否有 Skill 上下文注入 | 验证 hooks.json / hooks-cursor.json 配置 |
| Skill 目录路径错误 | 检查 plugin.json 中的 `skills` 路径 | 确保路径指向正确的 `skills/` 目录 |
| description 字段不匹配 | 检查 SKILL.md frontmatter | 确保 `description` 包含触发关键词 |
| 会话已过长 | 上下文窗口已满 | 开始新会话或使用 `compact` |

**预防**：安装后第一件事运行验证："Tell me about your superpowers"。

---

## 问题 2：Hook 不触发

**症状**：会话启动后没有 Superpowers 上下文，Agent 表现为普通 Agent。

**Claude Code 排查**：

```bash
# 检查 hooks.json 是否存在
ls -la .claude-plugin/
cat hooks/hooks.json

# 手动运行 hook 脚本
./hooks/session-start
```

**Cursor 排查**：

```bash
# 检查 hooks-cursor.json
cat hooks/hooks-cursor.json

# 检查 plugin.json 中的 hooks 路径
grep "hooks" .cursor-plugin/plugin.json
```

**常见修复**：

1. **权限问题**：`chmod +x hooks/session-start`
2. **Shell 兼容性**：确保使用 `#!/usr/bin/env bash` 而非 `#!/bin/bash`
3. **路径空格**：确保 hooks.json 中使用转义双引号包围路径
4. **Bash 5.3+ 挂起**：`session-start` 已使用 `printf` 替代 heredoc，如果使用旧版本需手动更新

---

## 问题 3：Brainstorm Server 启动失败

**症状**：Visual Companion 无法打开，或服务器立即退出。

| 症状 | 原因 | 解决方案 |
|------|------|---------|
| `require() is not defined` | Node.js 22+ ESM 模式 | 确保服务器文件名为 `server.cjs`（非 `.js`） |
| 60 秒后退出 | Owner-PID 监控误判 | v5.0.6 已修复；更新到最新版本 |
| Windows 上无法启动 | `nohup`/`disown` 在 Git Bash 中失败 | 使用前台模式：`node server.cjs --foreground` |
| 端口被占用 | 旧服务器未正常退出 | `./scripts/stop-server.sh` 或手动 `kill` |

**手动启动排查**：

```bash
cd skills/brainstorming/scripts
node server.cjs --foreground --port 3000
# 观察控制台输出
```

---

## 问题 4：测试失败

**症状**：运行 `run-skill-tests.sh` 时某些测试不通过。

**排查步骤**：

1. **确认运行位置**：必须从 superpowers 插件目录运行
2. **确认 Claude Code CLI**：`claude --version`
3. **确认本地 dev marketplace**：检查 `~/.claude/settings.json` 中是否有 `"superpowers@superpowers-dev": true`
4. **使用 verbose 模式**：`./run-skill-tests.sh --verbose`
5. **增加超时**：`./run-skill-tests.sh --timeout 1800`

**Token 分析**：

```bash
python3 analyze-token-usage.py ~/.claude/projects/<session>.jsonl
```

如果 Token 消耗异常高，可能是 Skill 加载了过多上下文或进入了无限循环。

---

## 问题 5：Token 预算超限

**症状**：Agent 响应变慢、截断或丢失上下文。

**诊断**：

- 检查 SKILL.md 行数：主文件应 < 500 行
- 检查 reference 文件是否嵌套过深（应只有一层）
- 检查是否有多个 Skill 同时加载

**解决方案**：

1. **压缩 SKILL.md**：删除 Agent 已知的信息
2. **使用 Progressive Disclosure**：将详细内容移到 reference 文件
3. **减少 description 长度**：max 1024 chars
4. **遵循 Token 预算**：
   - getting-started 类型 < 150 words
   - frequently-loaded 类型 < 200 words
   - 其他 < 500 words

---

## 问题 6：Skill 未被正确触发

**症状**：Agent 应该使用某个 Skill 但没有使用。

**排查**：

1. **检查 description 字段**：是否包含用户输入中的关键词？
2. **CSO 检查**：description 是否在描述"何时使用"而非"如何工作"？
3. **关键词覆盖**：description 是否涵盖同义词和常见错误消息？
4. **Skill 冲突**：是否有多个 Skill 的 description 重叠？

**快速修复**：手动告诉 Agent 使用特定 Skill："请使用 brainstorming skill"。

---

## 问题 7：Git Worktree 问题

**症状**：worktree 创建失败或目录未被 `.gitignore` 忽略。

| 问题 | 解决方案 |
|------|---------|
| `fatal: is already checked out` | 删除旧 worktree：`git worktree remove <path>` |
| worktree 目录被提交 | 添加到 `.gitignore` 并提交 |
| 测试在 worktree 中失败 | 重新运行项目设置（npm install 等） |
| 找不到 `.worktrees/` | 使用 `git worktree list` 查看所有 worktree |

---

## 问题 8：跨平台兼容性问题

### Windows 特有问题

| 问题 | 解决方案 |
|------|---------|
| 单引号路径失败 | hooks.json 使用转义双引号 |
| `#!/bin/bash` 找不到 | 使用 `#!/usr/bin/env bash` |
| PID 监控失效 | v5.0.6 已修复，更新版本 |
| Symlink 需要管理员权限 | 使用 junction：`mklink /J` |

### macOS 特有问题

| 问题 | 解决方案 |
|------|---------|
| Homebrew Bash 5.3+ heredoc 挂起 | 已使用 `printf` 替代 |
| `/bin/bash` 版本过旧 | 使用 `#!/usr/bin/env bash` 调用 Homebrew 版本 |

### Linux 特有问题

| 问题 | 解决方案 |
|------|---------|
| `/bin/sh` 是 dash 不是 bash | 已使用 POSIX 兼容语法 |
| Tailscale SSH 的 EPERM | v5.0.6 已修复 Owner-PID 处理 |

---

## 问题 9：版本升级后行为变化

**v4 → v5 常见变化**：

| 变化 | 影响 | 应对 |
|------|------|------|
| Spec/Plan 目录变更 | 旧路径不再使用 | 迁移到 `docs/superpowers/specs/` 和 `docs/superpowers/plans/` |
| Commands 废弃 | `/brainstorm` 等不再工作 | 直接使用对应 Skill |
| SDD 推荐但非强制 | 行为更灵活 | 根据平台能力选择 |

---

## 通用排查清单

当遇到任何 Superpowers 相关问题时：

- [ ] 确认 Superpowers 版本：检查 plugin.json 中的 `version`
- [ ] 确认平台版本：Claude Code / Cursor / Codex / OpenCode / Gemini CLI
- [ ] 确认 Hook 触发：会话开始时是否看到 Skill 上下文
- [ ] 确认 Skill 可访问：Agent 能否描述 Superpowers
- [ ] 检查控制台输出：是否有错误信息
- [ ] 尝试新会话：排除上下文污染
- [ ] 更新到最新版本：`git pull` 获取最新修复

---

## 本章核心结论

1. **首要检查**：90% 的问题通过"确认安装 → 确认 Hook → 确认 Skill 加载"三步排查即可定位。
2. **平台差异**：不同平台的配置格式和路径约定不同，排查时先确认平台。
3. **版本更新**：许多已知 bug 在新版本中已修复，遇到问题先检查是否有更新。
