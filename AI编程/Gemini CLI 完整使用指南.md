
> 官方文档：https://geminicli.com/docs/

---

## 一、安装与启动

```bash
# 全局安装（需要 Node.js 18+）
npm install -g @google/gemini-cli

# 启动
gemini

# 无需安装直接运行
npx @google/gemini-cli
```

---

## 二、认证

启动后选择认证方式：

1. **Sign in with Google**（推荐，个人账号）→ 浏览器授权即可
2. **API Key**（适合 CI/无头环境）→ 设置环境变量 `GEMINI_API_KEY`
3. **Google Cloud / Vertex AI**（企业场景）→ 需配置 GCP 项目

**免费额度（个人 Google 账号）：** 60 次请求/分钟，1000 次请求/天

---

## 三、三大命令前缀

|前缀|用途|示例|
|---|---|---|
|`/`|Slash 命令，控制 CLI 自身行为|`/help`、`/model set gemini-2.5-pro`|
|`@`|注入文件/目录内容到提示词|`@src/ 解释这段代码`|
|`!`|执行 Shell 命令或切换 Shell 模式|`!git status`|

---

## 四、Slash 命令速查（`/`）

### 会话与历史

|命令|说明|
|---|---|
|`/resume` / `/chat`|打开会话浏览器，恢复历史会话（自动保存，无需手动操作）|
|`/chat save <tag>`|手动保存当前对话为标记检查点|
|`/chat resume <tag>`|恢复指定标记的对话|
|`/chat share [file]`|导出对话为 `.md` 或 `.json` 文件|
|`/chat delete <tag>`|删除指定检查点|
|`/restore [tool_call_id]`|撤销工具执行前的文件改动（需开启 checkpointing）|
|`/rewind`|回退对话历史并可选择撤销文件变更（快捷键：连按两次 `Esc`）|
|`/clear`|清屏（快捷键：`Ctrl+L`）|
|`/compress`|将上下文压缩为摘要，节省 Token|

### 模型与配置

|命令|说明|
|---|---|
|`/model set <name>`|切换模型（如 `gemini-2.5-pro`）|
|`/model manage`|打开图形化模型配置对话框|
|`/settings`|打开设置编辑器（等同编辑 `.gemini/settings.json`）|
|`/theme`|切换主题|
|`/vim`|切换 Vim 输入模式（支持 NORMAL/INSERT 模式）|
|`/editor`|选择外部编辑器|

### 内存与上下文

| 命令                   | 说明                        |
| -------------------- | ------------------------- |
| `/memory add <text>` | 向 AI 记忆中添加内容              |
| `/memory show`       | 查看当前加载的所有 `GEMINI.md` 内容  |
| `/memory list`       | 列出所有 `GEMINI.md` 文件路径     |
| `/memory refresh`    | 重新加载所有 `GEMINI.md` 文件     |
| `/init`              | 分析当前项目目录，自动生成 `GEMINI.md` |

### 工具与扩展

|命令|说明|
|---|---|
|`/tools`|列出可用工具|
|`/tools desc`|显示工具详细描述|
|`/mcp list`|列出已配置的 MCP 服务器和工具|
|`/mcp refresh`|重启 MCP 服务器并重新发现工具|
|`/mcp auth <server>`|为 MCP 服务器进行 OAuth 认证|
|`/extensions list`|列出已安装扩展|
|`/extensions install <url>`|从 Git 仓库或本地路径安装扩展|
|`/skills list`|列出所有 Agent Skills|
|`/skills enable/disable <n>`|启用/禁用指定 Skill|

### 工作区与权限

|命令|说明|
|---|---|
|`/directory add <path>`|添加工作目录（支持多目录）|
|`/directory show`|查看当前工作目录列表|
|`/permissions trust [path]`|信任指定目录|

### 计划与自动化

|命令|说明|
|---|---|
|`/plan`|切换到 Plan Mode（只读模式，先规划再执行）|
|`/plan copy`|复制当前计划到剪贴板|
|`/agents list`|列出所有子 Agent（需开启实验性功能）|
|`/hooks list`|列出所有 Hooks|

### 其他实用命令

| 命令                | 说明                   |
| ----------------- | -------------------- |
| `/stats session`  | 查看当前会话统计（工具调用次数、时长等） |
| `/stats model`    | 查看 Token 用量和配额       |
| `/copy`           | 复制最后一条输出到剪贴板         |
| `/shells`         | 查看/管理后台运行的 Shell 进程  |
| `/bug <描述>`       | 快速提交 GitHub Issue    |
| `/upgrade`        | 升级账号套餐               |
| `/about`          | 查看版本信息               |
| `/quit` / `/exit` | 退出                   |

### 自定义命令

在 `~/.gemini/commands/` 或 `<project>/.gemini/commands/` 目录下创建 `.toml` 文件定义快捷命令，通过 `/commands reload` 重载。

---

## 五、`@` 文件注入命令

```bash
# 注入单个文件
@README.md 解释这份文档

# 注入整个目录（自动过滤 .gitignore 中的文件）
@src/ 概括这个项目的代码结构

# 路径含空格时转义
@My\ Documents/file.txt 翻译这个文件

# 混合使用
这个函数有什么问题？@lib/utils.js
```

> **说明：** `@` 命令默认会忽略 `node_modules/`、`dist/`、`.env`、`.git/` 等被 git 忽略的内容，可通过 `context.fileFiltering` 设置修改此行为。

---

## 六、`!` Shell 命令

```bash
# 单次执行 shell 命令
!ls -la
!git status
!npm run build

# 输入 ! 单独切换到 Shell 模式（所有输入直接作为 shell 命令执行）
!
# 再次输入 ! 退出 Shell 模式
```

> **注意：** Shell 模式下的命令拥有与终端相同的权限，请谨慎操作。

---

## 七、核心配置文件

### `GEMINI.md` — 项目上下文记忆

放置在以下位置，Gemini CLI 会自动读取（层级加载）：

- `~/.gemini/GEMINI.md`（全局）
- `<project>/GEMINI.md`（项目级）
- 任意子目录中的 `GEMINI.md`

**用途：** 描述项目背景、编码规范、常用命令等，让每次对话都带有项目上下文。

```bash
# 快速生成 GEMINI.md
/init
```

### `.gemini/settings.json` — 设置文件

```json
{
  "theme": "dark",
  "model": "gemini-2.5-pro",
  "experimental": {
    "enableAgents": true,
    "plan": true
  },
  "context": {
    "fileFiltering": true
  }
}
```

通过 `/settings` 可视化编辑，或直接修改文件。

### `.geminiignore` — 忽略文件

语法同 `.gitignore`，控制 `@` 命令和文件工具读取时忽略哪些文件。

---

## 八、核心功能概览

### Plan Mode（计划模式）

先生成只读计划，确认后再执行——适合复杂、高风险任务。用 `/plan` 进入。

### Checkpointing（检查点）

工具执行文件变更前自动创建快照，可用 `/restore` 一键回滚。

### MCP Servers（模型上下文协议）

扩展 Gemini CLI 能力，连接外部服务（数据库、API 等）。配置后用 `/mcp list` 查看。

### Agent Skills（Agent 技能）

按需加载的专业工作流模块（如代码审查、测试生成等），用 `/skills list` 管理。

### Headless Mode（无头模式）

适合 CI/CD 管道，非交互式运行：

```bash
gemini --prompt "解释这个文件" --input_file main.py
```

### Rewind（回溯）

通过 `/rewind` 或连按两次 `Esc`，回退对话并可选择撤销对应的文件改动。

### Subagents（子 Agent）

实验性功能，协调多个 AI Agent 完成复杂并行任务，需在 `settings.json` 中开启 `experimental.enableAgents: true`。

---

## 九、快捷键总结

|快捷键|功能|
|---|---|
|`Ctrl+L`|清屏|
|`Esc` × 2|Rewind（回退对话）|
|`Alt+Z` / `Cmd+Z`|撤销输入框操作|
|`Shift+Alt+Z` / `Shift+Cmd+Z`|重做输入框操作|
|`Ctrl+K`（在 CLI 界面）|搜索命令|

---

## 十、实用技巧

1. **多目录工作区：** 用 `/directory add` 一次管理多个项目目录
2. **Token 不够用？** `/compress` 压缩上下文；或用 `/stats model` 查看配额
3. **复用常用提示词：** 创建自定义命令 `.toml`，用 `/your-command` 一键调用
4. **项目记忆：** 维护好 `GEMINI.md`，让每次会话都"懂"你的项目
5. **回滚误操作：** 开启 Checkpointing 后，任何文件改动都可用 `/restore` 撤回
6. **查看 Token 用量：** `/stats model` 实时掌握配额消耗情况
7. **导出对话：** `/chat share report.md` 将对话存为 Markdown 文档留存