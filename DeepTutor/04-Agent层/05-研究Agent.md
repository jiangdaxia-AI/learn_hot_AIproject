# 05 — 研究 Agent（Deep Research Pipeline）

> Understand-Anything 知识图谱: 26 节点, 25 边, 7 层
> **源码位置**：`deeptutor/agents/research/`
> **核心文件**：`pipeline.py`（2800行）、`capability.py`（99行）、`mode_strategy.py`（149行）、`request_config.py`（249行）、`data_structures.py`（564行）
> **辅助文件**：`utils/citation_manager.py`、`utils/json_utils.py`、`utils/token_tracker.py`
> **一句话总结**：研究 Agent 是 DeepTutor 的"深度调研引擎"——从用户的一个问题出发，自动澄清需求→拆解子话题→逐个研究→生成带引用的结构化报告。

---

## 目录

1. [整体架构](#整体架构)
2. [核心数据结构](#核心数据结构)
3. [DeepResearchCapability — 能力入口](#deepresearchcapability--能力入口)
4. [ResearchPipeline — 四阶段管线](#researchpipeline--四阶段管线)
5. [Phase 1: 需求澄清（Rephrase）](#phase-1-需求澄清rephrase)
6. [Phase 2: 话题拆解（Decompose）](#phase-2-话题拆解decompose)
7. [Phase 3: 逐块研究（Research Blocks）](#phase-3-逐块研究research-blocks)
8. [Phase 4: 报告生成（Reporting）](#phase-4-报告生成reporting)
9. [研究模式与策略](#研究模式与策略)
10. [引文管理器](#引文管理器)
11. [动态话题队列](#动态话题队列)
12. [总结](#总结)

---

## 整体架构

如果把研究 Agent 比作一家"调研咨询公司"：

| 文件 | 角色比喻 | 职责 |
|------|----------|------|
| `capability.py` | 公司前台 | 接收客户委托，验证参数，创建管线 |
| `pipeline.py` | 项目总监 | 四阶段流程：澄清需求→拆解话题→逐块研究→撰写报告 |
| `mode_strategy.py` | 服务菜单 | 定义 4 种研究模式（笔记/报告/对比/学习路径）的策略参数 |
| `request_config.py` | 合同条款 | 验证请求参数，将模式+深度转换成可执行的策略配置 |
| `data_structures.py` | 工作台 | 定义 TopicBlock（话题块）、ToolTrace（工具痕迹）、DynamicTopicQueue（动态队列） |

---

## 核心数据结构

### TopicBlock（话题块）

**源码位置**：`data_structures.py` 第188-237行

```python
@dataclass
class TopicBlock:
    block_id: str            # 唯一标识，如 "block_1"
    sub_topic: str           # 子话题名称
    overview: str            # 话题概述/背景
    status: TopicStatus      # 状态：PENDING / RESEARCHING / COMPLETED / FAILED
    tool_traces: list[ToolTrace]  # 工具调用痕迹列表
    iteration_count: int     # 当前迭代次数
    created_at: str          # 创建时间
    updated_at: str          # 更新时间
    metadata: dict           # 额外元数据
```

**大白话解释**：TopicBlock 是调研任务的最小调度单元。每个子话题（如"牛顿第一定律"）就是一个 TopicBlock，它有独立的状态（等待研究→研究中→已完成→失败），记录了 AI 在研究这个子话题时调用了哪些工具、得到了什么结果。

### ToolTrace（工具痕迹）

**源码位置**：`data_structures.py` 第66-185行

```python
@dataclass
class ToolTrace:
    tool_id: str             # 工具调用唯一标识
    citation_id: str         # 引文 ID（如 CIT-1-01）
    tool_type: str           # 工具类型：rag / web_search / paper_search / code_execution
    query: str               # 查询语句
    raw_answer: str          # 原始结果（可能被截断，上限 50KB）
    summary: str             # Note Agent 生成的核心摘要
    timestamp: str           # 时间戳
    raw_answer_truncated: bool  # 原始结果是否被截断
    raw_answer_original_size: int  # 截断前的原始大小
```

**大白话解释**：AI 在研究一个子话题时，可能会调用多次工具（比如搜 3 次网页、查 2 次知识库）。每次工具调用都会留下一个 ToolTrace，记录"查了什么、查到了什么、摘要是什么"。这些痕迹最终会变成报告里的引文。

**截断机制**（第82-133行）：原始工具结果可能非常大（比如网页搜索返回几千字），ToolTrace 会自动截断到 50KB。截断时会智能处理 JSON 结构，优先保留关键字段（如 `answer`、`content`、`chunks`）。

### DynamicTopicQueue（动态话题队列）

**源码位置**：`data_structures.py` 第240-563行

这是研究 Agent 最独特的设计之一：**话题队列可以在研究过程中动态增长**。

```python
class DynamicTopicQueue:
    research_id: str         # 研究任务 ID
    blocks: list[TopicBlock] # 话题块列表
    block_counter: int       # 块编号计数器
    max_length: int | None   # 队列最大容量
```

**核心方法**：

| 方法 | 说明 |
|------|------|
| `add_block(title, overview)` | 在队尾添加新话题 |
| `append_child(parent, title, overview)` | 子话题追加（由 APPEND 触发） |
| `find_similar(title)` | 模糊匹配查重，防止重复话题 |
| `is_full()` | 检查队列是否已满 |
| `get_all_pending_blocks()` | 获取所有待研究的话题 |
| `mark_researching(id)` / `mark_completed(id)` / `mark_failed(id)` | 状态转换 |

**大白话解释**：DynamicTopicQueue 就像是一个"待办事项清单"，但有一个特殊能力——在研究过程中，AI 发现某个话题还需要深入探讨，可以往清单里追加新的子话题。这就是"动态"的含义。

---

## DeepResearchCapability — 能力入口

**源码位置**：`capability.py` 第37-99行

### 类定义

```python
class DeepResearchCapability(BaseCapability):
    manifest = CapabilityManifest(
        name="deep_research",
        description="Agentic-loop deep research with iterative report generation.",
        stages=["rephrasing", "decomposing", "researching", "reporting"],
        tools_used=["rag", "web_search", "paper_search", "code_execution"],
        cli_aliases=["research"],
    )
```

**基本信息**：
- **能力名称**：`deep_research`
- **命令行别名**：`research`
- **可用工具**：知识库检索、网络搜索、论文搜索、代码执行
- **阶段**：澄清需求 → 拆解话题 → 逐块研究 → 撰写报告

### run() — 两阶段调用模式

**源码位置**：第47-99行

研究 Agent 有一个独特的设计：**同一个能力可能被调用两次**。

```
第一次调用（无 confirmed_outline）：
  Phase 1 (澄清) → Phase 2 (拆解) → 返回大纲预览 → 等待用户确认

第二次调用（有 confirmed_outline）：
  跳过 Phase 1+2 → Phase 3 (研究) → Phase 4 (报告) → 返回完整报告
```

**大白话解释**：这就像调研公司先给客户看"我们打算从这几个方面调研，您看可以吗？"，客户确认后再开始实际调研和写报告。这样避免了 AI 花了大量时间研究用户不感兴趣的方向。

---

## ResearchPipeline — 四阶段管线

**源码位置**：`pipeline.py` 第267-1964行

### __init__() — 初始化

**源码位置**：第276-397行

**输入参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `language` | str | 语言 |
| `runtime_config` | dict | 运行时配置（包含模式、深度、策略参数） |
| `kb_name` | str | 知识库名称 |
| `enabled_tools` | list[str] | 启用的工具列表 |

**初始化做了什么**（大白话）：
1. 从 `runtime_config` 中读取各阶段的配置（最大迭代次数、并行数量、执行模式等）
2. 设置 LLM 客户端（模型、API 地址等）
3. 加载提示词模板
4. 设置温度为 0.3（比出题管线略低，研究需要更严谨）

### run() — 主入口

**源码位置**：第402-437行

**输入**：
- `context`：统一上下文
- `topic`：研究主题
- `confirmed_outline`：用户确认的子话题列表（第二次调用时传入）
- `attachments`：附件
- `stream`：事件流

**输出**：`dict` 包含 `response`（报告文本）、`metadata`（模式、话题数、引文数等）

---

## Phase 1: 需求澄清（Rephrase）

**源码位置**：`pipeline.py` 第622-703行

### 业务场景

用户说"帮我研究一下机器学习"——这个需求太模糊了。是研究机器学习的历史？算法对比？还是应用场景？Phase 1 的目标就是通过对话把模糊的需求变清晰。

### _rephrase() 方法

**源码位置**：第622-703行

**流程**：

```
AI 向用户提问（通过 ask_user 工具）
    ↓
用户回答
    ↓
AI 再提问（如果还需要澄清）
    ↓
用户再回答
    ↓
（最多 3 轮对话）
    ↓
AI 输出 FINISH：提炼后的研究主题
```

**关键参数**：
- `rephrase_max_rounds = 3`：最多和用户对话 3 轮
- `rephrase_max_questions_per_round = 3`：每轮最多问 3 个问题
- `rephrase_max_iterations = 8`：内层循环最多 8 步

**唯一可用工具**：`ask_user`。如果 AI 试图调用其他工具（如 `web_search`），会被拒绝并提示"这个阶段只能使用 ask_user"。

**轮次上限**：如果 3 轮对话后 AI 还没 FINISH，系统会强制要求"请立即用已有信息提炼研究主题"。

**容错处理**：如果整个澄清阶段失败（比如 `ask_user` 工具不可用），直接使用用户原始主题，不阻塞后续流程。

### _RephraseLoopHost

**源码位置**：第2573-2777行

这是驱动澄清循环的"主机"类。它负责：
- 限制工具调用仅为 `ask_user`
- 控制对话轮次上限
- 处理用户回复的暂停/恢复机制
- 将用户的回答注入到对话上下文中

---

## Phase 2: 话题拆解（Decompose）

**源码位置**：`pipeline.py` 第708-781行

### 业务场景

澄清后的研究主题是"对比监督学习、无监督学习和强化学习在图像识别中的应用"。Phase 2 把这个主题拆成若干子话题，每个子话题对应一个独立的"研究块"。

### _decompose() 方法

**源码位置**：第708-754行

**输入**：提炼后的研究主题

**输出**：`list[SubTopicItem]`，每个包含 `title` 和 `overview`

**流程**：
1. AI 输出一个 JSON 数组，每项包含 `title`（子话题标题）和 `overview`（简要描述）
2. 解析 JSON（宽容解析，支持多种格式）
3. 限制数量不超过 `initial_subtopics` 配置值
4. 如果解析失败，用原始主题作为唯一子话题（降级策略）

**子话题数量**（由研究深度决定）：

| 深度 | 自动模式 | 手动模式 |
|------|----------|----------|
| quick | 2 | 2 |
| standard | 4 | 3 |
| deep | 6 | 4 |

---

## Phase 3: 逐块研究（Research Blocks）

**源码位置**：`pipeline.py` 第786-1037行

### 业务场景

Phase 2 拆出了 4 个子话题。Phase 3 对每个子话题逐个进行深入研究——调用搜索工具、阅读资料、做笔记。

### _research_block() — 研究一个话题块

**源码位置**：第786-885行

**协议标签**：`THINK → TOOL → APPEND → FINISH`

- **THINK**：AI 的思考过程（中间步骤，显示在追踪面板）
- **TOOL**：调用工具（rag、web_search、paper_search、code_execution）
- **APPEND**：发现新话题，追加到队列
- **FINISH**：研究完成，输出总结

**输入**：
- 当前话题块（TopicBlock）
- 队列（可以追加新话题）
- 引文管理器（记录工具调用结果）
- 上下文

**输出**：`ResearchedBlock`（包含话题块 + 研究总结）

**关键机制**：

1. **工具白名单**（第100-102行）：研究阶段只允许 4 种工具：`rag`、`web_search`、`paper_search`、`code_execution`。聊天中的便利工具（如 `write_memory`、`ask_user`）在研究阶段不可用。

2. **强制使用工具**（第2337-2350行）：如果 AI 在第一轮就直接 FINISH 而没调用任何工具，系统会拒绝并提示"请先调用工具收集证据"。

3. **APPEND 动态追**（第2458-2565行）：AI 在研究过程中可以说"这个话题还需要深入，追加一个子话题 X"。`on_intermediate` 方法会：
   - 解析 AI 提出的新话题标题
   - 查重（模糊匹配，阈值 0.85）
   - 检查队列容量
   - 通过则追加到队尾，不通过则返回拒绝原因

### _drive_queue() — 队列调度器

**源码位置**：第975-1037行

**执行模式**：

| 模式 | 行为 |
|------|------|
| `series`（串行） | 一次只研究 1 个话题，完成后再研究下一个 |
| `parallel`（并行） | 一次最多同时研究 3 个话题（`max_parallel_topics`） |

**大白话解释**：串行模式像一个人逐个完成任务，适合快速研究。并行模式像多个人同时工作，适合深度研究，但需要更多资源。

**安全上限**：最多 20 轮调度（`queue_max_length * 4`），防止死循环。

### 工具结果摘要 + 引文记录

**源码位置**：`_summarise_and_record` 第2352-2415行

每当研究阶段调用了一个可引用的工具（rag、web_search、paper_search、code_execution），系统会：

1. 调用 Note Agent（`_summarise_tool_result`，第1042-1082行）把原始结果压缩成摘要
2. 生成引文 ID（如 `CIT-1-01`）
3. 创建 ToolTrace 记录，存入 CitationManager
4. 把工具消息的内容替换为 `[CIT-1-01] 摘要内容`

**大白话解释**：工具返回的原始结果可能几千字，直接堆在上下文里会撑爆。Note Agent 就像一个"速记员"，把长篇大论压缩成要点，同时保留引文 ID 以便报告阶段引用。

---

## Phase 4: 报告生成（Reporting）

**源码位置**：`pipeline.py` 第1093-1195行

### 业务场景

所有子话题都研究完了，现在要把研究成果整合成一篇结构化的报告。

### _write_report() — 报告撰写流程

**源码位置**：第1093-1195行

**报告结构**：

```
# 报告标题
## 1. Introduction（引言）
## 2. 子话题A（正文）
## 3. 子话题B（正文）
## 4. 子话题C（正文）
## N. Conclusion（结论）
## References（参考文献）
```

**每个子阶段**：

| 子阶段 | 协议标签 | 说明 |
|--------|----------|------|
| 报告大纲 | `OUTLINE` | AI 规划报告结构，每个 section 对应哪些研究块 |
| 引言 | `INTRO` | 介绍研究背景和范围 |
| 正文段落 | `SECTION` | 每个 section 对应一个或几个研究块，带引文 |
| 结论 | `CONCLUSION` | 总结研究发现 |
| 参考文献 | 无（纯代码生成） | 从引文管理器提取，编号后渲染 |

**流式输出**：每个 section 都在生成时实时流式推送到前端，用户可以看到报告"边写边显示"。

### 引文处理

**三个步骤**：

1. **规范化**（`_normalise_report_markdown`，第1262-1290行）：AI 在报告正文中写的引文标记（如 `[CIT-1-01]`）需要被识别和清理。如果 AI 错误地写了不存在的引文 ID，会被删除。

2. **链接化**（`_linkify_report_citations`，第1235-1260行）：把纯文本引文标记 `[CIT-1-01]` 转换成可点击的 Markdown 链接 `[1](#ref-cit-1-01)`，指向参考文献列表。

3. **编号**（`_citation_ids_in_first_appearance`，第2018-2031行）：引文按在报告中首次出现的顺序编号（1, 2, 3...），而不是按生成顺序。

### 报告大纲解析

**源码位置**：`_parse_report_outline` 第1384-1442行

**输入**：AI 输出的 JSON，包含 `title` 和 `sections` 列表

**输出**：`ReportOutline` 对象

**容错处理**（`_repair_report_section_coverage`，第1444-1505行）：
- 如果 AI 漏掉了某些研究块，自动分配到最相关的 section
- 如果 section 的 `block_ids` 为空，自动匹配最相似的研究块
- 如果还有遗漏，追加一个"Additional Findings"章节

### 参考文献渲染

**源码位置**：`_render_reference_list` 第1197-1233行

参考文献以 HTML `<details>` 折叠面板形式渲染，包含：
- 每条引文的编号
- 工具类型（RAG / Web Search / Paper Search）
- 查询语句
- 摘要
- 网页来源链接（如有）

---

## 研究模式与策略

**源码位置**：`mode_strategy.py` 第1-149行

### 四种研究模式

| 模式 | 风格 | 产出 | 适用场景 |
|------|------|------|----------|
| `notes` | 学习笔记 | 结构化笔记，含定义框、关键要点 | 学生学习某个概念 |
| `report` | 研究报告 | 正式报告，含引言、正文、结论 | 深度调研某个话题 |
| `comparison` | 对比分析 | 含对比表格的报告 | 比较多个方案/概念 |
| `learning_path` | 学习路径 | 含检查点的学习路径 | 规划学习路线 |

### ModeStrategy 策略参数

**源码位置**：第24-74行

每个模式都有独立的策略参数：

| 参数 | 说明 |
|------|------|
| `rephrase_enabled` | 是否启用需求澄清阶段 |
| `decompose_mode` | 话题拆解模式（auto=AI自动拆解，manual=手动指定） |
| `single_pass_threshold` | 单次研究阈值 |
| `min_section_length` | 报告每段最小长度 |
| `enable_citation_list` | 是否生成参考文献列表 |
| `enable_inline_citations` | 是否在正文中内联引用 |

### 输出校验

**源码位置**：`validate_output` 第76-95行

每种模式有特定的输出校验规则：

| 模式 | 校验规则 |
|------|----------|
| notes | 必须包含 Definition 框和 Key Takeaway 块 |
| comparison | 必须包含 Markdown 表格 |
| learning_path | 必须包含 Checkpoint 块 |

---

## 引文管理器

**源码位置**：`utils/citation_manager.py`

CitationManager 是研究阶段的"档案室"，负责：
- 存储所有工具调用的结果
- 为每条引文生成唯一 ID
- 格式化引文用于报告展示
- 支持按引用 ID 检索

---

## 动态话题队列

**源码位置**：`data_structures.py` 第240-563行

### 查重机制

**源码位置**：`find_similar` 第344-372行

当 AI 通过 APPEND 提出新话题时，系统会进行模糊匹配查重：

1. 标准化文本（去空格、小写）
2. 精确匹配 → 直接返回重复
3. 序列相似度（`difflib.SequenceMatcher`）
4. Token 重叠度（Jaccard 相似度 + 包含度）
5. 综合评分 ≥ 0.85 → 视为重复

**大白话解释**：这防止了 AI 用不同的措辞反复提出同一个话题。比如"监督学习算法"和"监督式学习方法"会被识别为重复。

### 队列容量

| 研究深度 | 队列最大容量 |
|----------|-------------|
| quick | 2 |
| standard | 5 |
| deep | 8 |
| manual | `subtopics + 2` |

---

## 总结

### 数据流全景图

```
用户输入："对比监督学习和无监督学习"
         │
         ▼
第一次调用（无 confirmed_outline）
         │
         ├─ Phase 1: 澄清
         │   AI 追问："你关注的是算法对比还是应用场景对比？"
         │   用户回答："算法对比"
         │   → 提炼主题："对比监督学习和无监督学习的核心算法"
         │
         ├─ Phase 2: 拆解
         │   AI 拆解为 4 个子话题：
         │   1. 监督学习核心算法
         │   2. 无监督学习核心算法
         │   3. 两类算法的数学基础对比
         │   4. 性能评估指标对比
         │   → 返回大纲预览，等待用户确认
         │
         ▼
用户确认大纲
         │
         ▼
第二次调用（有 confirmed_outline）
         │
         ├─ Phase 3: 逐块研究
         │   对每个子话题：
         │     AI 调用 web_search → Note Agent 摘要 → 存引文
         │     AI 调用 paper_search → Note Agent 摘要 → 存引文
         │     （AI 可能 APPEND 新话题 → 追加到队列）
         │     → FINISH：输出研究总结
         │
         ├─ Phase 4: 报告生成
         │   OUTLINE → INTRO → SECTION1 → SECTION2 → ... → CONCLUSION → REFERENCES
         │   （每段流式推送，实时显示）
         │
         ▼
最终输出：结构化报告 + 参考文献列表
```

### 关键设计决策

| 决策 | 原因 |
|------|------|
| 两阶段调用（预览→确认→执行） | 避免 AI 花大量时间研究用户不感兴趣的方向 |
| 动态话题队列（APPEND） | 研究过程中可能发现新的重要方向，不能丢 |
| 工具结果摘要（Note Agent） | 原始结果太长，需要压缩才能有效利用 |
| 引文自动编号（按首次出现） | 让报告引文顺序自然，符合学术习惯 |
| 串行/并行可切换 | 快速研究用串行，深度研究用并行 |
| 话题查重（模糊匹配） | 防止 AI 重复研究同一个话题 |
| 四种研究模式 | 不同场景需要不同的报告风格和深度 |
| 强制使用工具 | 研究不能空口无凭，必须基于实际检索 |
| 报告大纲自动修复 | AI 输出不完美时，通过代码逻辑补全，不阻塞流程 |

