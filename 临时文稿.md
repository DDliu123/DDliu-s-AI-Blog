>    项目地址：https://github.com/langgenius/dify
	Dify 是一个开源的 LLM (Large Language Model) 应用开发平台，提供直观的界面来构建 AI 驱动的应用程序。它将多种强大的功能整合到一个统一的平台上：AI 工作流、RAG (Retrieval-Augmented Generation) 管道、Agent 编排、模型管理以及全面的观测性功能

# Dify整体架构

![[../Pictures/Pasted image 20260315165703.png]]

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

### 三、扩展生态层：v1.0 的架构转折点

2025 年 2 月 28 日发布的 v1.0.0 是 Dify 发展史上的分水岭，彻底重构底层架构，将原本耦合在核心代码中的模型和工具全部迁移为插件，形成"核心框架 + 插件生态"的分层结构。最具革命性的变化是引入 Agent 策略插件，允许开发者通过 ReAct/Function Calling 等模式定义智能体决策逻辑，配合新推出的 Marketplace，迅速汇聚了 120+ 官方与社区插件。热插拔插件系统采用 Python 的 importlib 动态加载机制，实现插件的即插即用，插件更新无需重启服务。

MCP 协议的接入是另一个重要变化：Dify 支持将构建好的工作流或 Agent 直接转变为标准 MCP Server，使其可被无限数量的 MCP 客户端访问；同时也支持通过标准化 MCP 协议接入外部 API、数据库和服务，消除集成复杂性。

### 四、LLMOps 运营层：区别于传统开发平台的核心

LLMOps 模块支持随时间监视和分析应用程序日志与性能，开发者可以根据生产数据和标注持续改进提示词、数据集和模型。

这一层的价值逻辑是：**AI 应用不是"开发完就结束"，而是需要持续迭代**。标注回复（人工编辑高质量答案覆盖 AI 回答）、Prompt IDE（可视化调试和多模型对比）、内容审查（敏感词和安全过滤）共同构成了一个运营闭环。

### 五、发布集成层：BaaS 策略的体现

Dify 提供基于 API 的开发方式（后端即服务），可以直接访问网页应用，也可以接入 API 集成到现有应用中，无需关注复杂的后端架构和部署过程。 [Gitee](https://gitee.com/dify_ai)

2025 年还新增了 Trigger 触发器，作为事件驱动型任务调度组件。 [53AI](https://www.53ai.com/news/dify/2025112961328.html)这意味着 Dify 应用不再只能被动响应用户输入，还可以由定时任务、Webhook 事件、外部信号主动触发。

### 六、技术架构

Dify 采用模块化的微服务架构，各功能组件可独立部署、水平扩展。本地版通过 Docker Compose 一键启动，插件在安全沙箱中执行。生产环境推荐 Kubernetes 部署，配合 PostgreSQL、Redis 和向量数据库。
### 七、从 PM 视角看 Dify 的产品战略

| 维度   | 判断                                                                         |
| ---- | -------------------------------------------------------------------------- |
| 目标用户 | 开发者和创业团队为主，非技术用户上手有门槛                                                      |
| 核心价值 | 把构建 AI 应用的通用基础设施开源，降低重复造轮子成本                                               |
| 商业模式 | 开源社区引流 → 云服务收费 + 企业版授权                                                     |
| 护城河  | 插件生态 + 社区 + 部署便利性，而非模型本身                                                   |
| 局限性  | Dify 内置了代码块、格式化、逻辑判断等功能，但对于非技术员工学习成本较高；拖拽式编排难以降低 token 消耗，直接用在生产环境成本居高不下。  |
| 竞争格局 | 对个人开发者和 POC 场景是首选；有完整研发团队的企业更倾向用 LangChain 等框架自建                           |




# Dify具体功能模块

### 一、模型管理（核心设计思想：Provider + Model 二层插件体系）

核心架构：
![[../Pictures/Pasted image 20260315165732.png]]

Dify 把模型管理拆成两个层次：**供应商（Provider）** 管资质和凭据，**模型（Model）** 管具体的调用行为。这个分离让系统非常灵活——同一个供应商可以挂载几十个不同能力的模型，而凭据只需配置一次。

供应商支持三种模型配置方式：预定义模型（predefined-model），用户只需配置统一的供应商凭据即可使用供应商下的预定义模型；自定义模型（customizable-model），用户需要为每个模型单独配置凭据，如 Xinference，它同时支持 LLM 和 Text Embedding，但每个模型都有唯一的 model_uid，需要分别配置；从远程获取（fetch-from-remote），只需配置统一供应商凭据，模型列表通过凭据信息从供应商远程拉取，如 OpenAI 的微调模型，配置一个 api_key 即可让 Dify Runtime 获取到开发者所有的微调模型。这三种方式支持共存。

这套架构本质上是一个**开放的模型接入标准**——只要写一个 YAML + 一个 Python 类，任何新厂商都能以插件形式接入，既不需要改动 Dify 核心代码，也不影响已有供应商。

这解释了为什么 Dify 能支持 200+ 模型：不是因为 Dify 团队一家一家手写集成，而是定义了一套规范，让社区开发者和厂商自己来贡献插件，Dify 只维护标准和审核机制。这正是它作为开源项目能快速扩展模型生态的核心原因。


# 02 工具管理






###  知识库管理

架构拆解

先看整体流程全景，再拆每个格式的具体解析逻辑。整体流程清晰后，现在深入每个阶段，重点拆解各格式的解析逻辑。

![[../Pictures/Pasted image 20260315164603.png]]

### 一、ETL 引擎：两套解析体系可切换

这是整个知识库架构最关键的设计决策。Dify 提供两套 ETL 引擎，通过环境变量 `ETL_TYPE` 切换：

**内置 ETL**（默认）：用 Python 原生库直接解析，轻量、无需外部依赖，适合大多数场景。**Unstructured ETL**：接入 `unstructured.io` 外部服务，擅长复杂版式识别和表格结构保留，但需要自行部署或调用付费 API。

内置 ETL 和 Unstructured ETL 对 PDF 使用了不同的提取器。`UnstructuredPDFExtractor` 专为图片型 PDF 设计，通过 API 调用或直接处理，能按标题分块提取内容，返回 Document 对象列表。而普通 `PdfExtractor` 对两种 ETL 类型都适用，主要处理文字型 PDF。

Dify 当前对 HTML 的提取会剥离所有标签（包括表格），默认 PDF 提取器只输出纯文本。只有在使用 Unstructured ETL 时，才有可能部分保留表格结构，但效果因文档格式而异。这是当前知识库模块最明显的局限之一，对表格密集型文档影响较大。

---

### 二、五种文件格式的解析逻辑详解现在逐格式深入讲解。

![[../Pictures/Pasted image 20260315164649.png]]

### 三、Word（.docx / .doc）

`python-docx` 库将 `.docx` 文件视为一组有序的 `paragraph` 和 `table` 对象。解析时按文档顺序遍历，提取每个段落的 `.text` 属性并拼接。遇到 `table` 对象时，逐行逐格读取单元格文本，转换为 Markdown 表格格式输出，格式为 `| cell1 | cell2 |` 并加上分隔行 `| --- | --- |`。

`.doc` 旧格式（Word 97-2003）的内部结构完全不同，`python-docx` 无法处理，需要调用系统命令行工具 `antiword` 进行转换，这意味着生产环境的 Docker 镜像必须预装该工具。图片、批注、页眉页脚、艺术字全部被忽略——这对以图文混排为主的文档影响很大。

---

### 四、Excel（.xlsx / .xls）

`openpyxl` 按工作簿 → 工作表 → 行 → 单元格的层级遍历。每个 Sheet 的数据被提取为一段文本，Sheet 名称作为段落标识。单元格值调用 `.value` 属性，公式单元格只读取计算结果值，**不读取公式本身**，这通常是正确行为，但如果公式结果是 `#REF!` 之类的错误值，会被原样写入文本。

合并单元格是个陷阱：`openpyxl` 对合并区域只在左上角格有值，其余格返回 `None`，所以输出结果会出现大量空列，语义上产生歧义。对于以表格为核心内容的 Excel（如财务报表、排班表），建议先在 Excel 中手动取消合并并填充值，或者先导出为 CSV 再上传。

---

### 五、PPT（.pptx / .ppt）

`python-pptx` 将幻灯片视为 `Shape` 对象的集合。每个 Shape 可以是文本框、表格、图表、图片等不同类型。解析时遍历每张幻灯片的所有 Shape，对有 `text_frame` 属性的 Shape 提取文本，按 Shape 在幻灯片上的位置顺序（从上到下、从左到右）拼接。

每张幻灯片的内容作为一个独立段落，幻灯片编号作为分隔标记，这是一个合理的默认分段策略——每页幻灯片通常表达一个完整的观点。演讲者备注（Notes）也可以被提取，包含了大量口述内容，有时比正文更有价值。图表、SmartArt、嵌入视频的文字内容无法提取。`.ppt` 旧格式同样需要依赖 LibreOffice 进行格式转换。

---

### 六、PDF

这是解析难度最高、效果差异最大的格式。

**内置 ETL 路径**：使用 `pypdf` 库，调用每页的 `extract_text()` 方法。对于**文字型 PDF**（文本以字符流形式存在于文件内），能直接提取，但多栏排版、文字环绕等复杂布局会导致文本顺序混乱。对于**扫描型 PDF**（本质是图片），`pypdf` 提取结果为空字符串，这是最常见的"上传了文件但知识库没有内容"的根本原因。

**Unstructured ETL 路径**：`UnstructuredPDFExtractor` 专为处理图片型 PDF 设计，在 API 可用时通过 API 调用，否则直接处理文件，能按标题进行内容分块，返回 Document 对象列表。这条路径能处理扫描件（借助 OCR），也能更好地识别表格结构，但需要自行部署 `unstructured.io` 服务。

对于需要高质量提取表格的场景，建议在上传 Dify 之前，先用 `tabula-py`、`camelot` 或 `unstructured.io` 等工具在外部预处理，将表格转为 CSV 或 Markdown 格式，再上传结构化后的文件。

---

### 七、Markdown

`.txt`、`.markdown`、`.md`、`.html`、`.htm`、`.xml` 等纯文本格式直接用 `_extract_text_from_plain_text()` 处理——即直接 UTF-8 读取文件字节流，没有任何解析层。

这意味着 Markdown 文件中的所有标记（`#`、`**`、`-`、表格 `|`）都会原样保留在文本中传给分段器。这反而是优势：分段器可以用 `\n#` 作为天然的语义分隔符，按标题切分；Markdown 表格语法本身就是结构化的，不存在"表格丢失"的问题；代码块的缩进和换行也会完整保留。

这就是为什么业内普遍的建议是：**如果可以选择文件格式，优先上传 Markdown**。把 Word、PDF 预处理成 Markdown 再导入，是提升 RAG 质量最简单有效的手段。

## Dify 知识库分块机制完整拆解

分块是 RAG 质量的核心变量——切得太大，检索噪声多；切得太小，上下文断裂。Dify 提供了从底层算法到产品策略的完整解决方案。

先看整体的分块决策树：现在逐层拆解每个分块策略的内部机制。

![[../Pictures/Pasted image 20260315165520.png]]

### 一、底层算法：两种 Splitter

Dify 实际使用两种分片策略，都在 `api/core/rag/splitter` 中实现：`EnhancedRecursiveCharacterTextSplitter`（递归查看字符分割文本，支持基于特定分隔符进行分割，类似 LangChain 方案）；以及 `FixedRecursiveCharacterTextSplitter`（同样是递归字符分割，但支持用户指定分段标识符和分段最大长度等信息）。自动分段对应 `EnhancedRecursiveCharacterTextSplitter`，自定义分段对应 `FixedRecursiveCharacterTextSplitter`。

两者的**核心算法相同**，都是递归分割——差别只在于参数是系统内定还是用户可配。

---

### 二、递归分割算法的内部执行逻辑

这是理解整个分块机制的关键。递归分割不是简单地按固定字符数截断，而是按**优先级降序**尝试语义分隔符：递归分割的核心逻辑是：一旦找到第一个能匹配的分隔符，就用它分割文本，并将后续分隔符列表更新为剩余的列表，用于对过长子块的递归分割。然后检查每个切分后的文本块长度——短于 `chunk_size` 的直接保留，长于 `chunk_size` 的继续用下一级分隔符递归处理。

长度计量用的是 GPT-2 分词器（BPE），通过对文本编码计算 token 数，`merges.txt` 文件包含 BPE 合并规则，`vocab.json` 是词汇表，`special_tokens_map.json` 定义特殊标记。这意味着无论你接入的是哪家厂商的模型，Dify 分块时都统一用 GPT-2 的分词方式计算长度，这是一种实用的工程妥协——避免对每个供应商维护不同的 tokenizer。

![[../Pictures/Pasted image 20260315165606.png]]

### 三、通用模式的三种子策略

**自动分段**：`EnhancedRecursiveCharacterTextSplitter` 自动推断最佳分隔符组合，用户无需配置任何参数。适合快速上手，对大多数普通文档效果合理。

**自定义分段**：`FixedRecursiveCharacterTextSplitter`，用户手动设置分段标识符（默认 `\n`，可用正则表达式自定义，如 `\n\n` 按段落）、分段最大长度（默认 500 Tokens，上限 4000 Tokens）以及重叠长度。是生产环境调优的主要手段。

**Q&A 分段模式**：通过 LLM 为每段文本生成问答对（Q&A pairs），检索时匹配用户问题与预生成的相似问题，返回对应答案段落。这个策略的核心洞察是：用户提问和文档原文的语义空间往往不同（用户问"怎么退款"，原文写"退款流程如下"），而问题匹配问题的语义相似度远高于问题匹配段落。代价是需要消耗大量 Token 预处理。

---

### 四、父子模式：解决通用分段的两大矛盾

通用分段有两种对立的不利情况：检索结果精准但分散，没有完整上下文；检索结果上下文够了但不够精准。父子模式用双层结构同时解决这两个问题：子块精准匹配查询（拆得很细，一句话就是一个块），父块提供上下文（然后根据匹配到的子块找到对应的父级，补充完整信息）。

父子分段支持两种父文档模式：段落模式下根据预设分隔符规则和最大长度将文本拆分为段落，每个段落视为父分段，适用于内容清晰且段落相对独立的大文档；全文模式下直接将全文视为单一父分段，出于性能原因仅保留前 10000 Tokens，适用于文本量较小但段落间互有关联、需完整检索全文的场景。

在源码实现中，父子分段通过继承和模板方法模式实现。PARAGRAPH 模式先用父分段规则切块，再对每个父块用子分段规则生成子块，构建层级文档结构；FULL_DOC 模式则直接处理全文生成子块。父子分段在索引时只对子块进行向量化存储，检索时通过子块找到父块，将父块内容返回给 LLM。

---

### 五、关键参数调优建议

不同文档类型的参数推荐：技术文档建议 1200 字符最大长度和 100 字符重叠，博客建议 1500 字符和 50 字符重叠，FAQ 文档建议 800 字符并启用 Q&A 分段。

还有一个重要的工程细节：父子模式下同样的文件处理时间约为通用模式的 4 倍（约 20 分钟 vs 5 分钟），原因是子块数量更多。在算力不足的私有化部署场景下，父子分段可能导致任务堆积，应谨慎评估服务器配置后再决定是否使用。

分段模式还有一个产品层面的硬约束值得关注：分段模式一旦选定后续无法变更，知识库内新增的文档也将遵循同样的分段模式。这意味着知识库创建时的模式选择是一个重要的架构决策，建议在正式上传大量文档前先用少量文档测试分段效果。



# 04 工作室管理



## 0401 工作流




## 0402 ChatFlow



# Agent

![[../Pictures/Pasted image 20260315170829.png]]
### 一、入口与预检：请求到执行前做了什么

调用 `/console/api/apps/xxx/chat-messages` 接口后，触发 `AgentChatAppGenerator.generate` 方法，`app_model.mode` 为 `'agent-chat'`。整个 `_generate_worker` 预检流程依次执行：Token 数量是否足够 → 组织 Prompt → 敏感词检测 → 标注回复匹配 → 填写来自外部数据工具的变量输入 → 再次重新组织 Prompt → 再次敏感词检测 → 加载工具变量 → 初始化 model 实例。

预检的顺序设计很精妙：敏感词检测放在工具调用之前，避免不合规请求消耗工具调用资源；标注回复匹配放在模型调用之前，命中后直接返回预设答案，完全跳过 LLM 推理，节省 Token。

---

### 二、策略选择：Function Calling vs ReAct，谁来决定

根据 LLM 模型确认使用 `FUNCTION_CALLING` 模式还是 `CHAIN_OF_THOUGHT`（CoT + ReAct）：如果模型支持 `MULTI_TOOL_CALL` 或 `TOOL_CALL`，则使用 `FUNCTION_CALLING`；否则使用 `CHAIN_OF_THOUGHT` 模式。确定模式后，再根据 LLM 本身的工作模式分配具体 Runner：`CHAIN_OF_THOUGHT` + Chat 模式 → `CotChatAgentRunner`；`CHAIN_OF_THOUGHT` + Completion 模式 → `CotCompletionAgentRunner`；`FUNCTION_CALLING` 模式 → `FunctionCallAgentRunner`。

这个自动选择机制是 Dify Agent 最重要的工程设计之一——开发者无需关心"这个模型支不支持 Function Calling"，Dify 在运行时自动适配。当你在界面把模型从 GPT-4 换成 DeepSeek，后台会悄悄从 `FunctionCallAgentRunner` 切换到 `CotChatAgentRunner`，行为模式不同，但对用户透明。

---

### 三、两种推理范式的内部执行逻辑

接下来是整个 Agent 架构最核心的部分，用一张图对比两种范式的执行差异：**Function Calling 模式（`FunctionCallAgentRunner`）**![[../Pictures/Pasted image 20260315170858.png]]

`FunctionCallAgentRunner.run()` 的核心是一个 `while function_call_state` 循环，直到步数达到 `max_iteration_steps` 或 `function_call_state` 为 False 时退出。每轮循环依次执行：创建本轮思考记录 → 组织 Prompt（`_organize_prompt_messages()`）→ 计算 LLM 最大 Token 限制（`recalc_llm_max_tokens()`）→ 调用 `model_instance.invoke_llm()` 得到流式 chunks → 遍历 chunks 判断是否有工具调用指令。如果没有选择出工具，则大模型返回什么就 yield 什么；如果选择出了工具，就循环调用每个工具，然后将工具结果追加到 Prompt 继续下一轮循环。

**ReAct 模式（`CotChatAgentRunner`）**

`CotAgentRunner` 的迭代循环与上述大致一致，只是没有结构化的 `tool_call`，而是判断是否有 `action`。如果没有 action，直接返回最终答案；否则判断 action 是否为 `final_answer`，如果不是，则去调用每一个 action，然后重复上述步骤直到达到 `max_iteration_steps`。ReAct 提示模板让 LLM 按 `Thought: → Action: → Action Input: → Observation:` 格式输出，`CotAgentOutputParser` 用正则表达式解析这些字段，提取工具名和参数。

---

### 四、BaseAgentRunner：五大模块的协同设计

`BaseAgentRunner` 的核心职责拆解为五个模块：初始化模块（接收租户ID、对话实例、模型配置、用户消息等参数）；工具管理模块（`_convert_tool_to_prompt_message_tool` 将工具转换为模型可识别的 `PromptMessageTool` 格式，包含工具名称、描述、参数 schema）；历史消息模块（`organize_agent_history` 整理历史对话与代理思考记录构建上下文）；思考记录模块（`create_agent_thought` / `save_agent_thought` 创建和保存每步思考过程包括调用工具的输入输出和 LLM 用量）；模型交互模块（检查模型是否支持流式工具调用 `stream_tool_call` 和视觉能力 `ModelFeature.VISION`）。

其中工具转换的细节尤其重要——工具必须被翻译成模型能理解的格式才能被正确调用：

```python
# 工具被转换为 PromptMessageTool 格式
message_tool = PromptMessageTool(
    name="dataset_search",
    description="查询指定时间范围的产品销量数据",
    parameters={
        "type": "object",
        "properties": {
            "start_date": {"type": "string", "description": "开始日期 YYYY-MM-DD"},
            "end_date":   {"type": "string", "description": "结束日期 YYYY-MM-DD"}
        },
        "required": ["start_date", "end_date"]
    }
)
```

---

### 五、工具体系：50+ 工具的分类与接入机制

Dify Agent 支持的工具分为三类：内置工具（Dify 官方维护，开箱即用，如 Google Search、DALL-E、Wikipedia、Stable Diffusion 等）；自定义工具（用户通过 OpenAPI / Swagger 规范自定义导入）；知识库检索工具（`DatasetRetrieverTool`，自动将知识库检索能力以工具形式暴露给 Agent）。

v1.0 之后，Agent 策略本身也变成了可插拔的插件，`strategies/` 目录存放具体的策略实现。官方实现的 Agent 策略插件包括 Function Calling 和 Reason+ReAct 两种，开发者可以通过继承基类开发自定义策略插件，实现不同的推理和工具调用逻辑。

---

### 六、AgentThought：让推理过程对用户可见

这是 Dify Agent 体验层最有价值的设计。每次迭代开始时，系统都会在数据库创建一条 `AgentThought` 记录，通过 SSE 实时推送到前端。用户在界面上看到的"思考中…调用工具…获得结果…"的动态展示，就来自这个机制。

每次工具调用的完整链路都被记录：工具名称、输入参数、执行结果、耗时、以及本轮 LLM Token 用量，这些数据既用于前端实时展示，也写入日志系统供后续 LLMOps 分析。

---

### 七、安全机制：防止无限循环

`max_iteration_steps = min(app_config.agent.max_iteration, 99) + 1`——最大迭代次数上限硬编码为 99 次，防止模型陷入工具调用死循环消耗大量 Token。在最后一次迭代时，系统会强制清空工具列表（`prompt_messages_tools = []`），迫使 LLM 在没有工具可用的情况下直接生成最终答案，而不是继续尝试工具调用。

当 Agent 匹配不到对应工具时，不同大模型表现不同——假如使用了三个工具，但只匹配到一个，就使用这个工具；如果一个都没有匹配到，则直接返回大模型的原始输出，不会报错中断。

---

### 八、v1.0 之后的架构变化：策略插件化

v1.0 是一个重要分水岭。在 Dify v1.0 中，Agent 策略从硬编码的 Runner 类升级为可插拔插件，`strategy/plugin.py` 负责加载外部策略插件。这意味着开发者可以在不修改 Dify 核心代码的前提下，实现自定义推理策略——比如 Tree of Thought、Self-Consistency 采样、或者特定领域的 Domain-specific Agent 策略——只需按插件规范实现并注册即可。

这个变化将 Agent 的"推理大脑"也变成了开放的插件市场，延续了 Dify 整体的"核心框架 + 生态插件"战略。

## Dify Agent 多轮对话与上下文记忆机制完整拆解

这个问题触及一个根本矛盾：**LLM 本身是无状态的**，每次 API 调用都从零开始，完全不记得上一次说了什么。Dify 的记忆机制本质上是在无状态的模型上方，工程化地模拟出"有记忆"的体验。

先看整体记忆架构：现在逐层深入每个环节。

![[../Pictures/Pasted image 20260315172042.png]]

### 一、根本原理：每次调用都重新"喂"历史

这是理解一切的前提。LLM 没有记忆，Dify 的做法是每次用户发新消息时，把**数据库里存储的历史对话全部取出来，拼进这次的 Prompt 一起发给 LLM**。LLM 看到了历史，就好像"记得"一样——但实际上它只是在处理一个很长的输入。

用一个简单类比：LLM 是一个每次都会失忆的人，Dify 相当于每次跟他说话前，先把他的日记本读给他听。

---

### 二、conversation_id：记忆的容器

每个新会话在 PostgreSQL 的 `conversations` 表中生成一条记录，分配唯一 UUID 作为 `conversation_id`。同一个会话内的所有消息都关联这个 ID。

调用 API 时，`conversation_id` 的传递方式决定了记忆的归属：

```json
// 第一轮（不传 conversation_id，系统新建会话）
{ "query": "我叫张三", "conversation_id": "" }

// 返回中会包含新生成的 conversation_id
{ "conversation_id": "abc-123-xyz" }

// 第二轮（传入同一个 conversation_id，接续上下文）
{ "query": "你还记得我叫什么吗？", "conversation_id": "abc-123-xyz" }
```

如果新建一个对话（不传 `conversation_id` 或传空字符串），上一次会话的记忆完全不会被载入。这正是社区里有人发现"新建会话后 Agent 忘记了之前的名字"的根本原因——这是设计行为，不是 Bug。

---

### 三、organize_agent_history：历史组织的核心方法

`_organize_prompt_messages` 整合上下文（系统指令、历史对话、用户当前查询），确保 LLM 理解完整场景；调用 LLM 时传入 `tools` 参数，明确告知可用工具的结构，引导 LLM 生成规范的工具调用指令。

历史消息被组装成标准的 messages 数组，结构如下——这个数组完整还原了"对话的时间线"：

```python
[
  SystemMessage("你是一个智能助手，可使用以下工具：..."),
  HumanMessage("我叫张三，帮我查一下明天的天气"),      # 第1轮 user
  AIMessage("好的，正在查询... [工具调用: weather]"),   # 第1轮 assistant + AgentThought
  ToolMessage("明天晴，25°C"),                          # 工具返回结果
  AIMessage("明天天气晴朗，气温25°C，适合出行"),         # 第1轮最终回答
  HumanMessage("那我需要带伞吗？"),                     # 第2轮 user（当前）
]
```

值得注意的是，**AgentThought（工具调用的思考链）也会被包含在历史里**。这意味着 LLM 在第二轮不仅知道用户问了什么、自己答了什么，还知道上一轮调用了什么工具、得到了什么结果。这使得多轮对话中的工具调用可以基于前轮的工具结果继续推理。

---

### 四、TokenBufferMemory：窗口大小与截断机制

Dify 使用 `TokenBufferMemory` 来缓冲最近对话消息，当启用记忆后自动将这段历史加入每次 LLM 调用。系统后端硬编码了上限：最多 2000 Tokens、500 条消息，无论你怎么配置都不会超过这个边界，以防止失控的成本。

Token 截断的核心公式是：可用提示 Token 数 = 模型最大上下文 − 用户定义的输出 Token 数。如果当前上下文的 Token 计数超过可用提示 Token 数，Dify 会智能截断——对于聊天应用，这通常涉及从对话历史中删除最旧的消息，直到总 Token 计数适合预算。

截断策略遵循两个优先级原则：System Message（系统指令 + 工具列表）永远不被截断，因为没有它 Agent 不知道自己是谁、有什么工具可用；历史消息按从旧到新的顺序删除，保留最近的对话轮次。

Window Size 参数控制 Agent 能"记住"多少轮之前的对话历史，对于需要进行多轮连贯对话的 Agent 至关重要，使其能够理解代词指代和上下文关联。

---

### 五、AgentThought 如何融入历史

这是 Agent 模式和普通 Chatbot 模式在记忆上最大的区别。普通 Chatbot 的历史只有 `user/assistant` 两种角色的消息；Agent 的历史还包含**工具调用的完整链路**：

每次迭代创建 `agent_thought` 记录，用于存储本轮的思考、工具调用、结果等信息（可追溯）。`create_agent_thought` 方法在数据库中创建 `MessageAgentThought` 记录，包含工具名称、输入参数、位置序号等；`save_agent_thought` 方法在工具调用完成后更新记录，补充工具返回结果（observation）、记录 LLM 的 Token 使用量，并序列化工具输入输出为 JSON 字符串。

这些 AgentThought 在下一轮对话开始时会被重新读取并拼入历史，让 LLM 在新一轮推理时能看到完整的"上一轮我做了什么"，从而实现跨轮的工具调用连续性。

---

### 六、长期记忆：跨会话的两种实现方式

上面描述的短期记忆在新建会话后全部清零。要实现跨会话的长期记忆，Dify 提供两种机制：

**知识库向量检索**：将用户信息、偏好、历史结论写入知识库，每次对话时检索相关内容注入 System Prompt。这是被动的——系统自动检索相关记忆，不需要开发者手动管理。

**会话变量（Conversation Variables）**：在 Chatflow 中创建"对话变量 `Conver_History`"数组存储每轮对话，Agent 回复后通过"模板转换"和"变量赋值"节点更新此历史记录，为 Agent 提供完整的对话上下文。这是主动的——开发者在工作流里显式地读写变量，控制哪些信息需要被跨会话保留。

两种方式的本质区别：向量检索适合"从海量历史中找相关片段"，会话变量适合"精确控制某些状态必须被记住"。

---

### 七、一个完整的三轮对话发生了什么

用一个具体例子把所有机制串起来：

```
第1轮：用户说"我叫张三，帮我查北京明天天气"
  → Dify 新建 conversation (id=abc)
  → 组装 Prompt: [System] + [User: 我叫张三...]
  → Agent 调用天气工具 → 写入 AgentThought
  → 回答"明天晴25°C" → 写入 Message 表

第2轮：用户说"那我需要带伞吗？"(同一 conversation_id)
  → 读取 conversation abc 的历史：第1轮 user+assistant+thought
  → 组装 Prompt: [System] + [历史] + [User: 那我需要带伞吗？]
  → LLM 看到历史，知道上一轮查的是北京明天晴，直接回答"不需要带伞"
  → 写入 Message 表

第3轮：用户说"帮我把这个天气发邮件给我同事"(同一 conversation_id)
  → 读取历史：两轮完整记录
  → LLM 知道天气内容是什么，调用邮件工具，直接引用上下文中的天气数据
```

如果第3轮新建一个会话，LLM 就不知道"这个天气"指的是什么，因为历史没有被载入。