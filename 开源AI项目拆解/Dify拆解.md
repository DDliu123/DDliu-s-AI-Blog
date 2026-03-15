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
[[../dify_model_management_architecture.svg]]

Dify 把模型管理拆成两个层次：**供应商（Provider）** 管资质和凭据，**模型（Model）** 管具体的调用行为。这个分离让系统非常灵活——同一个供应商可以挂载几十个不同能力的模型，而凭据只需配置一次。

供应商支持三种模型配置方式：预定义模型（predefined-model），用户只需配置统一的供应商凭据即可使用供应商下的预定义模型；自定义模型（customizable-model），用户需要为每个模型单独配置凭据，如 Xinference，它同时支持 LLM 和 Text Embedding，但每个模型都有唯一的 model_uid，需要分别配置；从远程获取（fetch-from-remote），只需配置统一供应商凭据，模型列表通过凭据信息从供应商远程拉取，如 OpenAI 的微调模型，配置一个 api_key 即可让 Dify Runtime 获取到开发者所有的微调模型。这三种方式支持共存。

这套架构本质上是一个**开放的模型接入标准**——只要写一个 YAML + 一个 Python 类，任何新厂商都能以插件形式接入，既不需要改动 Dify 核心代码，也不影响已有供应商。

这解释了为什么 Dify 能支持 200+ 模型：不是因为 Dify 团队一家一家手写集成，而是定义了一套规范，让社区开发者和厂商自己来贡献插件，Dify 只维护标准和审核机制。这正是它作为开源项目能快速扩展模型生态的核心原因。


# 02 工具管理






# 知识库管理

整体流程遵循「提取 → 转换 → 加载」三大阶段，由 IndexingRunner 统一编排，按格式分发至对应提取器与索引处理器执行。

## 一、整体编排：IndexingRunner（核心入口）

- 核心逻辑：通过 `run()` 方法依次触发提取、转换、片段持久化、嵌入加载全流程；
    
- 提取委派：`_extract()` 方法判断数据源类型（上传文件/Notion 导入/网页抓取），构建 ExtractSetting 配置，将任务委派给匹配的 IndexProcessor。
    

## 二、格式分发：ExtractProcessor（中心路由）

- 核心作用：根据文件后缀 + ETL_TYPE 配置，路由到对应格式的提取器；
    
- 两种解析模式：
    
    - 默认模式：使用 Python 内置库完成解析；
        
    - 非结构化模式（ETL_TYPE="Unstructured"）：指定格式（.doc/.ppt/.epub 等）路由到 Unstructured API 提取器；Markdown 文件在 is_automatic=True 时，使用 UnstructuredMarkdownExtractor；
        
- 接口规范：所有提取器均实现 BaseExtractor 统一接口。
    

## 三、各格式文档提取核心逻辑

|格式|提取器类|依赖工具/库|ETL 模式|核心实现原理|输出粒度|
|---|---|---|---|---|---|
|.docx|WordExtractor|python-docx|两种均支持|遍历段落/表格并保序；表格转 Markdown；提取内嵌图片生成 Markdown 链接；处理超链接为 Markdown 格式|1 个 Document（全文件内容）|
|.doc|UnstructuredWordExtractor|Unstructured API|仅非结构化|基于标题分片（chunk_by_title），按最大字符限制拆分，自动生成多段内容|N 个 Document（按标题分片）|
|.xlsx|ExcelExtractor|openpyxl|两种均支持|扫描前10行启发式识别表头；每行转为「列名:值;列名:值」格式；单元格超链接转 Markdown 链接|每行 1 个 Document|
|.xls|ExcelExtractor|pandas/xlrd|两种均支持|读取后删除全空行；其余逻辑同 .xlsx|每行 1 个 Document|
|.pdf|PdfExtractor|pypdfium2|两种均支持|先检查缓存；逐页提取文本；识别内嵌图片格式（JPEG/PNG 等）并保存，追加 Markdown 图片链接|每页 1 个 Document|
|.md/.mdx（默认）|MarkdownExtractor|Python 标准库|仅默认|自动检测编码读取；按标题（^#+\s）拆分，识别代码块；保留章节结构|每标题章节 1 个 Document|
|.md/.mdx（自动）|UnstructuredMarkdownExtractor|Unstructured|非结构化+自动|调用 Unstructured API/本地 partition_md；按标题分片|N 个 Document（按标题分片）|
|.pptx|UnstructuredPPTXExtractor|unstructured.pptx|两种均支持|按幻灯片页码分组文本|每幻灯片 1 个 Document|
|.ppt|UnstructuredPPTExtractor|Unstructured API|仅非结构化|需配置 API 地址（否则抛错）；按页码分组文本|每幻灯片 1 个 Document|

## 四、索引处理器匹配：IndexProcessorFactory

核心逻辑：将 doc_form 标识映射为对应处理器实例，实现不同分片策略的分发：

- "text_model" → ParagraphIndexProcessor（通用扁平分片）；
    
- "qa_model" → QAIndexProcessor；
    
- "hierarchical_model" → ParentChildIndexProcessor（层级分片）。
    

## 五、转换阶段（清洗 + 分片）

### 5.1 文本清洗：CleanProcessor

- 核心方法：`clean()` 移除控制字符；
    
- 可选能力：去除多余空格、URL/邮箱（保留 Markdown 链接/图片占位符）。
    

### 5.2 文本分片：BaseIndexProcessor._get_splitter()

根据处理模式创建不同分片器：

- 自定义/层级模式：FixedRecursiveCharacterTextSplitter（先按用户指定分隔符拆分，再递归按「\n\n、。、. 、 、空字符」拆分）；
    
- 自动模式：EnhanceRecursiveCharacterTextSplitter（使用默认配置：max_tokens=500，chunk_overlap=50）。
    

### 5.3 分片处理器核心逻辑

- ParagraphIndexProcessor：清洗文档 → 分片为节点 → 分配 UUID/哈希值 → 提取内嵌图片附件；
    
- ParentChildIndexProcessor（层级分片）：
    
    - "paragraph" 模式：先拆父分片 → 再拆分子节点；
        
    - "full_doc" 模式：合并全文本为父文档 → 仅拆分子节点；子节点以 ChildDocument 存储在父文档的 children 属性中。
        

## 六、片段持久化：DatasetDocumentStore

核心方法：`add_documents()` 创建数据库记录：

- 基础分片：生成 DocumentSegment 记录（含 index_node_id、哈希、内容、位置、token 数）；
    
- 层级分片：额外创建 ChildChunk 记录。
    

## 七、嵌入与加载：IndexingRunner._load()

- 高质量索引：通过 ThreadPoolExecutor（最多10线程）按哈希分发文档，调用 `Vector.create()` 完成文本嵌入并写入向量库（父子模式仅子文档嵌入）；
    
- 轻量索引：独立线程调用 `Keyword.create()`，仅生成关键词索引。



# 04 工作室管理



## 0401 工作流




## 0402 ChatFlow



## 0403 Agent


