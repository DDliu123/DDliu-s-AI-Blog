得物技术：分块策略https://mp.weixin.qq.com/s/kvOnakKVr8xVrrC32HlhIg

## RAG需要分块的主要原因有以下两个：

1. ### Embedding长度限制
    

大多数文本嵌入模型（如OpenAl的text-embedding-3-small,或Hugging Face的all-MiniLM-L6-V2)都对输入文本长度有严格限制。

如果不分块，整个文档太长，无法直接生成embedding,导致无法进行向量化检索。

2. ### 提高语义相关性和精度
    

RAG的本质是：

> 通过用户问题去和“语义向量化后的内容块”做相似度匹配。

- 若文本片段划分过大：
    

易出现多主题混杂的问题，导致内容匹配精准度下降；即便片段中仅部分内容与目标相关，也会因整体区块的冗余信息产生干扰，不仅使得向量难以精准捕捉内容的特定细节，还会额外增加计算成本。

- 若文本片段划分过小：
    

则可能丢失上下文信息，导致句子碎片化和语义不连贯。

- 较小的块适用于需要细粒度分析的任务，例如情感分析，能够精确捕捉特定短语或句子的细节。
    
- 更大的块则更为合适需要保留更广泛上下文的场景，例如文档摘要或主题检测。
    

## 分块的三个关键组成部分：

多种分块策略从本质上来看，由以下三个关键组成部分构成：

- 大小：每个文档块所允许的最大字符数。
    
- 重叠：在相邻数据块之间，重叠字符的数量。
    
- 拆分：通过段落边界、分隔符、标记，或语义边界来确定块边界的位置。
    

  

## 五种常见的分块策略：

选择合适的分块策略，是解决RAG实际应用中准确性、召回率与复杂文档解析等痛点最直接有效的方式，也是我们建设RAG系统最关键的一个环节。 最常见的RAG分块策略包括： **固定大小分块、语义分块、****递归****分块、基于文档结构的分块、基于****LLM****的分块** 。

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=MDZhNzA0YmM4YjYxNDc4MWE2OWYwZGZjMTY4NGUzMWNfc0Q4dTQ5aXlxeG5xNmNLRzRqR2dSZHpxVGUycDNxVzNfVG9rZW46RDZnbWJEd28zb0pOSVR4Q3lTRWNCdjdUbkZkXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

RAG五种分块策略（图片来源： DailyDoseofDS ）

下面我们围绕这五种分块策略，系统介绍不同分块策略的基本原理、实现步骤、主要优缺点与适用场景。

### **固定大小分块**

#### **基本原理**

固定大小分块 （Fixed-size Chunking）， 将文本按固定长度（如字符数、单词数或token数）切分，每个块大小一致，可能通过重叠保留上下文连贯性。例如，将文档每256个字符切分为一个块，重叠20个字符以减少边界信息丢失。

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=NjU3ODIxMzQ3ZjM4NGU1MzJlNzA2ZTZiMDFkMzIxZTlfbE9iYVdqaUhzTXNkTTdWSFdSTXdDOVhPZTE2MlhBN1lfVG9rZW46TmVBbGJCdG9Db2dSNlZ4Vkh2cWNDSDdtblhiXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

固定大小分块示意 （图片来源： DailyDoseofDS ）

#### **实现步骤**

- **预设参数** ：定义块大小（如256 token）和重叠比例（如20 token）。
    
- **切分文本** ：按固定长度分割文本，允许相邻块部分重叠。
    
- **生成块列表** ：输出所有块作为独立单元。
    

#### **主要优点**

- **实现简单** ：无需复杂算法，代码实现高效。
    
- **标准化处理** ：块大小一致，便于批量处理和向量化。
    
- **资源友好** ：适合大规模文本处理，降低计算成本。
    

#### **主要缺点**

- **语义断裂** ：可能在句子或概念中间切分，破坏上下文完整性。
    
- **信息冗余** ：重叠区域可能导致重复存储和计算。
    
- **适用性受限** ：对结构化文本（如代码、技术文档）效果较差。
    

#### **适用场景**

- 非结构化文本（如新闻、博客）的初步处理。
    
- 对实时性要求高、需快速切分的场景。
    

**场景示例**

```Plaintext
[原文档]
"2023年Q3净利润同比增长5.2%（详见附录Table 7）"
[分块1] "2023年Q3净利润同比增长5.2%（详见"
[分块2] "附录Table 7）"  # 关键数据来源丢失！
```

### **语义分块**

#### **基本原理**

语义分块 （Semantic Chunking），根据句子、段落、主题等有语义内涵的单位对文档进行分段创建嵌入， 如果第一个段的嵌入与第二个段的嵌入具有较高的余弦相似度，则这两个段形成一个块。 通过合并相似内容，确保每个块表达完整的语义内容。

由于每个分块的内容更加丰富，它提高了检索准确性，让大模型产生更加连续和相关的响应。但是它依赖于一个阈值来确定余弦相似度是否显著下降，而这个阈值在不同类型文档中可能涉及不同的参数设置。

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=ODY0NjFjYzdjZGM3MGQyMDA1MzgxZmE5MjU0M2RjYWNfd0FicFVNY1laZGQxc3Y2NkI0cThLU3VEcUZReHl4T1NfVG9rZW46VjQ0S2J1TnBHb0dPUDZ4bDc0cmMzTEdrbjdLXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

语义分块流程

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=ODM4NDM5MzRiNzFmMTk3NTdiNjQ4YWY5MDdlMDZlYjFfc0ZROGNFOFdEWEZTR2lSbmdFem54ZXVXNG5oMGNwNExfVG9rZW46QW1WNWJtbFRJb2JTMGt4akdUVmNpNDlBbmJoXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

语义分块示意 （图片来源： DailyDoseofDS ）

#### **实现步骤**

- **分句/分段** ：将文本拆分为句子或段落。
    
- **生成嵌入** ：为每个单元计算向量表示。
    
- **相似度计算** ：依次比较相邻单元的余弦相似度。
    
- **动态合并** ：当相似度高于阈值时合并单元；相似度骤降时开始新块。
    

#### **主要优点**

- **语义完整性** ：保留自然语义结构，提升检索准确性。
    
- **上下文敏感** ：适应复杂逻辑关系（如因果、对比）。
    
- **生成质量** ：检索到的块更连贯，利于LLM生成精准回答。
    

#### **主要缺点**

- **计算复杂度高** ：需多次向量化计算和相似度比较。
    
- **阈值依赖** ：相似度阈值需人工调试，不同文档需不同参数。
    
- **实现门槛** ：依赖高质量嵌入模型和相似度算法。
    

#### **适用场景**

- 高精度问答系统（如法律、医疗领域）， 研究论文、行业分析报告等专业文档 。
    
- 需保留上下文逻辑的复杂文档（如论文、技术报告）。
    

**场景示例**

```Markdown
[分块]
区块1: "货币政策的宽松将推动市场流动性提升。"
区块2: "但需警惕通胀反弹带来的政策转向风险。"  
# 每个区块为完整语义单元
```

### **递归****分块**

#### **基本原理**

递归分块 （Recursive Chunking）， 先按主题或段落初步划分，再对超长块递归细分，直至满足大小限制。递归分块融合了结构化与非结构化处理逻辑， 与固定大小的分块不同，这种方法保持了语言的自然流畅性并保留了完整的内容语义。

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=MzY5OTFjYmJhNzJlMDEyZDA1NTc0ZWYyZDE0NTI4YzdfRjJzdDFObmk3RFo1ZVFxMERDNEptNXFtUFFCNFFQQUFfVG9rZW46Qm41OWJtNG1SbzFGSUR4eHJyM2MzSVp2bjdjXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

递归分块流程

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=NGE5MGEwMjliNjFhNzFkNzc4ZDg0NGQyMTQ5NmFhYTVfSU4ydGZKa2hvWDVUVEFqaGlsZ202c1ZReVFyejlYb1FfVG9rZW46RGw2dWJkY2pQb2NmanR4c3JibGNHOG54bkdnXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

递归分块示意 （图片来源： DailyDoseofDS ）

#### **实现步骤**

- 粗粒度切分：按段落、标题或主题初步划分大块。
    
- 检查大小：判断块是否超过预设长度（如1024 token）。
    
- 递归细分：超长，按固定大小或语义逻辑进一步切分。
    
- 终止条件：块大小符合要求时停止递归。
    

#### **主要优点**

- 灵活性强：平衡结构完整性与大小限制。
    
- 适应复杂内容：处理长文档（如书籍、长篇论文）时表现优异。
    
- 多策略融合：可结合固定大小或语义分块优化细分。
    

#### **主要缺点**

- 块大小不均：不同层级的块可能差异较大。
    
- 逻辑断裂风险：递归过程中可能破坏原文的自然段落结构。
    
- 实现复杂：需设计递归终止条件和分块策略。
    

#### **适用场景**

- 长文档处理（如企业年报、学术论文）， 书籍、技术手册等层级化文档 。
    
- 需兼顾结构化与非结构化内容的场景， 包含嵌套结构的合同文本 。
    

**场景示例**

```Plaintext
1.摘要 --> [保留完整]  
2.行业分析 --> [按子章节切分]
2.1 供需格局 --> [按段落切分]
2.2 竞争态势 --> [按段落切分]  
3.附录表格 --> [特殊处理]
```

### **基于文档结构的分块**

#### **基本原理**

基于文档结构分块 （Document Structure-based Chunking）， 利用文档固有结构（如标题 <h1> 、章节、列表 <ul> 、表格 <table> ）进行切分，每个结构单元作为一个块。 它通过与文档的逻辑部分对齐来保持结构完整性。这种分块适用于文档有清晰的结构，但很多时候，一个文档的结构会比想象中复杂，此外，很多时候文档章节内容大小不一，很容易超过块的大小限制，需要结合递归拆分再进行合并处理。

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWUyY2U3OTc4MzYzYjI2MGRjMjNjYWFlZjllYTJiZjRfeWg3WHJUM2VBT2ROVExSRVcwWHRMWDBLbGhhaW1jdThfVG9rZW46Qno5VmJ0Smtrb1ZDbHN4azEzU2NsMkdRblQwXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

基于文档结构分块流程

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=MDM0YmU0OTU0NGUxODE5N2RhN2UwYmJmNGFhYTBiZjRfZDNVaWZSeHBpb004TnVCSXltV3NDTk53UUtQd0c3cU9fVG9rZW46VWlGOWJqNnFPb2lOSE94M1ZXdWN4bGd0bmlnXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

基于文档结构分块示意 （图片来源： DailyDoseofDS ）

#### **实现步骤**

- **识别结构元素** ：解析文档中的标题、段落、小节等标记（如Markdown、XML）。
    
- **按结构切分** ：将每个结构单元（如“引言”、“结论”）独立为块。
    
- **处理超长部分** ：若某结构单元过大，再结合递归或固定大小分块细化。
    

#### **主要优点**

- **逻辑清晰** ：保留文档的层次化结构，便于定位信息。
    
- **检索高效** ：用户可通过标题快速定位相关内容。
    
- **格式****兼容性** ：适合结构化文档（如技术手册、报告）。
    

#### **主要缺点**

- **依赖格式标准化** ：对非结构化文本（如自由写作）效果差。
    
- **预处理复杂** ：需解析文档格式（如LaTeX、HTML），增加实现难度。
    
- **灵活性不足** ：难以处理混合结构内容（如图文混排）。
    

#### **适用场景**

- 结构化文档，如： 财报（表格数据）、技术文档（代码块）、合同（条款列表） 。
    
- 需按章节检索的场景（如法规数据库）， 任何含丰富格式标记的内容 。
    

**场景示例**

  

```Plaintext
[原始PDF表格]  

|项目|2023Q3|同比|
|--|--|--|
|营业收入|5.2亿|+12%|
|[结构化分块]|||
|{|||
|"type": "table",|||
|"title": "利润表摘要",|||
|"data": [["项目", "2023Q3", "同比"], ["营业收入", "5.2亿", "+12%"]]|||
|}  # 整表作为独立区块|||
```

### **基于****LLM****的分块**

#### **基本原理**

基于LLM的分块 （LLM-based Chunking）， 直接将原始文档输入大语言模型（LLM），由模型智能生成语义块。利用LLM的语义理解能力，动态划分文本，保证了分块语义的准确性，但这种分块方法对算力要求最高，对时效性与性能也将带来挑战。

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGEyMDkwZDE2ZDcyZGIxYTA1ZGQyOWZhYzYxYjMyN2JfVW9SM0E3aWZ4c1lrN3NEbUtYbXJiZ0lIbXFuSGk2Q0ZfVG9rZW46TXNLNGJNa0Jlb0xTcGR4c0lseWNuT2FFbkVlXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

基于LLM分块流程 （图片来源： DailyDoseofDS ）

#### **实现步骤**

- **输入文档** ：将完整文档送入LLM（如DeepSeek、GPT）。
    
- **生成块****指令** ：通过提示词（Prompt）引导模型按语义划分块。
    
- **示例提示词** ：“请将以下文档按语义划分为多个块，每个块需包含完整主题。”
    
- **输出块列表** ：模型返回划分后的块，可能包含逻辑标签（如“引言”、“方法论”）。
    

#### **主要优点**

- **高度智能化** ：适应复杂、非结构化文本（如自由写作、对话记录）。
    
- **动态适应性** ：根据文档内容自动调整块大小和逻辑。
    
- **生成质量** ：块语义连贯，减少人工干预。
    

#### **主要缺点**

- **计算成本高** ：依赖高性能LLM，资源消耗大。
    
- **可解释性差** ：模型决策过程难以追溯，可能产生不可预测的块。
    
- **依赖模型能力** ：效果受限于LLM的训练数据和语义理解能力。
    

#### **适用场景**

- 非结构化文本（如访谈记录，会议纪要，用户评论、社交媒体内容等）。
    
- 需高级语义分析的场景（如跨领域知识整合）
    

**场景示例**

```Plaintext
[原始分散段落]  
段落1: "A公司宣布收购B公司..."  
段落2: "交易金额达50亿美元..."  
段落3: "B公司核心资产为..."  
[LLM智能分块结果]  
"并购事件：A公司以50亿美元收购B公司（核心资产为...）"  # 跨段落聚合关键信息
```

**五种RAG分块策略总结对比**

|   |   |   |   |
|---|---|---|---|
|分块策略|优点|缺点|适用场景|
|固定大小分块|实现简单，资源高效|语义断裂，信息冗余|快速处理非结构化文本|
|语义分块|语义完整，检索精准|计算复杂，依赖阈值|高精度问答、复杂文档|
|递归分块|灵活适应长文档，保留结构|块大小不均，逻辑断裂风险|长篇技术文档、企业报告|
|基于结构的分块|逻辑清晰，检索高效|依赖格式标准化，预处理复杂|结构化文档（论文、白皮书）|
|基于LLM的分块|高度智能，适应非结构化文本|计算成本高，决策过程不可控|非结构化内容、跨领域整合|

---

### **RAG分块策略选择建议**

- **结合****递归****与结构分块** ：处理长文档时（如法律合同、表格、公式、技术手册）。
    
- **语义分块** ：对生成质量要求高、文档语义复杂时（如论文、医疗问答）。
    
- **使用****LLM****分块** ：处理非结构化或混合内容（如多模态文档）。
    
- **固定大小分块** ：快速部署或资源受限场景（如社交媒体、轻量级应用）。
    

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=ODVkNDFmNGFmODQ5ZDczYTljZmYyNGE0MWVhNTBjNzhfYjFjN01EZEdkeXhLV1BuUVY4eW9vclRGbTJLSDd2dTZfVG9rZW46TDhoNGJyR0tNbzV6eE94d3VwM2NKbTF6bktoXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

分块策略选择决策树

具体实施过程中，我们需要根据具体需求与文档类型选择分块策略，或组合多种方法（如“结构分块+语义细分”）以实现最佳效果。

  

  

## 特殊文档——Excel 表格分块策略：

1. ### 预处理
    

预处理是分块的基础，核心目标是解决合并单元格导致的信息缺失问题，并过滤无效数据，为后续分块提供 “干净” 的数据源。

1. #### 合并单元格的影响
    

Excel 中的合并单元格（如 “模块名称” 列合并多行）会导致解析工具（openpyxl、pandas 等）仅保留合并区域左上角单元格的值，其余单元格显示为`NaN`或空字符串。这会引发两个关键问题：

  

- 检索定位失效：行级分块丢失 “模块名称”“工作表类别” 等关键字段，无法通过业务维度（如 “商品管理模块”）筛选数据；
    
- 回答准确率下降：RAG 依赖的`metadata_filter`（元数据过滤）功能失效，LLM 无法获取完整上下文，回答准确率降低 15-25 个百分点。
    

##### 解决方案：合并单元格解除与值填充

通过`openpyxl`工具遍历合并区域，读取左上角单元格的值并填充至所有子单元格，再解除合并并保存为临时文件，确保每行数据包含完整字段信息。

##### 核心代码实现

```Python
from openpyxl import load_workbook  

def explode_merged_cells(path_in: str, path_out: str) -> None:  
    """
    解除Excel中的合并单元格，并将左上角值填充至所有子单元格
    :param path_in: 原始Excel文件路径
    :param path_out: 处理后Excel文件保存路径
    """# 加载工作簿并指定目标工作表（可根据需求修改工作表名称）
    wb = load_workbook(path_in)  
    ws = wb["核心功能"]  # 示例工作表名称，实际需替换为业务中的工作表名# 遍历所有合并区域（需用list()转换，避免遍历中修改集合）for rng in list(ws.merged_cells.ranges):  
        # 1. 解除合并单元格
        ws.unmerge_cells(str(rng))  
        # 2. 获取合并区域左上角单元格（核心值来源）
        tl_cell = ws[rng.min_row][rng.min_col - 1]  # openpyxl列索引从0开始，需减1# 3. 填充区域内所有单元格for row in range(rng.min_row, rng.max_row + 1):  
            for col in range(rng.min_col, rng.max_col + 1):  
                ws.cell(row=row, column=col, value=tl_cell.value)# 保存处理后的文件，避免修改原始数据
    wb.save(path_out)  

# 调用函数：处理“电商系统功能清单.xlsx”并保存为临时文件
explode_merged_cells("电商系统功能清单.xlsx", "处理后_功能清单.xlsx")  
```

##### 处理效果对比

|   |   |
|---|---|
|处理前（合并单元格）|处理后（填充完整）|
|行 2：用户管理（非空），行 3-4：用户管理（空）|行 2-4：用户管理（均非空）|
|行 5：商品管理（非空），行 6：商品管理（空）|行 5-6：商品管理（均非空）|
|功能 ID、描述等字段与模块名称关联断裂|每行字段与模块名称完全关联|

2. #### 无效数据过滤与结构化转换
    

- 过滤无效数据：用`pandas`删除全空行、重复行，避免分块时引入冗余信息：
    

```Python
import pandas as pd
# 读取处理后的Excel，过滤全空行
df = pd.read_excel("处理后_功能清单.xlsx", engine="openpyxl")
df = df.dropna(how="all")  # 移除全空行
df = df.drop_duplicates()  # 移除重复行
```

- 结构化转文本：将表格数据转换为 “人类可理解的文本格式”，保留字段关联关系：
    
    - 有表头表格：每行转换为 “表头 1：值 1；表头 2：值 2；...”（如 “功能 ID：USR-001；功能名称：用户注册”）；
        
    - 无表头表格：用行列索引标识（如 “第 3 行第 2 列：用户注册；第 3 行第 3 列：基础功能”）；
        
    - 保留工作表标识：每个数据块前添加 “工作表【XXX】：”，避免跨工作表信息混淆。
        

2. ### 核心分块策略：分层设计，平衡 “原子性” 与 “上下文完整性”
    

分块的核心矛盾是 “原子性”（检索时精准命中单个信息点）与 “上下文完整性”（LLM 理解信息所属业务背景）的平衡。本文整合三种思路，形成基础行级分块、进阶两级分块、特殊语义区域分块三类策略，适配不同业务场景。

1. #### 基础策略：行级分块（适配规整数据报表）
    

适用于内容规整、无复杂业务模块划分的 Excel（如员工信息表、销售明细表），核心是 “按工作表 + 固定行数” 分块，确保每个块包含完整表头，避免字段关联断裂。

1. ##### 分块逻辑
    

- 以工作表为单位：每个工作表独立分块，避免跨工作表信息混杂；
    
- 按行数分组：将连续 N 行数据作为一个块（N 根据内容密度调整，短文本行取 5-10 行，长文本行取 2-3 行）；
    
- 块内必含表头：每个块的开头重复表头信息，确保块的 “自包含性”（即使单独检索该块，也能理解字段含义）。
    

2. ##### 代码实现（基于 pandas）
    

```Python
from typing import List, Dict

def excel_basic_chunking(file_path: str, chunk_size: int = 5) -> List[Dict]:"""
    基础行级分块：按工作表+固定行数分块，保留表头
    :param file_path: Excel文件路径
    :param chunk_size: 每个块包含的最大行数（不含表头）
    :return: 分块结果列表（含内容与元数据）
    """
    chunks = []# 读取所有工作表
    xls = pd.ExcelFile(file_path)for sheet_name in xls.sheet_names:
        df = pd.read_excel(xls, sheet_name=sheet_name)
        df = df.dropna(how="all")  # 过滤全空行if df.empty:continue
        
        headers = df.columns.tolist()  # 获取表头# 将每行转换为“表头：值”格式的文本
        row_texts = []for _, row in df.iterrows():# 跳过空值字段，避免冗余
            row_str = "; ".join([f"{col}：{row[col]}" for col in headers if pd.notna(row[col])])
            row_texts.append(row_str)# 按chunk_size分块，每个块包含“工作表标识+表头+数据”for i in range(0, len(row_texts), chunk_size):
            chunk_rows = row_texts[i:i+chunk_size]# 构建块内容
            chunk_content = f"工作表【{sheet_name}】：\n"
            chunk_content += f"表头：{headers}\n"
            chunk_content += "数据：\n" + "\n".join([f"- {r}" for r in chunk_rows])# 构建元数据（便于后续定位原始位置）
            chunk_meta = {"sheet": sheet_name,"start_row": i + 2,  # Excel行索引从1开始，表头占1行"end_row": min(i + chunk_size + 1, len(row_texts) + 1),"level": "basic"}
            chunks.append({"content": chunk_content, "metadata": chunk_meta})return chunks

# 使用示例：对“员工信息表.xlsx”按5行分块
basic_chunks = excel_basic_chunking("员工信息表.xlsx", chunk_size=5)for i, chunk in enumerate(basic_chunks):print(f"基础块{i+1}：\n{chunk['content']}\n元数据：{chunk['metadata']}\n---")
```

3. ##### 分块示例（员工信息表）
    

```Plain
工作表【员工基本信息】：
表头：[姓名, 年龄, 部门, 入职时间]
数据：
- 姓名：张三；年龄：30；部门：技术部；入职时间：2022-01-15
- 姓名：李四；年龄：28；部门：市场部；入职时间：2023-03-20
- 姓名：王五；年龄：35；部门：技术部；入职时间：2021-09-10
元数据：{"sheet": "员工基本信息", "start_row": 2, "end_row": 4, "level": "basic"}
```

2. #### 进阶策略：两级分块（适配含业务模块的复杂 Excel）
    

适用于包含明确业务模块划分的 Excel（如电商系统功能清单、产品参数表），通过 “父块（模块级）+ 子块（行级）” 的分层设计，既保留业务背景，又实现原子化检索。

1. ##### 分块逻辑
    

- 父块：对应业务模块 / 功能域（如 “用户管理”“商品管理”），包含模块名称、总记录数等背景信息，不重复子块内容，用于提供业务上下文；
    
- 子块：对应行级原子化记录（如 “用户注册”“商品上架”），包含完整字段信息，用于精准检索；
    
- 关联机制：子块通过`metadata`中的 “模块名称” 与父块关联，检索时可通过父 ID 回溯上下文。
    

2. ##### 分块示例与代码实现
    

如下Excel表格：

|   |   |   |   |   |
|---|---|---|---|---|
|模块名称|功能 ID|功能名称|功能类型|描述|
|用户管理|USR-001|用户注册|基础|支持手机号 / 邮箱注册|
||USR-002|用户登录|基础|支持密码 / 验证码登录|
||USR-003|个人信息修改|扩展|修改昵称、头像等信息|
|商品管理|PROD-001|商品上架|核心|填写名称、价格、库存等|
||PROD-002|商品下架|核心|支持批量 / 单个下架|

1. ###### 生成父块（模块级）
    

父块为每个模块的 “摘要型背景”，包含模块名称和记录数，不重复子块内容。

```Python
from langchain.docstore.document import Document  

# 按“模块名称”分组，生成父块  
parent_docs = []  
for module, sub_df in df.groupby("模块名称"):  
    parent_content = f"模块名称: {module}\n总记录数: {len(sub_df)}"  # 仅保留结构信息  
    parent_docs.append(  
        Document(  
            page_content=parent_content,  
            metadata={"module": module, "level": "parent"}  # 标记层次和模块  
        )  
    )  
```

父块实例：

- 父块 1（用户管理）：
    

```Markdown
模块名称: 用户管理  
总记录数: 3  
```

- metadata: `{"module": "用户管理", "level": "parent"}`
    
- 父块 2（商品管理）：
    

```Plain
模块名称: 商品管理  
总记录数: 2  
```

- metadata: `{"module": "商品管理", "level": "parent"}`
    

2. ###### 生成子块（行级）
    

子块为每行的原子化记录，需包含列名 - 值对（方便 LLM 理解）和 metadata（支持追溯）。

```Python
child_docs = []  
for idx, row in df.iterrows():  
    # 行内容转换为Markdown格式（列名: 值）  
    row_content = "\n".join([  
        f"**功能ID**: {row['功能ID']}",  
        f"**功能名称**: {row['功能名称']}",  
        f"**功能类型**: {row['功能类型']}",  
        f"**描述**: {row['描述']}"  
    ])  
    # 生成子块Document  
    child_docs.append(  
        Document(  
            page_content=f"# 模块: {row['模块名称']}\n{row_content}",  # 标题关联模块  
            metadata={  
                "module": row['模块名称'],  # 关联父块模块  "level": "child",  
                "row_id": idx + 2,  # Excel行号（原表格行号，含表头）  "sheet": "核心功能"  
            }  
        )  
    )  
```

子块实例：

- 子块 1（用户注册）：
    

```Plain
# 模块: 用户管理  
**功能ID**: USR-001  
**功能名称**: 用户注册  
**功能类型**: 基础  
**描述**: 支持手机号/邮箱注册  
```

- metadata: `{"module": "用户管理", "level": "child", "row_id": 2, "sheet": "核心功能"}`
    
- 子块 2（商品上架）：
    

```Plain
# 模块: 商品管理  
**功能ID**: PROD-001  
**功能名称**: 商品上架  
**功能类型**: 核心  
**描述**: 填写名称、价格、库存等  
```

  

- metadata: `{"module": "商品管理", "level": "child", "row_id": 5, "sheet": "核心功能"}`
    

3. ###### 子块长度控制（可选）
    

若行内容过长（如 “描述” 字段含多段文本），用`RecursiveCharacterTextSplitter`递归切分，保持`parent_id`不变：

```Python
from langchain.text_splitter import RecursiveCharacterTextSplitter  

splitter = RecursiveCharacterTextSplitter(chunk_size=200, chunk_overlap=20)  # 假设短文本阈值  
child_chunks = splitter.split_documents(child_docs)  # 过长子块将被拆分  
```

4. ###### 检索流程：父子块协同工作
    

当用户提问：“电商系统中‘商品上架’属于哪个模块？功能类型是什么？” 时，检索流程如下：

- 向量检索子块： 向量库根据问题检索相似子块，命中 “商品上架” 对应的子块（子块 2）。
    
- 通过 metadata 关联父块： 子块 metadata 中的`"module": "商品管理"`指向父块 2，获取父块背景信息。
    
- 拼接上下文： 最终输入 LLM 的上下文为：
    

```Plain
父块：模块名称: 商品管理 总记录数: 2  
子块：# 模块: 商品管理  
**功能ID**: PROD-001  
**功能名称**: 商品上架  
**功能类型**: 核心  
**描述**: 填写名称、价格、库存等 
```

- LLM 回答： “‘商品上架’属于‘商品管理’模块，功能类型为核心。”
    

3. #### 特殊策略：语义区域分块（适配混合内容 Excel）
    

适用于包含 “表格 + 说明文本” 的混合内容 Excel（如带注释的财报、含使用说明的参数表），核心是按 “语义边界” 拆分区域，再对区域内内容细分。

1. ##### 分块逻辑
    

- 识别语义区域：通过空行、加粗标题行、单元格格式（如字体大小、颜色）分割不同语义区域（如 “Q1 销售数据”“数据说明”“汇总统计”）；
    
- 区域内分块：
    
    - 文本区域（如说明文字）：按句子 / 段落分块，使用常规文本分块器（如`RecursiveCharacterTextSplitter`）；
        
    - 表格区域（如数据表格）：按行级分块，保留字段关联；
        
    - 大单元格区域（如单个单元格含长文本）：视为独立 “小文档”，按段落拆分，保留单元格坐标元数据。
        

2. ##### 代码实现
    

```Python
def process_large_cell(cell_value: str, sheet_name: str, cell_coord: str) -> List[Document]:"""
    处理单个大单元格（含长文本），按段落分块
    :param cell_value: 单元格文本内容
    :param sheet_name: 工作表名称
    :param cell_coord: 单元格坐标（如“A10”）
    :return: 分块后的Document列表
    """if not cell_value or len(str(cell_value)) < 200:  # 短文本无需拆分return [Document(
            page_content=f"工作表【{sheet_name}】-{cell_coord}单元格：{cell_value}",
            metadata={"sheet": sheet_name, "cell": cell_coord, "level": "cell"})]# 长文本按段落拆分
    splitter = RecursiveCharacterTextSplitter(chunk_size=300, chunk_overlap=50, separators=["\n\n", "\n"])
    cell_chunks = splitter.split_text(cell_value)return [Document(
        page_content=f"工作表【{sheet_name}】-{cell_coord}单元格（分块{i+1}）：{chunk}",
        metadata={"sheet":
```

3. ##### 分块示例
    

在这份 Excel 文件中，“销售数据” 区域记录了每个月各类产品的销售数量、销售额等具体数据，每一行代表一个产品类别在一个月内的销售情况，表头明确标注了 “月份”“产品类别”“销售数量”“销售额” 等字段。例如：

|   |   |   |   |
|---|---|---|---|
|月份|产品类别|销售数量|销售额|
|1 月|产品 A|100|10000|
|1 月|产品 B|150|15000|
|2 月|产品 A|120|12000|

“销售趋势分析” 区域则以段落文字的形式，对每个月的销售趋势进行了详细描述，包括哪些产品销量增长、哪些产品销量下滑以及可能的原因分析等。例如：“1 月份，产品 A 销量较上月有所增长，主要原因是市场推广活动效果显著，吸引了更多新客户购买。而产品 B 销量保持稳定，维持在较高水平。”

  

“汇总统计” 区域通过公式计算得出了整个季度的销售总量、销售总额，以及各类产品的销售占比等数据。如：“本季度销售总额为 1000000 元，其中产品 A 销售额占比 30%，产品 B 销售额占比 40%……”

  

对于这样一个混合内容的 Excel 文件，采用语义区域分块的具体操作如下：首先，识别出各个语义区域。通过观察可以发现，“销售数据” 区域是一个规整的表格，其上方的表头行与下方的数据行紧密关联，构成了一个完整的语义单元；“销售趋势分析” 区域是纯文本段落，用于对销售数据进行解读和分析；“汇总统计” 区域则通过公式计算出关键统计信息，与前面的销售数据和分析部分共同构成了完整的销售分析报告。

  

对于 “销售数据” 区域，按照行级进行分块，每一行数据作为一个子块，确保每个子块包含完整的产品销售信息（月份、产品类别、销售数量、销售额）以及对应的表头信息。如对于 “1 月 产品 A 100 10000” 这一行数据，生成的分块内容可以是：“工作表【季度销售分析报告】：表头：[月份，产品类别，销售数量，销售额]；数据：- 月份：1 月；产品类别：产品 A；销售数量：100；销售额：10000”。

  

对于 “销售趋势分析” 区域，利用文本分块器，按照段落进行分块。如上述关于 1 月份销售趋势的描述段落，可单独作为一个分块，内容为：“工作表【季度销售分析报告】：销售趋势分析段落：1 月份，产品 A 销量较上月有所增长，主要原因是市场推广活动效果显著，吸引了更多新客户购买。而产品 B 销量保持稳定，维持在较高水平。”

  

对于 “汇总统计” 区域，将计算得出的关键统计信息作为一个整体分块。如：“工作表【季度销售分析报告】：汇总统计：本季度销售总额为 1000000 元，其中产品 A 销售额占比 30%，产品 B 销售额占比 40%……”

---

  

## 特殊的分块方式

1. ### 句子窗口检索（Sentence-Window Retrieval）
    

准确地讲，Sentence-Window Retrieval（检索期再扩窗）不是切片方式，而是 检索策略 —— 先以“单句 chunks”建索引，命中后再把前后 N 句 拼回去给 LLM。

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=MmU2MmM2ZGZiOTg5YmQ2OTA5YjA4NzMxY2I1NmE1NmNfNzlqZFFpb3VnV2x3VGliVWlDelhDam1pRm1mSFNLNlBfVG9rZW46WnQ4ZGJnbXpxb2g3WVN4ZTc5UWNYeFlybjFjXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

**原理概述：**

传统的 RAG 系统通常将文档按固定长度或段落进行切分，并对每个块进行向量化处理。然而，这种方法可能导致语义相关的信息被切割，影响检索效果。

句子窗口检索通过以下步骤优化这一过程：

1. 按句子切分文档：将文档按句子进行切分，每个句子作为一个最小的检索单元。
    
2. 构建句子窗口：对于每个句子，记录其前后若干个句子，形成一个“窗口”。这个窗口包含了目标句子及其上下文信息。
    
3. 向量化处理：仅对目标句子进行向量化处理，而将其窗口信息作为元数据存储。
    
4. 检索与生成：在检索阶段，根据用户查询与句子向量的相似度，找到最相关的句子，并将其对应的窗口信息提供给语言模型，以生成更准确的答案。
    

这种方法结合了细粒度的检索和丰富的上下文信息，提升了检索的精确度和生成的质量。

---

---

---

2. ### 自动合并检索（Auto-merging retrieval）
    

![](https://jqsc332c9hg.feishu.cn/space/api/box/stream/download/asynccode/?code=NjI3NzMwZjhmN2I5NWIxOTI4ZmI1ZjhkYzJjYTYxMjZfTjZpbGRHcGs3aGlHUzJWRllIS05JS0hyVG1sUmR2YVBfVG9rZW46Sk5pcWJsbElXb1ZHVXN4TXY0RmM0TzZ0bkRkXzE3NzM0NzkyMTM6MTc3MzQ4MjgxM19WNA)

Auto-merging Retrieval（自动合并检索）本质上是一种检索策略，而非单纯的 chunk（文本切分）方法。要实现这一策略，确实需要配合特定的 chunk 策略，尤其是层次化的 chunking（Hierarchical Chunking）。

我们前文说的 “Document structure-based chunking”（基于文档结构的切分）就属于“Hierarchical Chunking”（层次化切分）策略的一种。

**原理概述：**

自动合并检索的核心思想是将文档划分为多个层次的块（chunk），形成父子关系的树状结构。在检索过程中，如果多个相关的子块属于同一个父块，系统会自动将这些子块合并为其父块，从而提供更完整的上下文信息给大语言模型（LLM）

**实现步骤：**

1. 文档层次化拆分：使用如 LlamaIndex 的 HierarchicalNodeParser 或 Haystack 的 HierarchicalDocumentSplitter，将文档按预设的块大小（如 2048、512、128）递归拆分，构建出多层级的节点结构。
    
2. 索引构建：将最小的叶子节点（最小块）进行向量化，并存入向量数据库中，供后续检索使用。
    
3. 检索与合并：
    

- 在用户查询时，系统首先检索与查询最相关的叶子节点。
    
- 如果检索到的叶子节点中，有多个属于同一个父节点，并且超过设定的阈值（例如，超过该父节点子节点数量的50%），则系统会自动将这些子节点合并为其父节点，作为最终的检索结果返回
    

**优势：**

- 增强上下文完整性：通过合并相关的子块，提供更连贯的上下文信息，减少信息碎片化。
    
- 提高生成质量：更完整的上下文有助于大语言模型生成更准确、相关性更高的回答。
    
- 降低幻觉风险：提供更全面的信息，减少模型因上下文不足而产生的错误回答。
    

**注意事项：**

- 合理设置层次结构：根据文档的结构和内容，选择合适的 chunk 大小和层数，避免层次过多或过少影响检索效果。
    
- 调整合并阈值：根据具体应用场景，设置合适的合并阈值，确保在需要时合并相关子块，同时避免引入不相关的信息。
    
- 利用元数据管理父子关系：在构建层次结构时，确保每个子块都包含其父节点的引用信息，以便在检索时能够正确合并。