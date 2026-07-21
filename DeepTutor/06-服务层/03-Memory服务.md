# Memory 服务

> Understand-Anything 知识图谱: 26 节点, 25 边, 7 层
> 源码：`deeptutor/services/memory/`（约 10 个文件，核心 ~1500 行）
> 复杂度：`complex`

## 一句话说明

Memory 服务是 DeepTutor 的"长期记忆系统"——它不只记录对话历史，还能自动筛选"什么值得长期记住"、把短期记忆整理成长期记忆、在不同会话之间共享重要信息。就像人脑的海马体，负责把短期记忆 consolidation 成长期记忆。

**类比**：普通聊天机器人像一个"健忘的朋友"，每次聊天都从零开始；DeepTutor 像一个"记性好的老师"，记得你上次哪里不会、你的学习偏好、甚至你喜欢的讲解风格。

## 为什么需要三级记忆

普通的"对话历史"有几个问题：
1. **越来越长**：对话越多，历史越长，LLM 上下文窗口有限，最老的内容会被挤出
2. **噪音太多**：99% 的对话内容不值得长期记住（"你好"、"谢谢"、"明白了"）
3. **无法跨会话**：换了会话，之前的记忆全丢了

三级记忆解决了这些问题：

| 层级 | 存储内容 | 生命周期 | 容量 |
|------|---------|---------|------|
| **L1（事件流）** | 所有原始对话事件 | 临时 | 无限（但按时间截断） |
| **L2（短期记忆）** | 最近的关键信息摘要 | 中期 | 按主题聚合 |
| **L3（长期记忆）** | 用户偏好、学习档案 | 永久 | 精心筛选 |

## 源码逐模块解析

### 1. `store.py` — MemoryStore 门面类

**做什么**：Memory 服务的统一入口，所有读写操作都经过这里。

**`MemoryStore` 类**（第 47 行）：
- 无状态设计（stateless），可以全局单例使用
- 用户隔离通过 `paths.memory_root` 实现（基于 ContextVar）

**核心方法**：

#### `emit()` — 写入事件（L1）
**输入**：TraceEvent（对话事件）
**输出**：无
**逻辑**：把事件追加到 L1 事件流文件（`trace.jsonl`）

#### `read_l2()` — 读取短期记忆
**输入**：surface 名称（如 "chat"、"solve"）
**输出**：Memory Document（Markdown 格式的记忆文档）
**逻辑**：读取 L2 目录下的对应文件

#### `read_l3()` — 读取长期记忆
**输入**：slot 名称（如 "preferences"、"profile"）
**输出**：Memory Document
**逻辑**：读取 L3 目录下的对应文件

#### `apply_ops()` — 批量写入/编辑
**输入**：操作列表（AddOp, EditOp, DeleteOp）
**逻辑**：原子操作，要么全部成功，要么全部回滚。带预校验（pre-flight validation）。

### 2. `consolidator.py` — 记忆整合器

**做什么**：把 L1 的原始事件整理成 L2 的摘要，再把 L2 的摘要提炼成 L3 的长期记忆。

**核心流程**：
1. 读取 L1 中未处理的事件
2. 调用 LLM 生成摘要（"这段对话的核心要点是什么？"）
3. 把摘要合并到 L2 文档
4. 定期检查 L2，把值得长期保留的内容提升到 L3

**`ConsolidateResult`**：整合结果，包含：
- 处理了多少事件
- 生成了什么摘要
- 更新了哪些 L2/L3 文件

### 3. `document.py` — 文档格式

**做什么**：定义 Memory 文档的存储格式。

**格式**：Markdown 文件，使用特殊的引用语法：
- `[ref:xxx]`：引用条目
- `[snapshot:xxx]`：快照引用
- 支持新旧两种格式兼容（ref-keyed 和 entry-keyed）

**函数**：
- `parse()`：解析 Memory Markdown 文件
- `serialize()`：把内存结构序列化为 Markdown

### 4. `ops.py` — 操作模型

**做什么**：定义 Memory 的增删改操作。

**操作类型**：
- `AddOp`：添加新条目
- `EditOp`：编辑已有条目
- `DeleteOp`：删除条目

**`apply()`**：原子批量执行操作，带预校验。

### 5. `paths.py` — 路径管理

**做什么**：管理 Memory 文件的存储路径。

**关键函数**：
- `memory_root()`：Memory 根目录（按用户隔离）
- `trace_dir()` / `trace_file()`：L1 事件流路径
- `l2_dir()` / `l2_file()`：L2 短期记忆路径
- `l3_dir()` / `l3_file()`：L3 长期记忆路径

### 6. `settings.py` — 配置

**做什么**：Memory 服务的配置参数。

**`MemorySettings`**：
- `chunking`：文本切块参数
- `dedup`：去重策略
- `merge`：合并策略
- `audit`：审计日志

## 调用链路示例

```
学生和 AI 对话中...
  ↓
Agent 决定记录重要信息
  ↓
MemoryStore.emit(event) → 写入 L1
  ↓
（后台定期任务）
Consolidator.consolidate()
  ↓
  读取 L1 新事件
  → LLM 生成摘要
  → 合并到 L2
  ↓
（进一步整合）
Consolidator 检查 L2 质量
  → 值得长期保留的 → 提升到 L3
```

```
下次学生问 "上次讲的那个公式是什么？"
  ↓
Agent 调用 ReadMemoryTool
  ↓
MemoryStore.read_l2("chat") → 读取 L2 摘要
  ↓
找到相关内容 → 作为上下文传给 LLM
  ↓
LLM 回答："上次我们讲的是 F=ma，牛顿第二定律..."
```

## 与其他服务的关系

- **Memory + LLM**：Consolidator 用 LLM 生成摘要
- **Memory + Agent**：Agent 通过 ReadMemoryTool / WriteMemoryTool 读写记忆
- **Memory + Session**：会话服务管理对话上下文，Memory 管理长期记忆

## 设计要点

1. **三级分层**：L1 全量、L2 摘要、L3 精华，容量递减但价值递增
2. **LLM 驱动的整合**：不是简单的规则，而是用 LLM 判断什么值得记
3. **原子操作**：批量写入要么全成功要么全失败，不会出现半成功状态
4. **用户隔离**：每个用户的 Memory 独立存储，通过 ContextVar 实现

### L1 层详解：事件流

L1 是原始事件日志，记录了 Agent 交互中发生的所有事件。每个事件包含：
- 事件类型（用户提问、AI 回答、工具调用、用户反馈等）
- 时间戳
- 事件内容
- 相关元数据

**存储方式**：按会话（Session）组织，JSONL 格式存储。每个会话一个日志文件。

**写入流程**：
1. 会话中发生事件 → AgentLoop 或 Tool 调用 `MemoryStore.emit()`
2. `MemoryStore` 将事件追加到当前会话的 trace 文件
3. 事件立即持久化到磁盘

**设计意图**：L1 不做任何加工，只是"原样记录"。这保证了数据完整性，后续可以随时基于原始数据重新分析。

### L2 层详解：会话摘要

L2 是 L1 的"压缩版"，由 Consolidator 定期从 L1 生成。

**Consolidator 工作流程**：
1. 扫描 L1 中未处理的事件（backlog）
2. 调用 LLM 对事件进行摘要
3. 提取关键信息：决策、知识点、偏好
4. 去重：与已有 L2 条目比较，过滤重复信息
5. 合并：将新条目整合到 L2 文档中
6. 更新 L2 文档

**L2 条目结构**：
- 摘要标题
- 详细内容
- 来源事件 ID
- 创建时间
- 更新时间
- 相关知识点

**Consolidator 配置**：
- `max_events_per_batch`：每次处理最多多少事件
- `min_backlog`：至少积累多少事件才触发
- `dedup_threshold`：去重相似度阈值
- `llm_model`：用于摘要的 LLM 模型

### L3 层详解：跨会话持久记忆

L3 是跨会话的持久记忆，按"表面"（Surface）组织。每个 Surface 是一个独立的记忆空间。

**Surface 类型**：
- `preferences`：用户偏好（语言、风格、难度等）
- `knowledge`：知识映射（用户已掌握的知识点）
- `profile`：用户画像（年龄、学习目标、水平等）
- `custom`：自定义 Surface

**L3 操作**：
- `read(surface)`：读取指定 Surface 的全部内容
- `write(surface, content)`：写入/覆盖指定 Surface
- `search(surface, query)`：语义搜索 Surface 中的内容
- `merge(surface, entries)`：合并新条目

**L3 与 L2 的关系**：
- L2 是阶段性的（一个会话周期内）
- L3 是跨会话的（永久保留）
- Consolidator 可以将 L2 中的重要发现提升到 L3
- 用户也可以手动编辑 L3 内容

### Memory 的代码实现

**MemoryStore 类**：
- 无状态门面（Facade）模式
- 所有调用者通过它访问 Memory 子系统
- 内部路由到 L1/L2/L3 的具体实现

**关键方法**：
- `emit(event)`：记录事件到 L1
- `read_l2(session_id)`：读取 L2 摘要
- `read_l3(surface)`：读取 L3 持久记忆
- `write_l3(surface, content)`：写入 L3
- `consolidate(session_id)`：触发 Consolidator
- `get_overview()`：获取记忆概览

**并发安全**：
- `MemoryStore` 使用 `asyncio.Lock` 保护写操作
- 每个 key 有独立的锁，避免不必要的阻塞
- 读操作无需锁（最终一致性）



### Memory 的使用场景

**场景1：学生跨会话问同样的问题**
- 会话1：学生问"什么是光合作用"，AI 详细解释
- 会话2（一周后）：学生又问"光合作用"，AI 从 L2/L3 记忆中找到之前的回答，直接引用

**场景2：AI 主动提醒**
- 学生上次说"周五前要交作业"
- 系统在 L3 preferences 中记录了这条信息
- 周五时 AI 主动提醒"你的作业完成了吗？"

**场景3：个性化教学**
- L3 记录了学生的薄弱知识点："二次函数"
- 新一轮对话中，AI 主动推荐二次函数的练习题
- 学生掌握了 → L3 更新掌握状态

### Memory 的隐私设计

- 所有记忆数据存储在用户本地
- L3 内容可由用户手动编辑或删除
- 支持"遗忘"操作：用户可以要求 AI 忘记某条信息
- 记忆数据不离开用户的服务器

### Memory 与其他模块的交互

| 交互方 | 交互内容 |
|--------|---------|
| AgentLoop | 调用 ReadMemoryTool / WriteMemoryTool 读写记忆 |
| Chat API | 返回记忆状态给前端 |
| Memory 设置面板 | 用户配置记忆策略 |
| Consolidator | 定期从 L1 生成 L2 |
| 学习调度器 | 读取 L3 中的掌握度信息 |


### Memory 性能优化

**L1 写入优化**：
- 使用追加写入（append-only），避免随机写
- 批量刷新到磁盘，减少 IO 次数
- 异步写入，不阻塞主流程

**L2 生成优化**：
- 增量处理：只处理新事件，不重复处理旧事件
- 批量调用 LLM：一次处理多个事件
- 缓存去重：避免重复生成相同摘要

**L3 检索优化**：
- 全量加载到内存（L3 通常很小）
- 按需加载 Surface
- 语义搜索使用轻量级 Embedding 模型

### Memory 子系统的可测试性

- 每个 Memory 层都有独立的单元测试
- Consolidator 支持 dry-run 模式（预览而不写入）
- Memory 操作支持回滚
- 提供测试夹具（fixture）快速创建测试数据

