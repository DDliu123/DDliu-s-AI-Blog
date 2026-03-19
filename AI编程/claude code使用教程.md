# 添加上下文
在使用Claude处理编码项目时，上下文管理至关重要。你的项目可能有几十甚至几百个文件，但Claude只需要正确的信息就能有效地帮助你。过多不相关的上下文实际上会降低Claude的性能，因此学会引导它找到相关文件和文档是必不可少的。
## /init命令
当你第一次在新项目中启动Claude时，运行/init命令。这会告诉Claude分析你的整个代码库并理解
- 项目的目的和架构
- 重要命令和关键文件
- 编码模式和结构
分析完你的代码后，Claude会创建一个摘要并将其写入CLAUDE.md文件。当Claude请求创建此文件的权限时，你可以按Enter键批准每个写入操作，或者按Shift+Tab让Claude在整个会话期间自由写入文件。

## CLAUDE.md文件
CLAUDE.md文件有两个主要用徐：
- 引导Claude浏览你的代码库，指出重要命令、架构和编码风格
- 允许你给Claude特定或自定义的指令
这个文件会包，含在你向Claude发出的每个请求中，所以它就像是你项目的特久系统提示。

Claude识别三个不同位置的三个不同CLAUDE.md文件：
- CLAUDE.md-用/init生成，提交到源代码控制，与其他工程师共享
- CLAUDE.local.md-不与其他工程师共享，包含个人指令和对Claude的自定义
- ~/.claude/CLAUDE.md-用于你机器上的所有项目，包含你希望Claude在所有项目中遵循的指令

## 添加自定义指令
你可以通过向CLAUDE.md文件添加指令来自定义Claude的行为。例如，如果Claude在代码中添加了太多注释，你可以通过更新文件来解决这个问题。使用#命令进入"记忆模式”一这让你可以智能地编辑你的CLAUDE.md文件。只需输入类似这样的内容：
```
# Use comments sparingly. Only comment complex code.
```
## 使用'@'提及文件
当你需要Claude查看特定文件时，使用@符号后跟文件路径。这会自动将该文件的内容包含在你向Claude的请求中。
例如，如果你想询问你的身份验证系统并且你知道相关文件，你可以输入：
```
How does the auth system work? @auth
```
## 在CLAUDE.md中引用文件
你也可以使用相同的@语法直接在你的CLAUDE.md文件中提及文件。这对于与项目许多方面相关的文件特别有用。
例如，如果你有一个定义数据结构的数据库模式文件，你可以将其添加到你的CLAUDE.md中：
``` markdown
数据库架构定义在 `@prisma/schema.prisma` 文件中。每当你需要了解数据库中存储的数据结构时，都可以参考此文件。
```

当你以这种方式提及文件时，它的内容会自动包含在每个请求中，所以Claude可以立即回答关于你数据结构的问
题，而无需每次都搜索和读取模式文件。





---

## 🧠 核心理念：管理上下文窗口是第一要务

Claude Code 最重要的资源是上下文窗口。整个对话历史——包括每条消息、读取的文件、命令输出——都会占用上下文。当上下文接近上限时，模型性能会明显下降，Claude 可能开始"遗忘"早期指令或犯更多错误。

---

## 📁 一、配置 CLAUDE.md（项目记忆文件）

CLAUDE.md 是默认会进入每次对话的唯一文件。它需要告诉 Claude 三件事：**WHAT**（技术栈和项目结构）、**WHY**（项目目的和各模块功能）、**HOW**（如何在项目中工作，例如用 bun 还是 node，如何跑测试）。

**写 CLAUDE.md 的关键原则：**

- 研究表明，前沿思考型 LLM 能可靠遵循约 150-200 条指令。Claude Code 系统提示本身已包含约 50 条指令，因此你的 CLAUDE.md 应尽量精简，只放普遍适用的核心规则。
- 不要在 CLAUDE.md 里直接 @-file 文档（会每次嵌入整个文件撑爆上下文），而是提示 Claude 在需要时去读它，例如："遇到 FooBarError 时，参见 path/to/docs.md"。
- 避免只写"禁止"类约束，如"永远不要用 --foo-bar"。这会让 agent 卡住。应始终提供替代方案。
- 在子目录（如 /frontend 或 /backend）中创建额外的 CLAUDE.md，给 Claude 更聚焦的上下文。

---

## ✍️ 二、写出高质量的 Prompt

有效的 prompt 遵循 **CIF 结构：Context（上下文）、Intent（意图）、Format（输出格式）**。Claude Code 不会猜测你的目标，你必须明确说明。

**实用技巧：**

- 直接引用文件路径；提供 URL 让 Claude 自行读取文档或 Issue；拖入截图或设计稿以提供视觉上下文；明确指定边界条件。
- 参考具体示例文件，例如："按照 components/UserCard.tsx 的模式创建新组件"——一个具体例子胜过百字说明。
- 在提示词中加入 `ultrathink` 关键词可触发高强度推理模式。

---

## 🗂️ 三、上下文管理策略

**频繁清理上下文：**

- 使用 `/clear` 的建议：每次开始新任务都清空对话。旧历史会消耗 token，还会触发 Claude 的压缩调用，进一步浪费上下文。
- 在上下文使用率达到 50% 前手动执行 `/compact`，避免进入"agent 迟钝区"。
- 使用 `/context` 查看当前上下文占用，用 `/usage` 查看计划用量。

---

## 📐 四、先规划，再编码

Plan Mode 通过按两次 **Shift+Tab** 激活。在此模式下，Claude 成为研究和分析机器，无法修改任何文件——相当于"架构师模式"，只能观察、分析和规划，直到你批准才会执行。

规划优先是不可妥协的原则：每个高质量来源都强调，在编码前进行预先规划。"直觉编码"适合快速验证原型，但生产代码需要结构化思考、验证和文档。

---

## ⚡ 五、实用操作技巧

几个容易忽视的操作细节：

- **停止 Claude**：用 Escape，不是 Ctrl+C（后者会直接退出）
- **引用文件**：拖入时按住 Shift，否则会在新标签页打开
- **粘贴图片**：用 Ctrl+V，而非 Cmd+V
- **跳转历史消息**：按两次 Escape 显示历史消息列表

---

## 🔄 六、权限与自动化

- 运行 `claude --dangerously-skip-permissions` 可跳过每次操作的权限确认，类似 Cursor 的 "yolo 模式"，大幅提升自动化流畅度。
- 使用 `/install-github-app` 让 Claude 自动审查 PR，它尤其擅长发现逻辑错误和安全问题（而非纠结变量命名）。记得自定义审查 prompt，默认配置过于冗长。

---

## 🔧 七、Git 与代码质量

- 让 Claude 在完成每个任务步骤后就提交 commit，养成小步提交的习惯。
- 在 prompt 中附上测试用例、截图或预期输出，让 Claude 能自我验证——这是单一最高杠杆率的操作。
- 对 AI 生成的代码保持人工审查：AI 生成的代码表面上往往"能跑"，但可能包含微妙的 bug，测试是唯一可靠的验证机制。

---

## 🚀 八、进阶：并行与子 Agent

- 在 IDE 中打开多个 Claude Code 实例并行工作，前提是各实例处理代码库的不同部分。
- 在 prompt 中说 `use subagents` 可以让 Claude 将子任务分发给子 Agent，保持主上下文干净。

---

**一句话总结**：Claude Code 的使用上限取决于你对上下文的管理精度、prompt 的结构化程度，以及是否养成了"先规划再执行"的工作习惯。


---

CLAUDE.md 支持**多个位置**，采用分级加载机制，优先级由高到低：

---

### 📍 三个主要存放位置

**1. 项目根目录（最常用）**

```
your-project/
└── CLAUDE.md   ← 放这里，团队共享
```

这是最常见的位置，应该提交到 Git，用于与整个团队共享项目特定的指令、命令和代码风格规范。

**2. 用户主目录（全局个人设置）**

```
~/.claude/CLAUDE.md
```

位于 home 目录，对你所有项目生效，适合存放个人偏好、编码风格或到处都用的自定义工具快捷方式。

**3. 子目录（分模块精细控制）**

```
your-project/
├── CLAUDE.md
├── frontend/
│   └── CLAUDE.md   ← 仅前端相关指令
└── backend/
    └── CLAUDE.md   ← 仅后端相关指令
```

还可以用 `.claude/rules/` 目录将指令拆分成多个文件，并支持按文件路径作用域加载，只有 Claude 打开匹配文件时才会加载对应规则，节省上下文空间。

---

### 🔢 加载优先级

优先级顺序为：`.claude/settings.local.json` > 项目 `CLAUDE.md` > 全局 `~/.claude/CLAUDE.md`。

Claude Code 启动时会从当前工作目录**递归向上**查找所有层级的 CLAUDE.md，并自动合并加载。

---

### ⚡ 快速创建

最简单的方式是在项目目录下运行：

```bash
/init
```

Claude 会分析你的代码库，自动生成一个基础的 CLAUDE.md 文件，建议将其提交到版本控制。

这是个很好的问题！简单来说，**这两个位置的 CLAUDE.md 都需要手动创建**，没有专门的自动生成命令。这两种 CLAUDE.md **都需要手动创建**，没有专用命令自动生成，但方法很简单：

---

### 🌐 用户主目录（`~/.claude/CLAUDE.md`）

直接用任意文本编辑器或命令行创建即可：

```bash
# 方式一：用命令行创建并编辑
mkdir -p ~/.claude
nano ~/.claude/CLAUDE.md

# 方式二：用 VS Code 打开编辑
code ~/.claude/CLAUDE.md
```

这个全局目录 `~/.claude/` 是 Claude Code 自动维护的系统级目录，里面存放跨所有项目生效的全局设置。`CLAUDE.md` 就放在其中，适合写你的个人编码偏好、惯用工具和习惯。

写什么内容？典型示例：

```markdown
# 我的个人全局偏好
- 回复时使用中文
- 代码注释用英文
- 优先使用函数式写法
- 提交信息使用 Conventional Commits 格式
- 测试框架首选 Vitest
```

---

### 📂 子目录（`frontend/CLAUDE.md`、`backend/CLAUDE.md` 等）

同样是手动在对应子目录创建：

```bash
# 在 frontend 子目录创建
touch your-project/frontend/CLAUDE.md

# 在 backend 子目录创建
touch your-project/backend/CLAUDE.md
```

在 monorepo 中，Claude Code 会从你当前工作目录向上遍历整个目录树，加载沿途找到的所有 CLAUDE.md。比如你在 `packages/web/` 目录运行 Claude Code，它会同时加载根目录的 `CLAUDE.md` 和 `packages/web/CLAUDE.md`。

> ⚠️ **注意一个已知问题**：子目录的 CLAUDE.md 并不是在启动时就加载，而是只有当 Claude 实际读取到该子目录下的文件时，才会触发加载对应的 CLAUDE.md。 所以不要期望子目录规则在 session 一开始就生效。

---

### 💡 一个更优雅的替代方案：`@import` 语法

你可以在项目根目录的 CLAUDE.md 中，用 `@~/.claude/CLAUDE.md` 语法显式引入全局个人设置，避免重复维护。

```markdown
# 项目根目录的 CLAUDE.md
@~/.claude/CLAUDE.md   ← 引入全局个人偏好

## 项目专属规则
- 使用 Next.js App Router
- 数据库用 Drizzle ORM
```

这样就形成了一个清晰的分层结构：个人偏好统一维护在全局文件里，项目规则只写项目独有的内容。