>    项目地址：https://github.com/langgenius/dify
	Dify 是一个开源的 LLM (Large Language Model) 应用开发平台，提供直观的界面来构建 AI 驱动的应用程序。它将多种强大的功能整合到一个统一的平台上：AI 工作流、RAG (Retrieval-Augmented Generation) 管道、Agent 编排、模型管理以及全面的观测性功能

# Dify整体架构

[[../dify_product_architecture.svg]]

### 一、应用类型层：四种范式解决不同场景

Dify 提供四种核心应用类型：Chatbot（对话助手，入门级解决方案，快速搭建智能对话系统）、Agent（进阶型框架，赋予 AI 自主决策与执行能力，基于用户意图动态推理并调用工具）、Chatflow（支持上下文记忆的多轮交互系统，适合客户服务、教育辅导等需要追踪对话上下文的场景）、以及 Workflow（面向单次任务的自动化批处理引擎，适合数据提取、内容生成等场景）。

这四种类型的核心差异在于**谁来控制流程**：Chatbot 是用户主导的自由对话；Workflow 是开发者预定义的确定性流程；Agent 是 AI 自主规划的动态流程；Chatflow 是带记忆的半结构化流程。

### 二、核心能力层：三大引擎

**RAG 知识库引擎**

文档上传到知识库后需要进行分块和数据清洗，支持自动模式和自定义模式。检索支持三种方式：向量搜索（将查询向量化计算距离）、全文搜索（关键字匹配）、以及混合搜索（结合两者优势）；还支持 Rerank 模型对检索结果进行语义重排序优化。

**可视化工作流引擎**

工作流节点类型丰富：知识检索节点从知识库寻找相关内容；问题分类节点通过 LLM 推理匹配分类；条件分支节点支持 If/else/elif 条件切分流程；代码节点支持 Python/NodeJS 运行；模板节点基于 Jinja2 进行数据转换。 工作流的价值在于**把不确定的 AI 输出，锁定在确定性的流程结构里**，显著提升可控性。

**Agent 框架**

Dify 的 Agent 支持两种推理模式：Function Calling（对于 GPT-3.5、GPT-4 等支持该模式的模型，建议使用，性能更稳定）；以及 ReAct 框架（为不支持 Function Calling 的模型提供替代方案）。

# 01 模型管理

现在市面上大模型几乎都适配OpenAI的API协议（已在事实上成为行业标准）。

**API请求的三个核心部分**

- **URL（端点）**：固定路径 `/v1/chat/completions`，这是所有厂商兼容的关键——路径一样，只换域名前缀（`base_url`）
- **Headers**：告诉服务器"我是谁、我要发什么格式"。`Authorization: Bearer sk-xxx` 就是身份凭证，也就是 API Key
- **Body**：JSON 格式，核心是 `messages` 数组，按 `role`（system / user / assistant）顺序排列整个对话历史
- ![[../Pictures/Pasted image 20260315153229.png]]
![[../Pictures/Pasted image 20260315151747.png]]

![[../Pictures/Pasted image 20260315152246.png]]


# 02 工具管理






# 03 知识库管理





# 04 工作室管理



## 0401 工作流




## 0402 ChatFlow



## 0403 Agent


