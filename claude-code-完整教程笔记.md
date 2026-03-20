# Claude Code 完整教程笔记

> 整理自 Anthropic 官方文档（code.claude.com/docs），整理时间：2026-03-20

---

## 目录

1. [什么是 Claude Code](#1-什么是-claude-code)
2. [安装与登录](#2-安装与登录)
3. [快速上手：第一个会话](#3-快速上手第一个会话)
4. [常用工作流](#4-常用工作流)
5. [Plan Mode：先规划再执行](#5-plan-mode先规划再执行)
6. [CLAUDE.md：持久化记忆系统](#6-claudemd持久化记忆系统)
7. [Subagents：专属子智能体](#7-subagents专属子智能体)
8. [Hooks：自动化钩子系统](#8-hooks自动化钩子系统)
9. [GitHub Actions 集成](#9-github-actions-集成)
10. [最佳实践与使用技巧](#10-最佳实践与使用技巧)
11. [常用命令速查表](#11-常用命令速查表)

---

## 1. 什么是 Claude Code

Claude Code 是 Anthropic 推出的 **AI 驱动的编程助手**，核心特点：

- **理解整个代码库**：不只是单文件，能跨多个文件分析和修改
- **真正执行操作**：读文件、写代码、运行命令、提交 Git，不只是"建议"
- **多端支持**：终端 CLI、VS Code 插件、JetBrains 插件、桌面 App、Web 浏览器
- **CI/CD 集成**：支持 GitHub Actions、GitLab CI/CD

### 能做什么？

| 场景 | 示例 |
|------|------|
| 自动化繁琐任务 | 写测试、修 lint 错误、解决 merge conflict、更新依赖 |
| 构建功能 / 修 Bug | 用自然语言描述需求，Claude 规划并实现 |
| Git 操作 | 暂存变更、写 commit message、创建 PR |
| 代码重构 | 将旧代码升级为现代写法 |
| 代码审查 | 分析变更并给出改进建议 |

---

## 2. 安装与登录

### 安装方式

**macOS / Linux / WSL（推荐）：**
```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell：**
```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD：**
```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

> ⚠️ Windows 需要先安装 [Git for Windows](https://git-scm.com/downloads/win)

**Homebrew（macOS）：**
```bash
brew install --cask claude-code
# 不会自动更新，需手动：brew upgrade claude-code
```

**WinGet（Windows）：**
```powershell
winget install Anthropic.ClaudeCode
```

### 账号要求

需要以下任一账号：
- **Claude Pro / Max / Teams / Enterprise**（推荐）
- **Claude Console**（API 访问，按量付费）
- **Amazon Bedrock / Google Vertex AI / Microsoft Foundry**（企业云）

### 登录

```bash
claude
# 首次启动会提示登录，按提示操作即可
# 登录后凭证会保存，无需每次重新登录

/login   # 切换账号时使用
```

---

## 3. 快速上手：第一个会话

### 启动

```bash
cd /path/to/your/project
claude
```

### 了解代码库

```text
what does this project do?
what technologies does this project use?
where is the main entry point?
explain the folder structure
```

### 第一次代码修改

```text
add a hello world function to the main file
```

Claude 会：
1. 找到合适的文件
2. 展示建议的修改
3. 请求你的确认
4. 执行修改

> 💡 Claude 在修改文件前**始终会请求确认**。可以逐个批准，也可以开启"Accept all"模式。

### Git 操作

```text
what files have I changed?
commit my changes with a descriptive message
create a new branch called feature/quickstart
show me the last 5 commits
help me resolve merge conflicts
```

### 修 Bug / 加功能

```text
add input validation to the user registration form
there's a bug where users can submit empty forms - fix it
```

### 其他常见操作

```text
# 重构
refactor the authentication module to use async/await instead of callbacks

# 写测试
write unit tests for the calculator functions

# 更新文档
update the README with installation instructions

# 代码审查
review my changes and suggest improvements
```

---

## 4. 常用工作流

### 4.1 理解新代码库

**快速概览：**
```text
give me an overview of this codebase
explain the main architecture patterns used here
what are the key data models?
how is authentication handled?
```

**定位相关代码：**
```text
find the files that handle user authentication
how do these authentication files work together?
trace the login process from front-end to database
```

> 💡 技巧：先问宏观问题，再深入具体模块；使用项目领域语言提问

### 4.2 高效修 Bug

```text
# 1. 描述错误
I'm seeing an error when I run npm test

# 2. 请求修复建议
suggest a few ways to fix the @ts-ignore in user.ts

# 3. 应用修复
update user.ts to add the null check you suggested
```

> 💡 技巧：提供复现步骤；告知错误是偶发还是必现；粘贴完整错误信息

### 4.3 代码重构

```text
# 1. 找到需要重构的代码
find deprecated API usage in our codebase

# 2. 获取重构建议
suggest how to refactor utils.js to use modern JavaScript features

# 3. 安全地应用变更
refactor utils.js to use ES2024 features while maintaining the same behavior

# 4. 验证重构结果
run tests for the refactored code
```

> 💡 技巧：小步重构，每步都可测试；要求保持向后兼容

---

## 5. Plan Mode：先规划再执行

Plan Mode 让 Claude 在**只读模式**下分析代码库，制定计划后再执行，避免"解决了错误问题"。

### 何时使用 Plan Mode

- 需要修改多个文件的功能
- 不熟悉要修改的代码
- 想在执行前审查方案
- 复杂的架构变更

> 💡 简单任务（改个变量名、加一行日志）直接做，不需要 Plan Mode

### 如何开启

**会话中切换：**
- `Shift+Tab` → 第一次切换到 Auto-Accept Mode（`⏵⏵ accept edits on`）
- 再次 `Shift+Tab` → 切换到 Plan Mode（`⏸ plan mode on`）

**启动时直接进入 Plan Mode：**
```bash
claude --permission-mode plan
```

### 推荐四步工作流

```
Step 1 - Explore（Plan Mode）
  read /src/auth and understand how we handle sessions and login.
  also look at how we manage environment variables for secrets.

Step 2 - Plan（Plan Mode）
  I want to add Google OAuth. What files need to change?
  What's the session flow? Create a plan.
  # 按 Ctrl+G 在编辑器中直接编辑计划

Step 3 - Implement（Normal Mode）
  implement the OAuth flow from your plan. write tests for the
  callback handler, run the test suite and fix any failures.

Step 4 - Commit（Normal Mode）
  commit with a descriptive message and open a PR
```

---

## 6. CLAUDE.md：持久化记忆系统

每次 Claude Code 会话都从空白上下文开始。CLAUDE.md 是解决这个问题的核心机制。

### 两种记忆机制对比

| | CLAUDE.md 文件 | Auto Memory（自动记忆）|
|---|---|---|
| **谁写** | 你 | Claude 自动写 |
| **内容** | 指令和规则 | 学到的模式和偏好 |
| **范围** | 项目 / 用户 / 组织 | 每个工作目录 |
| **加载时机** | 每次会话开始 | 每次会话开始（前200行）|
| **适合写** | 编码规范、工作流、架构决策 | 构建命令、调试经验、Claude 发现的偏好 |

### CLAUDE.md 文件位置

| 范围 | 位置 | 用途 |
|------|------|------|
| **组织级**（IT 管理） | Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | 公司编码规范、安全策略 |
| **项目级**（团队共享） | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 项目架构、编码标准、常用工作流 |
| **用户级**（个人偏好） | `~/.claude/CLAUDE.md` | 个人代码风格、工具快捷方式 |

### 快速生成 CLAUDE.md

```text
/init
```

Claude 会分析你的代码库，自动生成包含构建命令、测试指令和项目约定的 CLAUDE.md。

### 写好 CLAUDE.md 的原则

1. **控制长度**：每个文件目标 200 行以内，太长会降低遵循率
2. **结构清晰**：用 Markdown 标题和列表组织，便于 Claude 扫描
3. **具体明确**：写具体可操作的指令，而非模糊描述
4. **按规则拆分**：大项目用 `.claude/rules/` 目录按文件类型分类规则

### CLAUDE.md 示例结构

```markdown
# 项目概述
这是一个 TypeScript + React 的 SaaS 应用。

## 构建与测试
- 构建：`npm run build`
- 测试：`npm test`
- Lint：`npm run lint`

## 编码规范
- 使用 async/await，不用 callback
- 组件用函数式写法
- 所有新功能必须有单元测试

## 架构说明
- API 层在 `/src/api`
- 状态管理用 Zustand
- 样式用 Tailwind CSS
```

---

## 7. Subagents：专属子智能体

Subagents 是专门处理特定任务的 AI 助手，每个都有独立的上下文窗口、自定义 system prompt 和工具权限。

### 内置 Subagents

| Agent | 模型 | 用途 |
|-------|------|------|
| **Explore** | Haiku（快速） | 只读搜索和分析代码库 |
| **Plan** | 继承主会话 | Plan Mode 下的代码库研究 |
| **General-purpose** | 继承主会话 | 复杂多步骤任务（探索+修改）|
| **Bash** | 继承主会话 | 在独立上下文中运行终端命令 |

### 为什么用 Subagents？

- **保护主上下文**：探索和实现分开，避免主会话 context 被撑满
- **强制约束**：限制 subagent 只能用特定工具
- **跨项目复用**：用户级 subagent 在所有项目中可用
- **控制成本**：把简单任务路由到更便宜的模型（如 Haiku）

### 查看和使用 Subagents

```text
/agents
# 查看所有可用 subagents，也可以在这里创建新的

# 自动委派
review my recent code changes for security issues
run all tests and fix any failures

# 明确指定
use the code-reviewer subagent to check the auth module
have the debugger subagent investigate why users can't log in
```

### 创建自定义 Subagent

Subagent 是带 YAML frontmatter 的 Markdown 文件：

```markdown
---
name: code-reviewer
description: Reviews code changes for security issues, performance problems, and style violations
model: claude-haiku-4-5
tools:
  - Read
  - Bash
---

You are a code reviewer focused on security and performance.
When reviewing code:
1. Check for SQL injection, XSS, and other security vulnerabilities
2. Identify performance bottlenecks
3. Verify error handling is complete
4. Ensure tests cover edge cases
```

**存放位置：**
- 项目级：`.claude/agents/your-agent.md`（团队共享）
- 用户级：`~/.claude/agents/your-agent.md`（个人使用）

---

## 8. Hooks：自动化钩子系统

Hooks 是在 Claude Code 生命周期特定节点自动执行的脚本，可以是 Shell 命令、HTTP 端点或 LLM Prompt。

### Hook 事件列表

| 事件 | 触发时机 |
|------|---------|
| `SessionStart` | 会话开始或恢复时 |
| `UserPromptSubmit` | 提交 prompt 前 |
| `PreToolUse` | 工具调用前（可阻止） |
| `PostToolUse` | 工具调用成功后 |
| `Stop` | Claude 完成响应时 |
| `SessionEnd` | 会话结束时 |
| `PreCompact` | 上下文压缩前 |
| `PostCompact` | 上下文压缩后 |

### 示例：阻止危险命令

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# .claude/hooks/block-rm.sh
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Destructive rm -rf command blocked"
    }
  }'
fi
```

---

## 9. GitHub Actions 集成

在 PR 或 Issue 中 `@claude` 一下，Claude 就能自动分析代码、创建 PR、实现功能、修复 Bug。

### 快速安装

在终端中运行：
```text
/install-github-app
```

### 手动安装步骤

1. 安装 Claude GitHub App：https://github.com/apps/claude
   - 需要权限：Contents（读写）、Issues（读写）、Pull requests（读写）
2. 在仓库 Secrets 中添加 `ANTHROPIC_API_KEY`
3. 复制 workflow 文件到 `.github/workflows/`

### 基础 Workflow 配置

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # 响应评论中的 @claude 提及
```

### 使用方式

在任意 PR 或 Issue 评论中：
```
@claude 帮我审查这个 PR 的安全问题
@claude 实现 issue #42 描述的功能
@claude 修复这个测试失败
```

---

## 10. 最佳实践与使用技巧

### 核心原则：管理好 Context Window

> Claude 的 context window 会快速填满，填满后性能会下降。这是最重要的资源。

**策略：**
- 用 `/compact` 压缩上下文
- 复杂任务拆分成多个会话
- 用 Subagents 把探索和实现分开
- 安装自定义 status line 实时监控 context 用量

### 技巧 1：给 Claude 验证自己工作的方法

这是**最高杠杆**的单一操作。

| 策略 | 不好的做法 | 好的做法 |
|------|-----------|---------|
| 提供验证标准 | "实现一个验证邮箱的函数" | "写 validateEmail 函数，测试用例：user@example.com 返回 true，invalid 返回 false。实现后运行测试" |
| 验证 UI 变更 | "让 dashboard 好看一点" | "[粘贴截图] 实现这个设计，截图对比结果，列出差异并修复" |
| 解决根本原因 | "构建失败了" | "构建报这个错误：[粘贴错误]。修复它并验证构建成功，解决根本原因，不要只是屏蔽错误" |

### 技巧 2：提供具体上下文

```text
# 不好
fix the bug in the login

# 好
fix the bug in /src/auth/login.ts where users with special characters
in their email can't log in. The error is "Invalid email format" but
the email is valid. Follow the same validation pattern used in /src/auth/register.ts
```

### 技巧 3：探索 → 规划 → 实现 → 提交

不要让 Claude 直接跳到写代码。先用 Plan Mode 探索和规划，再切回 Normal Mode 实现。

### 技巧 4：像对同事一样说话

```text
# 不好（太模糊）
make the code better

# 好（具体目标）
refactor the UserService class to reduce duplication between
createUser and updateUser methods. They share 80% of the same
validation logic that should be extracted into a helper.
```

### 技巧 5：充分利用 CLAUDE.md

把项目的编码规范、常用命令、架构决策都写进 CLAUDE.md，避免每次会话重复解释。

---

## 11. 常用命令速查表

### 会话命令

| 命令 | 功能 |
|------|------|
| `claude` | 启动交互式会话 |
| `claude "your task"` | 直接执行任务（headless 模式）|
| `claude --permission-mode plan` | 以 Plan Mode 启动 |
| `/help` | 查看所有可用命令 |
| `/resume` | 继续上一个会话 |
| `/login` | 切换账号 |
| `/logout` | 退出登录 |

### 上下文管理

| 命令 | 功能 |
|------|------|
| `/compact` | 压缩上下文，释放 token |
| `/clear` | 清空当前会话上下文 |
| `Shift+Tab` | 切换权限模式（Normal → Auto-Accept → Plan）|

### 项目配置

| 命令 | 功能 |
|------|------|
| `/init` | 自动生成 CLAUDE.md |
| `/agents` | 查看 / 创建 Subagents |
| `/install-github-app` | 安装 GitHub Actions 集成 |

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | 中断当前操作 |
| `Ctrl+G` | 在编辑器中打开计划（Plan Mode 中）|
| `Shift+Tab` | 切换权限模式 |

---

## 参考资源

- 官方文档：https://code.claude.com/docs
- 文档索引：https://code.claude.com/docs/llms.txt
- GitHub Actions 示例：https://github.com/anthropics/claude-code-action/tree/main/examples
- Claude 定价：https://claude.com/pricing

---

*本笔记整理自 Anthropic 官方文档，涵盖 Claude Code 核心功能与最佳实践。*
*最后更新：2026-03-20*
