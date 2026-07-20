# AgenticChatPipeline — 聊天能力组装与 AgentLoop 编排器

> 源码位置：`deeptutor/agents/chat/agentic_pipeline.py`（1386 行）
> 核心类：`AgenticChatPipeline`（第 181 行）
> 配合类：`AgentLoop`（`agent_loop.py` 第 149 行，667 行）

---

## 1. 一句话理解

如果把 DeepTutor 的一次对话比作一场演出，那么 AgenticChatPipeline 就是演出前的舞台总监——它负责搭好舞台（组装工具）、布置灯光（配置 LLM 参数）、安排演员（启动 AgentLoop），以及告诉演员可以演什么、不能演什么。而 AgentLoop 就是台上的主要演员，负责一板一眼地执行每一轮对话。

---

## 2. 模块定位：Pipeline 的总导演角色

从架构上看，AgenticChatPipeline 处于 Agent 层的核心位置，它是连接用户请求和 AI 回复之间的唯一编排入口。

用户请求 -> AgenticChatPipeline.run() -> AgentLoop.run() -> 用户看到回复
              |                          |
         组装工具、配置参数          执行 LLM 调用 + 工具分发

Pipeline 负责 5 件事：

| 职责 | 说明 |
|------|------|
| 工具组装 | 根据用户设置、上下文条件、能力开关，决定本轮可以调用哪些工具 |
| LLM 配置 | 确定用哪个模型、什么温度、多少 token 预算 |
| 提示词构建 | 把系统提示词、工具清单、知识库摘要、能力提示块拼成完整 prompt |
| AgentLoop 启动 | 创建 AgentLoop 实例并启动主循环 |
| Deferred 工具管理 | 管理 MCP（Model Context Protocol）工具的懒加载和权限控制 |

---

## 3. 构造函数：初始化时做了什么

AgenticChatPipeline.__init__() 位于源码第 184-257 行，主要做了以下初始化：

### 3.1 语言与 LLM 配置（第 186-206 行）

- self.language：语言支持中英文切换，默认跟随用户设置
- self.llm_config：LLM 配置从全局配置服务读取，包括模型名称、API Key、Base URL 等
- self.binding：标识底层 LLM 提供商类型（openai / anthropic / 其他）

### 3.2 工具注册表（第 201-204 行）

- self.registry：全局工具注册表，包含所有可用工具的定义
- self._usage：Token 用量追踪器，记录每轮 LLM 调用的输入/输出 token 数
- self._deferred_loader：延迟工具加载器，管理 MCP 动态工具

### 3.3 对话参数（第 207-231 行）

- self._max_rounds：最大循环轮数，默认 8 轮。防止 AI 无限循环调用工具
- self._exploring_max_tokens：探索阶段每轮 LLM 最大输出 token，默认 1600
- self._respond_max_tokens：回答阶段最大输出 token，默认 8000

### 3.4 提示词与 Client 配置（第 233-257 行）

- self._prompts：提示词从 agents/chat/prompts/zh/ 或 agents/chat/prompts/en/ 目录加载
- self._prompt_assembler：ChatPromptAssembler 负责把零散的提示词片段拼成完整的系统提示词

---

## 4. 核心方法：run() — 一次对话的完整编排

run() 方法位于第 301-321 行，是 Pipeline 的唯一公开入口。每一次用户发送消息，都会调用这个方法。

它分 7 步执行：

步骤 1：_prepare_deferred_tools() — 准备 MCP 动态工具（第 444-498 行）
步骤 2：_exec_allowed() — 检查代码执行权限（第 527-558 行）
步骤 3：_compose_enabled_tools() — 组装可用工具（第 560-595 行）
步骤 4：_can_use_native_tool_calling() — 判断是否使用原生工具调用
步骤 5：_build_llm_tool_schemas() — 构建 JSON Schema（第 669-708 行）
步骤 6：new AgentLoop() — 创建 AgentLoop 实例（第 313-320 行）
步骤 7：await loop.run() — 启动主循环

### 步骤 1：准备 Deferred 工具

_prepare_deferred_tools() 是整个 Pipeline 中最复杂的初始化步骤之一。它负责管理 MCP（Model Context Protocol）工具——这些工具不是写死在代码里的，而是通过 MCP 协议动态注册的。核心逻辑：

1. 启动 MCP 管理器，确保 MCP 服务正在运行
2. 根据用户权限确定可用工具白名单
3. 如果用户绑定了 PageIndex 知识库，会自动加载对应的 MCP 工具
4. 创建 DeferredToolLoader 实例，用于后续懒加载工具定义

权限控制分两层：调用方白名单（Partner 模式下的工具限制）和用户白名单（普通用户的权限授予文件）。

### 步骤 2：检查代码执行权限

_exec_allowed() 决定当前用户是否可以执行代码。它检查三层隔离策略：

| 隔离级别 | 行为 |
|---------|------|
| SYSTEM | 系统级隔离，管理员可为每个用户单独开关 exec |
| APPLICATION | 应用级隔离，只有管理员可用（或本地单用户模式） |
| 其他 | 默认禁止 exec |

### 步骤 3：工具组装 — 最核心的决策逻辑（第 560-595 行）

_compose_enabled_tools() 是 Pipeline 的大脑，它决定了本轮对话可以使用哪些工具。这个决策基于多个维度：

- 用户在设置中勾选的工具（requested_tools）
- 上下文条件标志（ToolMountFlags）：
  - has_kb：是否绑定了知识库 -> 开启 rag
  - has_memory：用户是否有记忆 -> 开启 read_memory
  - has_notebooks：用户是否有笔记本 -> 开启 notebook 工具
  - has_skills：是否有可用技能 -> 开启 read_skill
  - has_deferred_tools：是否有 MCP 工具 -> 开启 load_tools
  - has_exec：是否允许代码执行 -> 开启 exec
  - has_code：是否允许代码执行 -> 开启 code_execution
- 能力专属工具（capability_owned）：激活的能力会叠加自己的工具
- 独占模式（exclusive）：某些能力会替换整个工具表面
- Partner 模式：有特殊的工具替换逻辑（如 partner_read_memory 替代 read_memory）

工具组装的核心原则：

1. 条件挂载：工具不是始终可用的。例如，rag 工具只有在用户绑定了知识库时才会出现
2. 能力叠加：当用户激活了某个能力（如 Solve、Mastery），该能力的专属工具会叠加到基础工具集上
3. 独占模式：某些能力（如 Obsidian 知识库）会替换整个工具表面，而不是叠加
4. Partner 模式：Partner 有特殊的工具替换逻辑

---

## 5. 提示词构建：系统提示词是如何拼出来的

_build_system_prompt() 方法（第 325-343 行）和 _build_loop_messages() 方法（第 345-386 行）负责构造发送给 LLM 的完整消息列表。

系统提示词的组成部分：

1. ChatPromptAssembler 生成的基础提示词
2. 工具清单（tool_manifest）— 告诉 LLM 有哪些工具可用
3. 知识库提示（kb_note）— 告诉 LLM 用户挂载了哪些知识库
4. 延迟工具清单（deferred_tools_manifest）— MCP 动态工具列表
5. 笔记本清单（notebook_manifest）— 最多 30 个笔记本
6. 工作区提示（workspace_note）— 告诉 LLM 脚本文件应该写到哪里
7. 能力提示块（capability_blocks）— 激活的能力注入的提示词

### 知识库系统提示（第 1278-1295 行）

告诉 LLM：你现在可以访问这些知识库，调用 rag 工具时只能从中选择。同时也支持 PageIndex 引擎的知识库。

### 笔记本清单（第 712-727 行）

_build_notebook_manifest() 列出用户的所有笔记本（最多 30 个），格式如：
[用户的笔记本列表]
- nb_abc123 - 数学笔记 (12 records)
- nb_def456 - 物理笔记 (8 records)

### 工作区提示（第 1328-1360 行）

当 exec 可用时，系统提示词会包含工作区路径信息，告诉 LLM 脚本文件应该写到哪里，以及生成的文件如何呈现给用户。

---

## 6. 能力（Capability）系统：聊天 + 能力的灵活组合

### 什么是能力？

能力（Capability）是 DeepTutor 最独特的设计之一。它允许在不改变聊天对话流程的前提下，给 AI 添加额外的技能。就像给一个演员额外的剧本片段——他仍然在演同一出戏，但台词和动作更丰富了。

当前注册的能力有 5 个（capabilities/registry.py 第 13-19 行）：

| 能力 | 类名 | 说明 |
|------|------|------|
| 掌握度评估 | MasteryLoopCapability | 在学习模式下评估学生对知识点的掌握程度 |
| 解题 | SolveLoopCapability | 深度解题模式，支持多轮推理 |
| 知识库 | ObsidianCapability | 独占模式，只保留知识库相关工具 |
| 子代理 | SubagentCapability | 启动子代理处理复杂任务 |
| 上下文探索 | ExploreContextCapability | 在回答前先探索附加上下文 |

### 能力的激活与参与

Pipeline 通过 _active_loop_capabilities()（第 597-598 行）获取当前激活的能力，然后在以下环节参与：

1. 系统提示词：_capability_system_blocks()（第 616-626 行）收集每个能力的提示块
2. 工具注入：_capability_owned_tools()（第 609-614 行）收集能力的专属工具
3. 循环前种子：_capability_pre_loop_seed()（第 628-634 行）在用户消息中注入能力专属上下文
4. 循环前钩子：_capability_pre_loop_briefings()（第 636-667 行）执行异步预处理
5. 工具参数增强：augment_kwargs()（第 994-995 行）让能力注入私有参数

### 独占能力 vs 叠加能力

- 叠加能力（Solve、Mastery、Subagent、ExploreContext）：在聊天工具的基础上叠加自己的工具
- 独占能力（Obsidian）：完全替换工具表面，聊天工具不再可见

---

## 7. 工具执行：从 LLM 工具调用到实际执行

### 工具分发（第 790-831 行）

_dispatch_tool_calls() 是工具调用的分发中心。它接收 LLM 返回的工具调用列表，逐一执行。

### 参数增强器（第 886-996 行）

_augment_tool_kwargs() 是一个巨大的 switch 逻辑，为每个工具注入服务端私有参数：

| 工具 | 注入的参数 |
|------|-----------|
| rag | mode="hybrid" |
| load_tools | _tool_loader（延迟加载器引用） |
| exec | _sandbox_user_id, _sandbox_workdir, _sandbox_mounts |
| code_execution | 同上，使用 code_runs 子目录 |
| imagegen / videogen | _workspace_dir（媒体输出目录） |
| cron | _cron_owner（定时任务归属信息） |
| reason / brainstorm | context（用户消息作为上下文） |
| paper_search | max_results=3, years_limit=3, sort_by="relevance" |
| web_search | query（用户消息作为搜索词）, output_dir |
| write_note | conversation_history（对话历史）, current_user_message |
| geogebra_analysis | image_base64（从附件中提取）, language |

### ask_user 暂停与恢复（第 833-884 行）

_await_user_reply_and_resolve() 处理 ask_user 工具触发的暂停等待用户回复场景：
1. LLM 调用 ask_user 工具 -> 对话暂停
2. 系统等待用户回复（通过 wait_for_user_reply 回调）
3. 用户回复后，将回复内容注入回 role=tool 消息
4. AgentLoop 继续下一轮

---

## 8. 知识库种子（KB Seed）：回答前的知识预热

_retrieve_kb_seed_block() 方法（第 1041-1076 行）在 AgentLoop 启动前，预先从知识库中检索相关内容，作为种子注入到用户消息中。

这个设计很巧妙：不等 LLM 主动调用 rag 工具，而是先把相关内容喂给它。这样 LLM 在第一轮就能看到知识库内容，可以更快地给出准确回答。

每个知识库最多检索 4000 字符（KB_SEED_CHARS_PER_KB，第 95 行），总共最多 3 个知识库（KB_SEED_MAX_KBS，第 94 行）。

---

## 9. 上下文窗口保护（第 1178-1213 行）

_guard_context_window() 是一个安全阀。当消息列表的 token 数超过上下文窗口的 90%（CONTEXT_WINDOW_GUARD_RATIO = 0.9，第 99 行），它会将旧的工具结果替换为截断标记。

这样即使工具调用产生了大量结果，也不会导致上下文溢出。

---

## 10. Partner 模式：多用户协作的特殊处理

当 context.metadata["source"] == "partner" 时（第 442 行），Pipeline 会进入 Partner 模式：
- 工具权限不从 Partner 用户的 grant 文件读取，而是从 owner 的元数据白名单
- 内存工具替换为 partner_read_memory / partner_write_memory
- exec 权限继承 owner 的权限

---

## 11. Pipeline 与 AgentLoop 的协作关系

```
AgenticChatPipeline                  AgentLoop
─────────────────                    ─────────
run()                                run()
  │                                    │
  ├─ _prepare_deferred_tools()         │
  ├─ _compose_enabled_tools()          │
  ├─ _build_llm_tool_schemas()         │
  │                                    │
  ├─ new AgentLoop(pipeline=self) ──→  │
  │                                    ├─ _capability_pre_loop_briefings()
  │  ←── 回调 ── _retrieve_kb_seed_block()
  │  ←── 回调 ── _build_loop_messages()
  │                                    ├─ _run_loop()
  │                                    │   ├─ _call_llm()
  │  ←── 回调 ── _guard_context_window()
  │                                    │   ├─ 如果有 tool_calls:
  │  ←── 回调 ── _dispatch_tool_calls()
  │  ←── 回调 ── _augment_tool_kwargs()
  │                                    │   ├─ 如果 ask_user:
  │  ←── 回调 ── _await_user_reply_and_resolve()
  │                                    │   └─ 如果没有 tool_calls: finish
  │                                    └─ emit_capability_result()
  └─ 完成
```

---

## 12. 关键设计决策与权衡

| 决策 | 为什么这么做 | 代价 |
|------|------------|------|
| 单 Pipeline 多能力 | 代码复用，聊天流程统一 | Pipeline 类变得很复杂（1386 行） |
| 工具条件挂载 | 避免 LLM 看到不可用的工具 | 挂载逻辑分散在多个地方 |
| KB Seed 预检索 | 加速第一轮回答 | 增加了一次额外的 RAG 调用 |
| Deferred 工具懒加载 | 支持 MCP 动态工具注册 | 增加了初始化复杂度 |
| 上下文窗口保护 | 防止 token 溢出 | 可能丢失重要上下文 |

---

## 13. 总结

AgenticChatPipeline 是 DeepTutor 聊天的总控中心。它把用什么模型、配什么工具、给什么提示词、允许多少轮这些决策集中在一处，然后交给 AgentLoop 去执行。它通过能力系统实现了灵活的功能扩展，通过工具组装策略实现了精细的权限控制，通过 KB Seed 和上下文窗口保护保证了对话质量。

如果你想深入理解一次对话的完整生命周期，建议按以下顺序阅读源码：
1. run() -> 了解整体流程
2. _compose_enabled_tools() -> 理解工具如何被选中
3. _build_loop_messages() -> 理解提示词如何构建
4. agent_loop.py 的 _run_loop() -> 理解 LLM 调用和工具分发
5. _augment_tool_kwargs() -> 理解每个工具的参数注入
