# DeepTutor 项目深度剖析报告（源码级）

> Understand-Anything 知识图谱: 26 节点, 25 边, 7 层
> **GitHub 项目**：https://github.com/HKUDS/DeepTutor
> **分析版本**：main 分支（zip snapshot），分析时间 2026-07-20
> **项目简介**：HKUDS 实验室开源的 AI 驱动终身个性化辅导系统，采用多 Agent 架构


> 📅 分析时间：2026-07-20
> 📊 数据来源：`.ua/knowledge-graph.json` + `.codegraph` 索引 + 核心源码阅读
> 🎯 目标读者：希望从源码层面理解 DeepTutor 的产品/技术人员
> 📌 说明：本报告只聚焦 DeepTutor 一个项目，对其核心模块、关键实体逐个源码级拆解。

---

## 一、系统全景

### 1.1 一句话定位

DeepTutor 是一个 **AI 驱动的智能教学平台**。它不是简单的"聊天机器人 + 题库"，而是一个以 **Agent 循环（AgentLoop）** 为核心、以 **知识库（RAG）** 为土壤、以 **学习记忆** 为大脑的个性化辅导系统。

### 1.2 目标用户与使用场景

| 角色 | 典型场景 | 系统提供的能力 |
|------|---------|---------------|
| 学生 | 问问题、刷题、查漏补缺 | 对话答疑、自动出题、自动评分、学习路径 |
| 教师 | 上传教材、追踪学生进度 | 知识库管理、题目生成、掌握度报表 |
| 内容创作者 | 制作互动教材 | Co-Writer 协作写作、Book 编辑器 |
| 管理员 | 配置模型、监控系统 | LLM 配置、Embedding 配置、沙箱策略 |

### 1.3 核心解决的教学痛点

1. **教材和答疑脱节**：学生问的问题，教材里明明有，但普通 AI 搜不到。DeepTutor 用 RAG 知识库把教材变成可检索语义库。
2. **练习没有针对性**：传统题库"千人一面"。DeepTutor 基于掌握度评估和间隔重复调度动态出题。
3. **AI 答案来源不可追溯**：DeepTutor 的工具调用和检索结果都带 `sources` 元数据，前端可展示引用。
4. **长期上下文容易丢失**：DeepTutor 用三级记忆架构（L1/L2/L3）+ 上下文折叠解决。

### 1.4 技术栈总览（源码视角）

| 技术/框架 | 在项目中的作用 | 对应源码位置 |
|-----------|-------------|-------------|
| Python 3.11+ | 后端主力语言 | 919 个 `.py` 文件 |
| FastAPI | HTTP API 路由层 | `deeptutor/api/` |
| Next.js / React / TSX | 前端界面 | `web/app/`, `web/components/` |
| Pydantic | 数据模型与校验 | 大量 `*_models.py` |
| SQLAlchemy / SQLite | 会话、配置持久化 | `deeptutor/services/session/` |
| OpenAI / Anthropic 兼容接口 | LLM 调用 | `deeptutor/services/llm/` |
| LlamaIndex / LightRAG / GraphRAG | 向量检索与知识图谱 | `deeptutor/services/rag/pipelines/` |

### 1.5 规模一览

| 指标 | 数值 |
|------|------|
| 总文件数 | 1,444 |
| 代码实体（函数/类/方法） | 23,030 |
| 实体间关系 | 60,362 |
| Python 文件 | 919 |
| TS/TSX 文件 | 380 |
| 配置文件（YAML） | 135 |
| codegraph DB 大小 | 69 MB |

---

## 二、核心架构分层（源码视角）

```
┌──────────────────────────────────────────────────────────────┐
│  Web 前端层                                                   │
│  Chat 界面 · Book 编辑器 · Co-Writer · 设置面板 · Dashboard    │
├──────────────────────────────────────────────────────────────┤
│  API 路由层（FastAPI）                                          │
│  /chat · /knowledge · /notebook · /co-writer · /partner        │
├──────────────────────────────────────────────────────────────┤
│  Agent 层（核心）                                               │
│  AgentLoop · AgenticChatPipeline · Capability 插件              │
├──────────────────────────────────────────────────────────────┤
│  服务层（Services）                                             │
│  LLM · RAG · Memory · Search · Embedding · Session · Parsing   │
├──────────────────────────────────────────────────────────────┤
│  学习与能力层                                                   │
│  Grading · Mastery · Scheduler · Solve · Explore              │
├──────────────────────────────────────────────────────────────┤
│  工具层（Tools）                                                │
│  Builtin · Question · Vision · MediaGen · Exec                │
├──────────────────────────────────────────────────────────────┤
│  基础设施层                                                     │
│  Config · StreamBus · Trace · Context · Events                 │
└──────────────────────────────────────────────────────────────┘
```

---

## 三、核心实体逐个源码级解析

以下按照知识图谱中的节点，分层次逐个展开说明，每个节点给出：定位、源码位置、核心职责、关键设计。


### API 层

#### api_main — API 路由入口 (复杂度: simple)
**实体类型**：module
**一句话说明**：FastAPI 应用主入口，注册所有路由模块，包括知识库、聊天、笔记本、协作写作、伙伴系统等 API 端点
**源码级解析**：
`api_main` 是 FastAPI 应用的入口，负责把所有路由挂到应用实例上。可以把它理解为"总机"：所有来自前端的 HTTP 请求先到这里，再被分到 /chat、/knowledge、/notebook 等子路由。源码中没有复杂业务逻辑，主要是注册和异常处理。

#### api_knowledge — KnowledgeRouter (复杂度: complex)
**实体类型**：class
**一句话说明**：知识库管理 API：上传文档、创建知识库、检索、删除等 REST 端点
**源码级解析**：
`KnowledgeRouter` 是知识库 REST API 的核心类。它暴露了一组端点：创建知识库（POST）、列出知识库（GET）、上传文件（POST /{kb_id}/files）、检索（POST /{kb_id}/search）、删除（DELETE）。每个端点背后都调用 `RAGService`。文件上传后会进入异步解析队列，`parsing` 服务负责把文档拆块、向量化、建索引。

#### api_chat — ChatRouter (复杂度: complex)
**实体类型**：class
**一句话说明**：聊天 API 端点，处理 SSE 流式对话、消息发送、会话管理
**源码级解析**：
`ChatRouter` 处理 SSE 流式聊天。它接收 `{session_id}/messages` 请求，构造 `ChatContext`，然后交给 `AgenticChatPipeline.run()`。响应通过 Server-Sent Events（SSE）逐字流回前端，这样用户能看到 AI"打字"的效果。源码中处理了工具调用事件、思考内容过滤、中断恢复等细节。

#### api_notebook — NotebookRouter (复杂度: moderate)
**实体类型**：class
**一句话说明**：学习笔记本 API：练习记录、错题管理、进度追踪
**源码级解析**：
`NotebookRouter` 提供学习笔记本的 CRUD 接口：练习记录、错题、笔记条目、学习进度。数据最终持久化到 SQLite/Postgres（由 session 服务管理）。

### Agent 层

#### agent_loop — AgentLoop (复杂度: complex)
**实体类型**：class
**一句话说明**：核心对话 Agent 循环：管理 LLM 调用、工具执行、上下文折叠、流式响应
**源码级解析**：
`AgentLoop` 是 DeepTutor 的心脏。源码在 `deeptutor/agents/chat/agent_loop.py`（667 行）。它采用"单循环"设计：一次用户提问 = 一个 AgentLoop 实例；每一轮调用 LLM，LLM 可以选择调用工具（narration）或直接回答（finish）。循环最多 8 轮，超过强制 finish。核心流程：
- 初始化 `AgentLoopState`
- 调用 `_run_loop()`
- 每轮 `_call_llm()` 流式请求 LLM
- 若返回 tool_calls，调用 `_dispatch_tool_calls()`
- 工具结果追加到上下文，继续下一轮
- 直到返回无 tool_calls，或强制 finish
关键设计：流式 SSE、上下文折叠、错误兜底。

#### agent_pipeline — AgenticChatPipeline (复杂度: complex)
**实体类型**：class
**一句话说明**：Agentic 聊天管道：编排 AgentLoop 的生命周期，包括初始化、运行、清理
**源码级解析**：
`AgenticChatPipeline` 是 AgentLoop 的"导演"，源码在 `deeptutor/agents/chat/agentic_pipeline.py`（1380 行）。主要工作：
1. 根据用户能力和上下文构建启用工具集
2. 从 RAG 检索 KB seed
3. 组装 system prompt 和 loop messages
4. 实例化 `AgentLoop` 并运行
5. 处理上下文窗口 guard
6. 返回流式结果给 ChatRouter
工具组合逻辑在 `_compose_enabled_tools()` 中，会根据用户是否挂载知识库、是否有记忆、是否允许执行代码等动态决定。

#### agent_coordinator — AgentCoordinator (复杂度: moderate)
**实体类型**：class
**一句话说明**：题目生成协调器：根据知识点或考试要求自动生成练习题
**源码级解析**：
`AgentCoordinator` 是题目生成的协调器。它接收知识点或难度要求，调用 LLM 生成题目，并返回结构化的题目数据。它不是简单 prompt，而是有模板和校验的生成流程。

#### agent_visualize — AnalysisAgent (复杂度: moderate)
**实体类型**：class
**一句话说明**：可视化分析 Agent：将数据转换为图表和可视化展示
**源码级解析**：
`AnalysisAgent` 负责把学习数据（如掌握度、练习分布）转换成图表。它调用 LLM 生成 ECharts / Plotly 配置，或直接输出 Mermaid 图，前端渲染。

#### agent_research — Research Agent (复杂度: complex)
**实体类型**：module
**一句话说明**：研究 Agent：深度调研和知识探索，支持多轮对话和上下文管理
**源码级解析**：
Research Agent 负责深度调研。它会多轮检索知识库、网页、文档，然后生成一份综合报告。适用于"帮我调研一下 XX 主题"这类任务。

### 基础设施层

#### core_agentic — Agentic 核心框架 (复杂度: complex)
**实体类型**：module
**一句话说明**：Agentic 框架核心：工具调用协议、标签分类、提示词模板、状态管理
**源码级解析**：
`core_agentic` 是 Agent 基础设施层。源码在 `deeptutor/core/agentic/`。包含：
- `loop.py`：底层 LLM 调用循环
- `tool_dispatch.py`：工具分发协议
- `labels.py`：标签分类（如 `<thinking>`、`<search>`）
- `client.py`：LLM 客户端封装
- `messages.py`：消息格式化
- `usage.py`：Token 用量追踪
它是整个 Agent 系统的"操作系统"。

#### core_events — 事件系统 (复杂度: moderate)
**实体类型**：module
**一句话说明**：事件总线：发布-订阅模式的事件系统，连接各模块间的通信
**源码级解析**：
`core_events` 是发布-订阅事件总线。各模块通过事件解耦，例如：文件上传完成 -> 触发解析事件 -> RAG 索引更新。源码中实现了同步和异步两种事件通道。

#### core_config — 配置管理 (复杂度: simple)
**实体类型**：module
**一句话说明**：应用配置：环境变量、模型设置、服务参数管理
**源码级解析**：
`core_config` 负责配置加载和运行时管理。DeepTutor 有大量 YAML/JSON 配置，`core_config` 提供统一的读取、校验、热更新能力。`runtime_settings.py`、`launch_settings.py`、`model_catalog.py` 都在这里。

### 服务层

#### svc_llm — LLM 服务 (复杂度: complex)
**实体类型**：module
**一句话说明**：LLM 集成层：多模型提供商（OpenAI, Anthropic, 本地模型）的统一接口，支持流式和非流式调用
**源码级解析**：
LLM 服务是 DeepTutor 调用大模型的统一抽象。源码在 `deeptutor/services/llm/`。它屏蔽了 OpenAI、Anthropic、Azure 等厂商的差异。核心组件：
- `LLMClient`：类式接口（legacy）
- `complete()` / `stream()`：工厂函数，推荐新代码使用
- `config.LLMConfig`：模型配置
- `provider_core/`：各厂商适配器
- `utils.py`：URL 清理、上下文窗口解析
`client.py` 中的注释明确说明 `LLMClient` 是 legacy 接口，新代码应直接使用工厂函数。

#### svc_rag — RAG 服务 (复杂度: complex)
**实体类型**：module
**一句话说明**：检索增强生成（RAG）：文档解析、向量存储、语义检索、文件路由
**源码级解析**：
RAG 服务源码在 `deeptutor/services/rag/service.py`。它通过工厂模式路由到不同的 pipeline：LlamaIndex、LightRAG、GraphRAG、PageIndex。每个 pipeline 都实现了统一的检索接口。
核心方法：
- `RAGService.query()`：对知识库做语义检索
- `RAGService.add_documents()`：添加文档
- `RAGService.delete_documents()`：删除文档
`factory.py` 负责根据配置选择 provider，`provider_binding.py` 负责解析绑定关系。RAG 是 DeepTutor 能把教材"读进去"的关键。

#### svc_memory — Memory 服务 (复杂度: complex)
**实体类型**：module
**一句话说明**：长短期记忆管理：L1/L2/L3 三级记忆架构，支持记忆整合和检索
**源码级解析**：
Memory 服务源码在 `deeptutor/services/memory/`。核心类 `MemoryStore` 是无状态门面，通过 `paths.memory_root` 隔离不同用户。它管理三级记忆：
- **L1 Trace**：原始事件序列，由 `trace.py` 管理
- **L2 Document**：结构化记忆文档，由 `document.py` 管理
- **L3 Snapshot**：长期画像，由 `snapshot.py` 管理
`consolidator/` 负责把 L1 整合到 L2/L3。`ops.py` 定义了 Add、Edit、Delete 三种记忆操作及审计。`ids.py` 使用 ULID 生成唯一 ID。

#### svc_search — 搜索服务 (复杂度: moderate)
**实体类型**：module
**一句话说明**：搜索集成：多搜索引擎聚合，支持 Tavily, Brave, DuckDuckGo 等
**源码级解析**：
Search 服务提供跨知识库、跨记忆、跨笔记本的统一搜索。它把不同数据源的搜索结果合并、排序、去重后返回给 Agent。

#### svc_session — 会话服务 (复杂度: moderate)
**实体类型**：module
**一句话说明**：会话管理：对话历史、上下文窗口、多用户会话隔离
**源码级解析**：
Session 服务负责会话状态持久化。一个 `Session` 包含多个 `Turn`，每个 `Turn` 包含多条 `Message`。它是 DeepTutor 能记住对话历史的基础。

#### svc_embedding — Embedding 服务 (复杂度: moderate)
**实体类型**：module
**一句话说明**：向量嵌入服务：文本向量化、相似度计算、模型选择
**源码级解析**：
Embedding 服务负责把文本转成向量。源码在 `deeptutor/services/embedding/`。支持 Cohere、OpenAI、本地模型等多种后端。不同 provider 通过 adapter 模式接入。

#### svc_parsing — 文档解析服务 (复杂度: moderate)
**实体类型**：module
**一句话说明**：多格式文档解析：PDF、DOCX、Markdown 等，支持 MinerU、Docling 引擎
**源码级解析**：
Parsing 服务负责把各种格式文档解析成纯文本。源码在 `deeptutor/services/parsing/`。支持的文档类型包括 PDF、DOCX、Markdown、HTML 等。不同格式由不同 parser 处理，最终统一输出带位置信息的文本块。

### 学习层

#### learning_grading — 题目评分系统 (复杂度: complex)
**实体类型**：function
**一句话说明**：自动评分：对用户答案进行错误分类和评分，支持多种错误类型识别
**源码级解析**：
自动评分源码在 `deeptutor/learning/grading.py`。它提供两个核心函数：
- `grade_answer(user, expected, question_type)`：判断答案是否正确
- `classify_error(user, expected)`：对错误进行分类（如概念错误、计算错误、应用错误等）
当前实现比较朴素：选择题精确匹配，简答题用 SequenceMatcher 计算相似度。`classify_error` 根据答案特征返回 `ErrorType`，供 Mastery 服务使用。

#### learning_scheduler — 学习调度器 (复杂度: complex)
**实体类型**：module
**一句话说明**：学习路径调度：间隔重复、知识点掌握度追踪、自适应学习计划
**源码级解析**：
学习调度器源码在 `deeptutor/learning/scheduler.py`。它根据掌握度、难度、上次复习时间，决定下一次复习时间。采用间隔重复思想：掌握度越低，复习间隔越短。

### 能力层

#### cap_solve — 解题能力 (复杂度: complex)
**实体类型**：module
**一句话说明**：解题模块：逐步引导解题过程，支持数学、编程、物理等多学科
**源码级解析**：
解题能力源码在 `deeptutor/capabilities/solve/`。它提供逐步解题引导，支持数学、编程、物理等。核心流程：
1. 解析题目
2. 调用 `solve` 工具生成解题步骤
3. 每步引导学生
4. 收集学生答案，调用 Grading 评分
5. 根据结果调用 Mastery 更新掌握度

#### cap_mastery — 掌握度评估 (复杂度: moderate)
**实体类型**：module
**一句话说明**：知识点掌握度评估：基于答题历史评估学生对各知识点的掌握程度
**源码级解析**：
掌握度能力源码在 `deeptutor/capabilities/mastery/`。它调用 `learning/mastery.py` 的 `compute_mastery`，基于最近 5 次答题记录计算掌握度。算法是"带置信度上限的加权平均"：
- 最近答案权重更高（0.5 -> 1.0）
- 答案太少时掌握度被 cap，防止"一次猜对"就判定掌握

#### cap_explore — 上下文探索 (复杂度: moderate)
**实体类型**：module
**一句话说明**：上下文探索：深入分析代码/文档的上下文关系，辅助理解复杂结构
**源码级解析**：
上下文探索能力源码在 `deeptutor/capabilities/explore_context/`。它让 Agent 能主动探索代码、文档、知识库的上下文，适合回答"这个知识点和那个知识点有什么关系"这类问题。

### Web 层

#### web_chat — Chat 界面 (复杂度: complex)
**实体类型**：component
**一句话说明**：主聊天界面：Next.js App Router 页面，支持多会话、流式响应、消息渲染
**源码级解析**：
Chat 界面源码在 `web/app/(workspace)/home/` 和 `web/components/chat/`。它接收 SSE 流，实时渲染 LLM 输出，支持多会话、消息引用、工具调用展示。`ChatMessage` 组件负责渲染不同类型的消息块。

#### web_book — Book 学习系统 (复杂度: complex)
**实体类型**：component
**一句话说明**：交互式学习 Book 系统：支持多种 Block 类型（Quiz, Code, FlashCards 等），学习进度追踪
**源码级解析**：
Book 学习系统源码在 `web/app/(workspace)/book/`。它支持多种 Block：Text、Quiz、Code、FlashCards、ConceptGraph、DeepDive、Animation、Timeline。每个 Block 对应一个 React 组件，后端存储为 JSON。

#### web_settings — 设置面板 (复杂度: moderate)
**实体类型**：component
**一句话说明**：系统设置面板：LLM、Embedding、TTS、STT、MCP、代理等各项配置
**源码级解析**：
设置面板源码在 `web/components/settings/`。用户可以配置 LLM provider、模型、Embedding、TTS/STT、MCP、代理等。配置通过 API 保存到后端。

#### web_co_writer — Co-Writer 协作 (复杂度: moderate)
**实体类型**：component
**一句话说明**：AI 协作写作系统：文档编辑、模板管理、实时协作
**源码级解析**：
Co-Writer 协作写作源码在 `web/app/(workspace)/co-writer/`。学生和 AI 一起写文档、制作学习材料。AI 可以编辑、续写、总结。

### 工具层

#### tools_builtin — 内置工具集 (复杂度: moderate)
**实体类型**：module
**一句话说明**：Agent 内置工具：文件操作、代码执行、搜索、浏览器等基础工具
**源码级解析**：
内置工具源码在 `deeptutor/tools/builtin/` 和 `deeptutor/tools/` 顶层。包括：
- `file_tools`：文件读写
- `exec_tool`：代码执行
- `web_search` / `web_fetch`：网页搜索
- `ask_user`：向用户提问并暂停等待回复
- `brainstorm`：头脑风暴
- `github_query`：GitHub 查询
- `media_gen_tool`：媒体生成
这些工具让 Agent 能执行真实动作。

#### tools_question — 题目工具 (复杂度: moderate)
**实体类型**：module
**一句话说明**：题目相关工具：生成题目、评分、获取提示、难度调节
**源码级解析**：
题目工具源码在 `deeptutor/tools/question/`。提供生成题目、提取题目、模拟考试等工具。`exam_mimic.py` 可以生成接近真实考试的题目组合。

#### tools_vision — 视觉工具 (复杂度: moderate)
**实体类型**：module
**一句话说明**：视觉工具：图像分析、OCR、图表识别
**源码级解析**：
视觉工具源码在 `deeptutor/tools/vision/`。支持图片分析、OCR、图表识别、坐标转换。学生可以上传图片题目，Agent 调用视觉工具理解图片内容。

---

## 四、关键模块源码级流程

### 4.1 AgentLoop 单循环详解

`deeptutor/agents/chat/agent_loop.py`（667 行）是 DeepTutor 的心脏。它的 docstring 已经把设计讲得很清楚：

> One chat turn = ONE agent loop over a single growing conversation.

也就是说，**一次用户提问 = 一个 AgentLoop 实例**。

#### 4.1.1 四种 Loop 状态

源码中通过 `AgentLoopState` 跟踪：

```
round_index    当前第几轮 LLM 调用
steps          当前已经执行了多少步工具
sources        收集到的来源引用
paused         是否在等待用户回复
outcome        最终结果
```

#### 4.1.2 一轮的典型流程

```
1. 构造 messages：system + history + user + KB seed
2. 调用 LLM（流式）
3. 如果 LLM 输出普通文本 -> 这可能是 finish，需要检查是否有 tool_calls
4. 如果有 tool_calls -> 这是 narration
   4.1 把文本内容作为 preamble 暂存
   4.2 分发每个 tool_call
   4.3 把结果以 role=tool 的形式追加到 messages
   4.4 round_index += 1，回到第 2 步
5. 如果没有 tool_calls -> 这是 finish
   5.1 文本作为最终答案
   5.2 构造 LoopOutcome 返回
6. 如果超过最大轮数 -> 强制 finish
```

#### 4.1.3 上下文折叠（Context Folding）

当工具返回结果很长时，AgentLoop 会触发 `_fold_context_checkpoint()`，把过长的工具结果压缩成摘要，再喂给下一轮 LLM。这样可以：
- 控制上下文长度
- 保留关键信息
- 避免超出 LLM 上下文窗口

#### 4.1.4 InlineThinkFilter

LLM 有时会在 `<thinking>` 标签里写内部推理。`InlineThinkFilter` 负责：
- 把 `<thinking>...</thinking>` 从用户可见内容中去掉
- 但保留在内部 messages 中，供后续轮次使用

#### 4.1.5 生活化比喻

AgentLoop 就像一个"会自己查资料的研究助理"：
- 学生问问题 = 用户输入
- 助理查书、查笔记、上网搜索 = tool_calls
- 助理自言自语分析 = `<thinking>`
- 助理最终回答 = finish
- 如果资料太多，助理先总结要点 = 上下文折叠

---

### 4.2 AgenticChatPipeline 编排详解

`deeptutor/agents/chat/agentic_pipeline.py`（1380 行）是 AgentLoop 的"导演"。

#### 4.2.1 入口方法 `run()`

```python
async def run(self, context: ChatContext) -> AsyncIterator[...]:
    # 1. 准备工具
    # 2. 准备 KB seed
    # 3. 构造 loop messages
    # 4. 创建 AgentLoop
    # 5. 运行并流式返回
```

#### 4.2.2 工具组合逻辑

工具不是写死的。`_compose_enabled_tools()` 会根据当前上下文动态决定启用哪些：

```python
mount_flags = ToolMountFlags(
    has_kb=bool(self._rag_kbs(context)),
    has_memory=user_has_memory(),
    has_notebooks=user_has_notebooks(),
    has_deferred_tools=...,
    has_exec=...,
)
```

例如：
- 如果用户挂载了知识库，启用 `rag_tool`
- 如果用户有记忆文件，启用 `partner_memory` 工具
- 如果允许执行代码，启用 `exec_tool`
- 如果有笔记本，启用 `write_note`、`list_notebook` 工具

#### 4.2.3 KB seed 注入

`_retrieve_kb_seed_block()` 会从用户挂载的知识库中检索相关内容，然后作为 seed block 注入到 system prompt 中。这样 LLM 在回答时就有教材依据。

---

### 4.3 Memory 三级记忆源码解析

源码在 `deeptutor/services/memory/`。

#### 4.3.1 MemoryStore（无状态门面）

`MemoryStore` 是所有记忆操作的入口。关键设计：**stateless**，每次调用通过 context variable 获取当前用户的 `memory_root`。

```python
class MemoryStore:
    async def add(self, surface: str, text: str) -> ApplyReport: ...
    async def edit(self, surface: str, ref: str, text: str) -> ApplyReport: ...
    async def delete(self, surface: str, ref: str) -> ApplyReport: ...
    async def search(self, surface: str, query: str) -> list[Entry]: ...
```

#### 4.3.2 L1 Trace

`trace.py` 记录每次交互的原始事件。文件按日期命名，格式为 JSONL。

```
memory_root/
  traces/
    2026-07-20.jsonl
    2026-07-21.jsonl
```

#### 4.3.3 L2 Document

`document.py` 定义了 `Document` 和 `Entry`。L2 文档是一个 Markdown/YAML 混合文件，按 "surface"（可以理解为不同的记忆本）组织。

```
memory_root/
  docs/
    surface_name.md
```

每个 Entry 包含：id、content、tags、refs、timestamp、来源 trace。

#### 4.3.4 L3 Snapshot

`snapshot.py` 管理长期画像，如用户偏好、薄弱知识点、学习目标。它是高度凝练的数据。

#### 4.3.5 Consolidator（记忆整合）

当 L1 Trace 积累到一定量，consolidator 会：
1. 读取 Trace 事件
2. 按主题分块
3. 去重
4. 合并相似条目
5. 生成/更新 L2 Document 条目
6. 生成审计报告

---

### 4.4 RAG 服务源码解析

源码在 `deeptutor/services/rag/service.py`。

#### 4.4.1 多 Pipeline 路由

```python
class RAGService:
    def __init__(self, provider: str | None = None, ...):
        # provider 为空时，从 KB 配置读取
        self._pipeline = get_pipeline(provider)

    async def query(self, kb_id, query, top_k=5):
        return await self._pipeline.query(kb_id, query, top_k)

    async def add_documents(self, kb_id, documents):
        return await self._pipeline.add_documents(kb_id, documents)
```

#### 4.4.2 支持的 Pipeline

| Pipeline | 文件位置 | 特点 |
|----------|---------|------|
| LlamaIndex | `pipelines/llamaindex/` | 通用，向量检索 |
| LightRAG | `pipelines/lightrag/` | 图结构增强 |
| GraphRAG | `pipelines/graphrag/` | 微软 GraphRAG |
| PageIndex | `pipelines/pageindex/` | 精确到页 |

#### 4.4.3 文档解析链路

```
上传文件
  -> Parsing Service
    -> 提取文本 + 位置信息
      -> Chunking
        -> Embedding
          -> 向量数据库
            -> 索引构建
```

---

### 4.5 Grading 与 Mastery 源码解析

#### 4.5.1 Grading

`deeptutor/learning/grading.py`：

```python
def grade_answer(user_answer: str, expected_answer: str, question_type: str = "short") -> bool:
    user = user_answer.strip().lower()
    expected = expected_answer.strip().lower()
    if not expected:
        return False
    if question_type == "choice":
        return user == expected
    if question_type == "short":
        return SequenceMatcher(None, user, expected).ratio() > 0.8
    ...

def classify_error(user_answer: str, expected_answer: str) -> ErrorType:
    # 根据答案特征返回错误类型
    ...
```

#### 4.5.2 Mastery

`deeptutor/learning/mastery.py`：

```python
_RECENCY_WEIGHTS = (0.5, 0.7, 0.85, 0.95, 1.0)

# 置信度上限：答题次数不足时不能算掌握
_CONFIDENCE_CAP = {1: 0.4, 2: 0.55, 3: 0.7, 4: 0.85}

def compute_mastery(attempts: list[bool]) -> float:
    recent = attempts[-5:]
    weights = _RECENCY_WEIGHTS[-len(recent):]
    score = sum((1.0 if c else 0.0) * w for c, w in zip(recent, weights)) / sum(weights)
    return min(score, _CONFIDENCE_CAP.get(len(recent), 1.0))
```

也就是说：
- 最近 5 次答题，按权重加权平均
- 答对次数越多、越近，分数越高
- 但答题次数少于 5 次时，分数会被 cap，避免"蒙对"

---

## 五、端到端流程追踪

### 5.1 流程一：学生提问并得到回答

```
学生输入
  -> ChatRouter (/chat/{session_id}/messages)
    -> AgenticChatPipeline.run(context)
      -> _compose_enabled_tools()
      -> _retrieve_kb_seed_block() [如果挂载 KB]
      -> AgentLoop
        -> LLM call 1
          -> 有 tool_calls? -> 执行工具 -> LLM call 2
          -> 无 tool_calls? -> finish
        -> 流式返回 SSE
    <- 前端逐字显示
```

### 5.2 流程二：上传教材构建知识库

```
教师上传 PDF
  -> KnowledgeRouter POST /knowledge/{kb_id}/files
    -> 保存文件到磁盘
    -> 发送事件到 Parsing Service
      -> 解析文本 + 位置信息
      -> Chunking
      -> Embedding Service 生成向量
      -> 存入向量数据库
      -> 构建 RAG 索引
    <- 返回构建状态
```

### 5.3 流程三：自动出题、评分、更新掌握度

```
学生选择知识点
  -> AgentCoordinator 生成题目
    -> LLM 调用生成结构化题目
      -> 学生作答
        -> Grading.grade_answer()
          -> classify_error()
            -> Mastery.compute_mastery()
              -> Scheduler 安排复习
                -> 结果写入 Notebook
```

---

## 六、核心数据实体关系

```
User
 ├── Session（对话会话）
 │    ├── Turn（一次对话轮次）
 │    │    ├── Message（用户/AI 消息）
 │    │    └── ToolCall（工具调用）
 │    └── Context（上下文）
 ├── Notebook（学习笔记本）
 │    └── Record（练习记录）
 ├── KnowledgeBase（知识库）
 │    ├── Document（文档）
 │    └── Chunk（文本块）
 ├── Memory（记忆）
 │    ├── L1 Trace
 │    ├── L2 Document
 │    └── L3 Snapshot
 └── LearningProfile（学习档案）
      ├── Mastery（知识点掌握度）
      └── Scheduler（复习计划）
```

---

## 七、源码阅读路线（推荐顺序）

1. `deeptutor/agents/chat/agent_loop.py` — Agent 循环
2. `deeptutor/agents/chat/agentic_pipeline.py` — 管道编排
3. `deeptutor/core/agentic/tool_dispatch.py` — 工具分发
4. `deeptutor/services/memory/store.py` — 记忆服务
5. `deeptutor/services/rag/service.py` — RAG 服务
6. `deeptutor/services/llm/client.py` + `factory.py` — LLM 调用
7. `deeptutor/learning/grading.py` — 自动评分
8. `deeptutor/learning/mastery.py` — 掌握度
9. `deeptutor/learning/scheduler.py` — 学习调度
10. `deeptutor/capabilities/solve/loop.py` — 解题能力
11. `deeptutor/services/parsing/` — 文档解析
12. `web/app/(workspace)/home/page.tsx` — 前端 Chat 入口

---

## 八、总结

DeepTutor 的技术架构可以概括为：

- **一个核心**：AgentLoop 单循环对话引擎
- **两个支柱**：RAG 知识库、三级记忆系统
- **三个闭环**：提问-回答、出题-评分-掌握度、学习-复习
- **四层架构**：Web 层 -> API 层 -> Agent 层 -> 服务层
- **五种核心能力**：对话、知识库、题目生成、自动评分、学习路径规划

它不是"套壳 ChatGPT"，而是一个完整的教学操作系统。
