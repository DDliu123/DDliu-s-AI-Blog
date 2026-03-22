
> 官方文档：https://code.claude.com/docs/en/overview

---

## 一、什么是 Claude Code

Claude Code 是 Anthropic 推出的**终端原生 AI 编程代理**，通过自然语言与你对话，直接读写代码、执行命令、管理 Git，覆盖整个开发工作流。

**核心能力：**

- 跨多文件理解和修改代码
- 写测试、修 lint、解 merge conflict、写 release notes
- 用自然语言描述需求 → 自动规划 + 多文件实现 + 验证
- 粘贴报错信息 → 溯源 + 定位根因 + 修复
- 原生操作 Git（暂存、提交、建分支、开 PR）

---

## 二、安装与启动

### 安装方式

|平台|命令|
|---|---|
|macOS (Native)|官网下载，自动后台更新|
|macOS (Homebrew)|`brew install claude-code`（需手动 `brew upgrade claude-code`）|
|Windows (WinGet)|`winget install Anthropic.ClaudeCode`（需手动升级）|
|Windows|需先安装 Git for Windows|

> ⚠️ npm 安装方式已废弃，请用官方推荐方式。

### 启动

```bash
cd your-project
claude
```

首次运行会提示登录。需要 Claude Pro / Max / Teams / Enterprise 订阅或 API 账号。

---

## 三、使用界面（多端同一引擎）

|界面|特点|
|---|---|
|**终端 CLI**|完整功能，直接在终端运行|
|**VS Code 扩展**|inline diff、@mention、计划审查、对话历史|
|**JetBrains 插件**|适用于 IDEA / PyCharm / WebStorm|
|**桌面 App**|可视化 diff、多 session 并排、定时任务|
|**Web 版**|浏览器内运行，无需本地安装，支持长任务|
|**GitHub Actions / GitLab CI**|CI/CD 集成，自动 code review|
|**Slack**|在 Slack 频道中 @claude|

所有界面共用同一份 CLAUDE.md、settings.json 和 MCP 配置。

---

## 四、核心工作原理（Agentic Loop）

Claude Code 接到任务后分三阶段循环执行：

```
1. 收集上下文（搜索文件、读代码、理解结构）
   ↓
2. 执行动作（编辑文件、运行命令、调用工具）
   ↓
3. 验证结果（运行测试、检查输出、确认正确）
   ↓ 循环直到完成
```

**启动时 Claude 可访问的内容：**

- 当前目录及子目录的所有文件
- 终端（任何可执行命令：构建工具、git、包管理器等）
- Git 状态（当前分支、未提交变更、历史记录）
- CLAUDE.md 文件（你写的项目上下文）
- Auto Memory（自动保存的学习记录，前 200 行自动加载）

---

## 五、持久化记忆：CLAUDE.md

CLAUDE.md 是放在项目根目录的 Markdown 文件，Claude 每次会话都会读取。

**用来写什么：**

- Bash 构建命令（`npm run dev`、`make test` 等）
- 代码风格约定
- 架构决策
- 禁止做的事
- 审查 checklist

**快速生成：**

```bash
/init   # 自动分析项目结构生成初始 CLAUDE.md
```

**加载规则：**

- 工作目录及父目录的 CLAUDE.md 在启动时加载
- 子目录的 CLAUDE.md 在进入该目录时加载
- 指令冲突时，越具体的优先级越高

**Compact Instructions 节：** 在 CLAUDE.md 中加 `## Compact Instructions` 节，控制上下文压缩时保留什么。

---

## 六、权限系统

### 权限模式（Shift+Tab 切换）

|模式|说明|
|---|---|
|**Normal**（默认）|每次操作前询问|
|**Auto-Accept** (`⏵⏵`)|文件操作自动批准，Bash 命令仍需确认|
|**Plan Mode** (`⏸`)|只分析不执行，Claude 会先制定计划再等你确认|
|**bypassPermissions**|完全跳过所有权限提示（仅在隔离环境中使用）|

启动时指定模式：

```bash
claude --permission-mode plan
```

### 权限规则配置（`.claude/settings.json`）

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  }
}
```

规则格式：`Tool` 或 `Tool(specifier)`，支持 glob 通配符。

### 配置层级（高→低）

```
Managed（企业管控）> 用户设置 > 项目设置 > 本地设置
```

---

## 七、扩展系统

### 扩展功能对比

|功能|作用|加载方式|适用场景|
|---|---|---|---|
|**CLAUDE.md**|持久上下文|每次会话自动加载|项目约定、固定规则|
|**Skills**|可复用知识/工作流|按需 or 手动 `/skill-name`|可重用的流程和知识|
|**Subagents**|独立上下文的 AI 子代理|Claude 自动委托或手动调用|隔离复杂任务|
|**Hooks**|自动化 shell 脚本|特定事件触发|格式化、lint、通知|
|**MCP**|外部服务连接|配置后可用|Jira、Figma、数据库等|
|**Plugins**|打包上述所有扩展|从 marketplace 安装|团队共享、批量配置|

---

## 八、Skills（技能）

Skills 是可复用的提示词 + 工作流，创建后用 `/skill-name` 调用，或由 Claude 自动识别触发。

### 创建 Skill

在 `~/.claude/skills/explain-code/SKILL.md`（全局）或 `.claude/skills/xxx/SKILL.md`（项目级）创建：

```markdown
---
name: explain-code
description: Explains code with visual diagrams. Use when explaining how code works.
---

When explaining code, always include:
1. Start with an analogy
2. Draw an ASCII diagram
3. Walk through step-by-step
4. Highlight a common gotcha
```

- `name` → 成为 `/explain-code` 命令
- `description` → Claude 判断何时自动触发
- 设 `disable-model-invocation: true` → 仅手动调用，不占 context

---

## 九、Subagents（子代理）

Subagents 是独立的 AI 助手，拥有**独立上下文**、**定制 system prompt** 和**专属工具权限**。

### 内置子代理

|代理|模型|用途|
|---|---|---|
|Explore|Haiku|只读搜索和代码分析|
|Research|Sonnet|Plan Mode 时收集上下文|
|statusline-setup|Sonnet|配置状态栏|
|Claude Code Guide|Haiku|回答 Claude Code 使用问题|

### 自定义子代理

在 `~/.claude/agents/`（全局）或 `.claude/agents/`（项目级）创建 `.md` 文件：

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior security engineer. Review code for:
- Injection vulnerabilities
- Authentication flaws
- Secrets in code
- Insecure data handling

Provide specific line references and suggested fixes.
```

**触发方式：**

```
"Use a subagent to review this code for security issues."
```

**何时用 Subagents vs Skills：**

- 任务输出量大、不想污染主上下文 → **Subagent**
- 需要强制限制工具权限 → **Subagent**
- 可复用的提示/工作流，在主对话中执行 → **Skill**

---

## 十、Hooks（自动化钩子）

Hooks 在 Claude Code 生命周期的特定节点自动执行 shell 命令，**强制执行、不可跳过**（不同于 CLAUDE.md 的建议性指令）。

### 主要事件

|事件|触发时机|
|---|---|
|`PreToolUse`|工具调用前（可 allow/deny/ask）|
|`PostToolUse`|工具调用后|
|`Notification`|Claude 等待用户输入时|
|`SessionEnd`|会话结束时|
|`ConfigChange`|配置文件被修改时|

### 配置示例

```json
// ~/.claude/settings.json
{
  "hooks": {
    "Notification": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "osascript -e 'display notification \"Claude needs input\" with title \"Claude Code\"'"
      }]
    }],
    "PostToolUse": [{
      "matcher": "Edit",
      "hooks": [{
        "type": "command",
        "command": "npm run lint --fix"
      }]
    }]
  }
}
```

**让 Claude 帮你写 Hook：**

```
"Write a hook that runs eslint after every file edit"
"Write a hook that blocks writes to the migrations folder"
```

---

## 十一、MCP（Model Context Protocol）

MCP 是连接外部服务的开放标准，让 Claude 可操作 Jira、Figma、数据库、Slack 等。

```bash
claude mcp add    # 添加 MCP 服务器
/mcp              # 查看当前 MCP 配置及上下文消耗
```

**支持的集成示例：** Google Drive、Jira、Figma、Notion、数据库、自定义 API

MCP 工具在 Hooks 中的命名格式：`mcp__<server>__<tool>`

---

## 十二、常用命令速查

### 内置斜杠命令

|命令|说明|
|---|---|
|`/init`|生成初始 CLAUDE.md|
|`/compact`|压缩上下文（可附加焦点：`/compact focus on API changes`）|
|`/context`|查看上下文使用情况|
|`/permissions`|查看和管理权限规则|
|`/hooks`|浏览当前 Hook 配置|
|`/mcp`|查看 MCP 服务器状态|
|`/model`|切换模型（sonnet/opus/haiku）|
|`/agents`|管理子代理|
|`/plugin`|浏览插件市场|
|`/clear`|清除会话|
|`/rename`|重命名当前会话|
|`/rewind`|回退到上一个检查点|
|`/bug`|报告 Bug|
|`/btw`|在不占历史的情况下快速提问|

### 快捷键

|快捷键|功能|
|---|---|
|`Shift+Tab`|切换权限模式（Normal → Auto-Accept → Plan）|
|`Ctrl+R`|搜索提示历史|
|`Ctrl+G`|在编辑器中打开当前计划|
|`Esc × 2`|回退到上一个检查点|
|`Ctrl+O`|进入 verbose 模式（显示详细工具输出）|

### CLI 标志

```bash
claude -p "your prompt"                   # headless 非交互模式
claude --permission-mode plan             # 以 Plan 模式启动
claude --model opus                       # 指定模型
claude --dangerously-skip-permissions     # 跳过权限（仅沙箱环境）
claude --agent my-agent                   # 以指定子代理启动
claude --name "session name"              # 命名会话
```

---

## 十三、上下文管理

- 上下文满时 Claude 自动压缩：优先清除旧工具输出，再摘要对话
- 早期对话中的详细指令可能丢失 → **重要规则写进 CLAUDE.md**
- MCP 服务器会消耗大量上下文 → 用 `/mcp` 检查
- Skills 默认只加载描述，实际内容按需加载
- Subagents 拥有独立上下文，不占主对话 context

---

## 十四、安全机制

- **Checkpoints（检查点）**：每次编辑文件前自动快照，随时用 `/rewind` 或 `Esc×2` 回退
- **Checkpoints 保留 30 天**
- **三种回退模式**：仅回退对话 / 仅回退代码 / 两者都回退
- **Git Worktrees**：子代理可在独立 worktree 中并行工作，互不冲突
- 建议搭配 **版本控制系统（Git）** 使用，检查点不替代 Git

---

## 十五、最佳实践

### 提示技巧

- 像和高级工程师对话一样提问，不要过度简化
- 浏览新代码库时，直接问"这个模块怎么运行的"而不是自己读
- 粘贴错误信息让 Claude 定位根因
- 用 `cat error.log | claude -p "分析这些日志"` 管道传输数据
- 给 Claude 文档和 API 参考的 URL
- 告诉 Claude 用 `gh`、`aws`、`gcloud` 等 CLI 工具操作外部服务

### 提示词策略

- 任务清晰具体时 → 直接给指令
- 复杂多文件改动 → 先用 **Plan Mode** 让 Claude 制定计划再确认执行
- 探索性任务 → 可以故意用模糊提示，观察 Claude 的理解再纠正
- 注意总结"哪些提示方式效果好"，形成个人最佳实践

### 工作流配置建议（从简到繁）

1. 先运行 `/init` 生成 CLAUDE.md，逐步完善
2. 常用命令 → 写成 Skill
3. 需要外部服务 → 接入 MCP
4. 需要每次必须执行的动作 → 配置 Hooks
5. 复杂隔离任务 → 定义 Subagents
6. 团队共享配置 → 打包成 Plugin

---

## 十六、Unix 哲学集成

Claude Code 遵循 Unix 管道哲学，可与其他工具任意组合：

```bash
# 分析日志并通知
tail -200 app.log | claude -p "发现异常请通过 Slack 告知"

# CI 自动翻译
claude -p "将新增字符串翻译成法语并发起 PR"

# 批量安全审查
git diff main --name-only | claude -p "检查这些改动文件的安全问题"
```

---

## 十七、模型选择

|别名|实际模型|适用场景|
|---|---|---|
|`sonnet`（默认）|Claude Sonnet 4.x|日常编码，性价比最高|
|`opus`|Claude Opus 4.x|复杂推理、架构设计|
|`haiku`|Claude Haiku 4.x|快速响应、低延迟任务|

切换：`/model` 命令或 `--model` 启动参数

---

_文档基于 Claude Code 官方文档整理，最后更新：2026年3月_