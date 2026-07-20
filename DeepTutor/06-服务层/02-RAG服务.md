# RAG 服务

> 源码：`deeptutor/services/rag/`（约 20 个文件，核心 ~3000 行）
> 复杂度：`complex`

## 一句话说明

RAG（Retrieval-Augmented Generation，检索增强生成）服务是 DeepTutor 的"教材搜索引擎"——教师上传 PDF/Word/Markdown 等教材，系统自动解析、切块、向量化，然后当学生提问时，从知识库中检索最相关的内容，作为上下文喂给 LLM，让 AI 基于教材回答，而不是凭空编造。

**类比**：就像考试时允许"开卷"——AI 不是凭记忆回答，而是先翻书找到相关段落，再基于书上的内容组织答案。

## 业务价值

没有 RAG，AI 回答教学问题只能靠"记忆"（训练数据），容易：
- **胡说八道**（幻觉）：编造不存在的公式、概念
- **过时**：训练数据截止日期之后的内容不知道
- **不精准**：无法引用特定教材的具体章节

有了 RAG：
1. 教师上传教材 → 系统自动解析为可检索的知识库
2. 学生提问 → 系统从知识库中检索相关段落
3. LLM 基于检索到的内容回答，同时标注引用来源
4. 学生看到"这个答案来自《高等数学》第 87 页"

## 核心流程

```
教师上传 PDF
    ↓
文档解析（Parsing 服务）
    ↓
文本切块（Chunking）
    ↓
向量化（Embedding 服务）
    ↓
存入向量数据库
    ↓
    ↓  —— 以上是"建库"阶段，以下是"检索"阶段
    ↓
学生提问
    ↓
问题向量化
    ↓
语义检索（在向量数据库中找最相似的段落）
    ↓
检索结果排序（Re-ranking）
    ↓
把检索结果作为上下文喂给 LLM
    ↓
LLM 基于上下文生成回答
```

## 源码逐模块解析

### 1. `service.py` — RAG 服务入口（RAGService 类）

**做什么**：统一入口，根据知识库绑定的 Provider 路由到对应的 Pipeline。

**关键方法**：

#### `__init__()` — 初始化
确定知识库的存储根目录（`kb_base_dir`），默认在运行时数据目录下。

#### `query()` — 检索
**输入**：知识库名称、查询文本
**输出**：最相关的文档片段列表
**逻辑**：找到知识库绑定的 Pipeline → 调用 Pipeline 的检索方法

#### `add_documents()` — 添加文档
**输入**：知识库名称、文件路径列表
**输出**：添加结果（成功/失败、文档数量）
**逻辑**：解析文档 → 切块 → 向量化 → 存入向量数据库

#### `delete_documents()` — 删除文档
从知识库中删除指定文档的向量数据。

### 2. `factory.py` — Pipeline 工厂

**做什么**：管理多种 RAG Pipeline，根据配置选择使用哪一个。

**支持的 Pipeline**：
- **LlamaIndex**：默认方案，功能最全，支持多种向量数据库
- **LightRAG**：轻量级方案，适合中小规模知识库
- **GraphRAG**：图谱增强方案，能理解实体之间的关系（比如"牛顿第二定律"和"力学"的关系）

**关键函数**：

#### `get_pipeline()` — 获取 Pipeline
根据 Provider 名称返回对应的 Pipeline 实例。

#### `list_pipelines()` — 列出所有可用 Pipeline
返回所有已注册的 Pipeline 名称列表。

#### `normalize_provider_name()` — 标准化 Provider 名称
把用户输入的名称（可能是缩写或别名）转换成标准名称。

### 3. `pipelines/llamaindex/` — LlamaIndex Pipeline

**做什么**：基于 LlamaIndex 框架实现的标准 RAG Pipeline。

**核心文件**：
- `embedding_adapter.py`：Embedding 适配器，把 DeepTutor 的 Embedding 服务适配成 LlamaIndex 需要的格式
- `ingestion.py`：文档摄取，解析文档 → 切块 → 向量化 → 存入索引
- `query.py`：检索查询，把用户问题转成向量 → 在索引中搜索 → 返回结果

**关键类**：

#### `LlamaIndexEmbeddingAdapter`（`embedding_adapter.py`）
**做什么**：适配器模式——把 DeepTutor 的 Embedding 服务包装成 LlamaIndex 需要的 `BaseEmbedding` 接口。

**方法**：
- `_get_text_embedding()`：对单个文本生成向量
- `_get_text_embeddings()`：批量生成向量

#### `IngestionPipeline`（`ingestion.py`）
**做什么**：完整的文档摄取流程。

**流程**：
1. 读取文件（支持 PDF、DOCX、MD、TXT 等）
2. 文本切块（默认每块 512 tokens，重叠 50 tokens）
3. 生成向量（调用 Embedding 服务）
4. 存入向量索引（默认用内存索引，生产环境可配置为 ChromaDB/Pinecone）

### 4. `file_routing.py` — 文件路由

**做什么**：根据文件类型选择合适的解析器。

**`classify_files()`**：判断文件类型，返回分类结果。
- PDF → PDF 解析器
- DOCX → DOCX 解析器
- MD → Markdown 解析器
- TXT → 纯文本解析器

### 5. 知识库管理

#### `KnowledgeBaseManager`（`manager.py`）
**做什么**：知识库的 CRUD 操作。

**方法**：
- `create_kb()`：创建新知识库（在磁盘上创建目录结构）
- `delete_kb()`：删除知识库（删除目录和所有向量数据）
- `list_kbs()`：列出所有知识库
- `get_kb_info()`：获取知识库的详细信息（文档数、状态等）

#### `KnowledgeBaseInitializer`（`initializer.py`）
**做什么**：初始化知识库——创建目录结构、配置文件、元数据。

## 调用链路示例

```
教师上传 PDF 教材
  ↓
API 层: KnowledgeRouter.upload_files()
  ↓
RAG 服务: RAGService.add_documents(kb_name, file_paths)
  ↓
Pipeline: LlamaIndexPipeline.ingest(files)
  ↓
  ├─ 文档解析: Parsing 服务解析 PDF → 纯文本
  ├─ 文本切块: 每 512 tokens 一块，重叠 50 tokens
  ├─ 向量化: Embedding 服务把文本块转成向量
  └─ 存储: 向量存入索引
  ↓
返回："已添加 3 个文档，共 156 个文本块"
```

```
学生提问 "牛顿第二定律是什么？"
  ↓
Agent 层: AgentLoop 决定调用 RAG 工具
  ↓
Tool: RAGTool.execute(query="牛顿第二定律是什么？", kb_name="物理教材")
  ↓
RAG 服务: RAGService.query(kb_name, query)
  ↓
Pipeline: 问题向量化 → 语义检索 → 返回 Top-5 相关段落
  ↓
Agent 把检索结果作为上下文传给 LLM
  ↓
LLM 基于上下文生成回答："牛顿第二定律是 F=ma..."
```

## 与其他服务的关系

- **RAG + Embedding**：RAG 用 Embedding 服务把文本转成向量
- **RAG + Parsing**：RAG 用文档解析服务解析 PDF/DOCX 等文件
- **RAG + LLM**：RAG 检索到的内容作为上下文传给 LLM
- **RAG + Memory**：知识库的元数据存储在 Memory 中

## 设计要点

1. **多 Pipeline 支持**：同一套接口，底层可以切换不同的 RAG 引擎
2. **文件路由**：根据文件类型自动选择合适的解析器
3. **增量更新**：支持添加/删除单个文档，不需要重建整个索引
4. **引用溯源**：检索结果带来源信息，LLM 可以标注引用

### RAG 检索流程详解

**第一步：文档接收**
教师通过知识库 API 上传文件（PDF、DOCX、Markdown、TXT 等），文件被保存到 `raw/` 目录。

**第二步：文档解析**
调用 `DocumentParser` 服务，将原始文件转换为纯文本。支持多种解析引擎：
- MinerU：适合学术 PDF
- Docling：适合 Office 文档
- PyPDF2 / pdfplumber：通用 PDF 解析
- markdown-it：Markdown 解析

**第三步：文本切块（Chunking）**
将长文本切成小块（chunk），每块约 500-1000 字符。切块策略：
- 按段落切分
- 重叠窗口（overlap），防止语义断裂
- 保留标题层级信息

**第四步：向量化（Embedding）**
每个 chunk 通过 Embedding 服务转换为向量（一串数字），存储到向量数据库。

**第五步：索引构建**
向量数据库建立索引，支持快速检索。DeepTutor 支持多种向量存储后端：
- ChromaDB（默认）
- FAISS
- Qdrant
- LanceDB

**第六步：查询处理**
学生提问时：
1. 问题被向量化
2. 在向量数据库中搜索最相似的 chunk
3. 返回 top-k 结果（通常 3-5 个）
4. 将检索结果拼接到 Prompt 中
5. LLM 基于检索结果生成回答

### RAG Pipeline 路由

DeepTutor 支持多种 RAG Pipeline，按需路由：

| Pipeline | 适用场景 | 特点 |
|----------|---------|------|
| LlamaIndex | 通用 RAG | 功能全面，支持多种索引类型 |
| LightRAG | 轻量级 RAG | 快速、简单，适合小规模知识库 |
| GraphRAG | 知识图谱 RAG | 适合复杂关系推理 |
| PageIndex | 页面级索引 | 按页面组织，支持页码引用 |

**Pipeline 选择逻辑**：
1. 创建知识库时，用户可以选择 Pipeline
2. 系统根据知识库类型自动推荐
3. 运行时通过 `RAGService` 的 `provider` 参数路由到正确的 Pipeline

### 知识库管理

**知识库生命周期**：
1. 创建（Create）：设置名称、描述、Pipeline 类型
2. 上传（Upload）：添加文档文件
3. 处理（Process）：后台解析、切块、向量化
4. 索引（Index）：构建检索索引
5. 查询（Query）：通过 API 检索
6. 更新（Update）：添加/删除文档，重新处理
7. 删除（Delete）：清理所有数据

**知识库状态**：
- `ready`：可用
- `processing`：处理中
- `error`：处理失败
- `empty`：无文档

### 检索策略

**语义检索（Semantic Search）**：
- 基于向量相似度
- 适合概念性问题，如"什么是光合作用"

**关键词检索（Keyword Search）**：
- 基于 BM25 算法
- 适合精确匹配，如"第3章第2节"

**混合检索（Hybrid Search）**：
- 语义 + 关键词结合
- 默认策略，兼顾准确性和覆盖面

**重排序（Reranking）**：
- 对初步检索结果进行二次排序
- 使用 Cross-Encoder 模型提高准确性
- 支持 Cohere Rerank、Jina Reranker 等



### RAG 性能优化

**检索速度优化**：
- 向量索引预加载到内存
- 批量查询减少往返次数
- 结果缓存（相同查询 5 分钟内复用）

**检索质量优化**：
- 查询改写：把学生口语化问题改写成更适合检索的形式
- 多路召回：同时用语义和关键词检索，合并结果
- 重排序：用 Cross-Encoder 模型对初步结果二次排序

### RAG 的局限性

1. 只能检索已上传的内容，不能回答教材外的问题
2. 检索质量依赖文档切块策略
3. 对图片、表格中的信息检索效果较差
4. 需要定期更新索引以反映文档变更
