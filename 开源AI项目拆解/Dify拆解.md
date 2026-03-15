>    项目地址：https://github.com/langgenius/dify
	Dify 是一个开源的 LLM (Large Language Model) 应用开发平台，提供直观的界面来构建 AI 驱动的应用程序。它将多种强大的功能整合到一个统一的平台上：AI 工作流、RAG (Retrieval-Augmented Generation) 管道、Agent 编排、模型管理以及全面的观测性功能


# 01 模型管理

现在市面上大模型几乎都适配OpenAI的API协议（已在事实上成为行业标准）。

**API请求的三个核心部分**

- **URL（端点）**：固定路径 `/v1/chat/completions`，这是所有厂商兼容的关键——路径一样，只换域名前缀（`base_url`）
- **Headers**：告诉服务器"我是谁、我要发什么格式"。`Authorization: Bearer sk-xxx` 就是身份凭证，也就是 API Key
- **Body**：JSON 格式，核心是 `messages` 数组，按 `role`（system / user / assistant）顺序排列整个对话历史
![[../Pictures/Pasted image 20260315151747.png]]

![[../Pictures/Pasted image 20260315152246.png]]


# 02 工具管理






# 03 知识库管理





# 04 工作室管理



## 0401 工作流




## 0402 ChatFlow



## 0403 Agent


