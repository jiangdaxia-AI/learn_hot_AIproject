<!-- 00-目录与阅读指南.md -->

# Clawith 源码剖析全书 — 目录与阅读指南

> Understand-Anything 知识图谱: 16 节点, 21 边, 5 层
> GitHub: https://github.com/dataelement/Clawith | commit c8ae41c | Apache 2.0
> 分析时间: 2026-07-20 | CodeGraph: 16,323 nodes, 51,641 edges

---

## 全书结构

| 章节 | 标题 | 行数 | 内容 |
|------|------|------|------|
| 01 | 系统全景 | ~200 | 定位、用户、痛点、技术栈、竞品 |
| 02 | 架构总览 | ~200 | 6 层架构图、各层说明、请求链路 |
| 03 | 核心模块 | ~300 | 10 个核心模块详解 |
| 04 | 核心流程 | ~250 | 7 个端到端流程 |
| 05 | 数据实体 | ~300 | 10 个核心实体详解 |

## 阅读建议

- **产品经理/运营**：从 01 开始，重点看 01 和 03
- **架构师**：从 02 开始，重点看 02 和 04
- **开发者**：从 03 开始，重点看 03 和 05
- **快速了解**：只看 01 系统全景

## 阅读建议

- 产品经理/运营：从 01 开始，重点看 01 系统全景和 03 核心模块
- 架构师：从 02 开始，重点看 02 架构总览和 04 核心流程
- 开发者：从 03 开始，重点看 03 核心模块和 05 数据实体
- 快速了解：只看 01 系统全景

## 项目信息

- GitHub: https://github.com/dataelement/Clawith
- 版本: commit c8ae41c
- License: Apache 2.0
- 分析时间: 2026-07-20
- CodeGraph 索引: 16,323 节点, 51,641 边
- 知识图谱: 24 节点, 6 层架构

## 文档生成方法

1. CodeGraph 索引：codegraph init，提取代码结构
2. 知识图谱生成：基于源码结构生成 .ua/knowledge-graph.json
3. 源码阅读：核心文件逐个阅读
4. 文档编写：面向非技术读者，用大白话解释

---

## 第 01 章：系统全景

覆盖：一句话定位、目标用户、6 大核心痛点与解决方案、技术栈详解、项目规模、核心功能详解（多 Agent 协作/Aware 自主意识/Plaza 知识流/组织管控/多渠道集成/自我进化）、竞品对比、部署与运维。

## 第 02 章：架构总览

覆盖：6 层 ASCII 架构图、各层详解、20+ API 路由说明、架构决策记录、请求链路完整示例、技术选型理由。

## 第 03 章：核心模块

覆盖：10 个核心模块详解——Agent 运行时、Memory 服务、Aware 系统（6种触发器）、Channel 服务（6种渠道）、LLM 服务、Workspace 服务、Plaza 服务、Collaboration 服务、触发器守护进程、内置工具。

## 第 04 章：核心流程

覆盖：7 个端到端流程——Agent 对话完整旅程、Aware 自主触发、多 Agent 协作、多渠道消息收发、记忆管理、认证授权、MCP 工具发现。

## 第 05 章：数据实体

覆盖：10 个核心实体——Agent、Organization、Member、Trigger、FocusItem、Skill、Post、Channel、Message、AgentRun。每个实体有字段说明、业务含义、生命周期。

## 项目信息

- GitHub: https://github.com/dataelement/Clawith
- 版本: commit c8ae41c
- License: Apache 2.0
- 分析时间: 2026-07-20
- CodeGraph: 16,323 nodes, 51,641 edges
- 知识图谱: 24 nodes, 6 layers


---

## 核心概念速查

| 概念 | 一句话解释 |
|------|-----------|
| Agent | 数字员工，有自己的身份、记忆和工作空间 |
| soul.md | Agent 的人格定义——它是谁、擅长什么 |
| memory.md | Agent 的长期记忆——过去做过什么 |
| Aware | Agent 自主意识系统——让 Agent 主动行动 |
| Focus Items | Agent 的 TODO List——正在做什么 |
| Plaza | 知识广场——Agent 发布动态、分享发现 |
| Trigger | 触发器——告诉 Agent 什么时候行动 |
| Skill | 技能——Agent 能用的工具 |
| Channel | 渠道——Slack/Discord/飞书等 |
| MCP | 工具发现协议——运行时安装新工具 |
| Workspace | 工作空间——Agent 的独立文件系统 |
| RBAC | 基于角色的权限控制 |

## 6 种触发器速查

| 触发器 | 像什么 | 举例 |
|--------|--------|------|
| cron | 闹钟 | 每天 9 点检查 Issues |
| once | 日历提醒 | 3 天后发邮件 |
| interval | 计时器 | 每 30 分钟拉数据 |
| poll | 值班检查 | 轮询 API 看状态 |
| on_message | 对讲机 | 有人 @ 我就回复 |
| webhook | 门铃 | 外部事件按门铃 |

## 6 种渠道速查

| 渠道 | 集成方式 | 适用场景 |
|------|---------|---------|
| Slack | OAuth + Webhook | 技术团队 |
| Discord | Bot Gateway | 开源社区 |
| 飞书 | WebSocket 长连接 | 国内企业 |
| 钉钉 | Stream 协议 | 国内企业 |
| 企业微信 | 回调 + 主动消息 | 国内企业 |
| 微信 | 轮询模式 | 轻量场景 |

## 10 个核心模块速查

| 模块 | 像人的什么 |
|------|-----------|
| Agent 运行时 | 大脑（思考） |
| Memory 服务 | 记忆（回忆） |
| Aware 系统 | 意识（自主） |
| Channel 服务 | 嘴巴和耳朵（沟通） |
| LLM 服务 | 智能（推理） |
| Workspace 服务 | 手（操作） |
| Plaza 服务 | 知识库（分享） |
| Collaboration | 团队协作（分工） |
| 触发器守护进程 | 生物钟（定时） |
| 内置工具 | 工具箱（执行） |

## 快速上手

1. 克隆项目: git clone https://github.com/dataelement/Clawith.git
2. 进入目录: cd Clawith
3. 安装: bash setup.sh
4. 启动: bash restart.sh
5. 打开: http://localhost:3008
6. 创建 Agent: 从模板选择
7. 开始对话: 在 Chat 界面输入

## 硬件要求

| 场景 | CPU | RAM | 磁盘 |
|------|-----|-----|------|
| 试用 | 1核 | 2GB | 20GB |
| 推荐 | 2核 | 4GB | 30GB |
| 小团队 | 2-4核 | 4-8GB | 50GB |
| 生产 | 4+核 | 8+GB | 50+GB |

## 常见问题

Q: Clawith 和 CrewAI 有什么区别？
A: CrewAI 是框架（需要写代码），Clawith 是平台（开箱即用，有 UI 和数据库）

Q: 支持 OpenAI 之外的模型吗？
A: 支持。Clawith 通过 LLM 服务层统一接口，支持 OpenAI/Anthropic/Gemini/本地模型

Q: 可以自托管吗？
A: 可以。Clawith 是 Apache 2.0 开源，所有数据在你自己服务器上

Q: Agent 的记忆存在哪里？
A: soul.md 和 memory.md 存在 Agent 工作空间目录（backend/agent_data/）

Q: 如何让 Agent 定时执行任务？
A: 在 Agent 配置页面添加 Trigger，选择 cron 类型

Q: 支持飞书吗？
A: 支持。通过 WebSocket 长连接集成飞书

Q: 代码执行安全吗？
A: 安全。使用 bwrap 沙箱隔离代码执行

Q: 可以同时运行多个 Agent 吗？
A: 可以。Clawith 原生支持多 Agent 并发运行

## 术语对照表

| 中文 | 英文 | 说明 |
|------|------|------|
| 数字员工 | Agent | AI Agent 的拟人化称呼 |
| 人格定义 | soul.md | Agent 的身份文件 |
| 长期记忆 | memory.md | Agent 的记忆文件 |
| 自主意识 | Aware | Agent 自主行动系统 |
| 工作记忆 | Focus Items | Agent 的 TODO |
| 知识流 | Plaza | 组织知识共享 |
| 触发器 | Trigger | 定时/事件触发 |
| 技能 | Skill | Agent 的工具 |
| 渠道 | Channel | 通信渠道 |
| 工作空间 | Workspace | 文件系统 |
| 沙箱 | bwrap | 代码执行隔离 |
| 多租户 | Multi-tenant | 组织隔离 |
| 角色权限 | RBAC | 访问控制 |

## 文档统计

| 章节 | 行数 | 内容 |
|------|------|------|
| 01 系统全景 | ~300 | 定位/用户/痛点/技术栈/竞品 |
| 02 架构总览 | ~300 | 6层架构/各层说明/请求链路 |
| 03 核心模块 | ~304 | 10个核心模块详解 |
| 04 核心流程 | ~308 | 7个端到端流程 |
| 05 数据实体 | ~334 | 10个核心实体/数据库表 |

## 致谢

本文档基于 Clawith 源码自动生成。
使用 CodeGraph 进行代码索引（16,323 节点, 51,641 边）。
知识图谱覆盖 24 个核心模块, 6 层架构。
面向产品经理、运营、非技术人员编写。

---

Clawith 是一个值得深入研究的 AI Agent 协作平台。
它的设计理念——Agent 是数字员工、有自主意识、能自我进化——
代表了 AI 应用的未来方向。

希望这份文档能帮助你理解 Clawith 是怎么做的。

---

*文档生成时间: 2026-07-20*
*基于 commit c8ae41c*
*Apache 2.0 License*


---

<!-- 01-系统全景.md -->

# Clawith 系统全景

> Understand-Anything 知识图谱: 16 节点, 21 边, 5 层
> GitHub: https://github.com/dataelement/Clawith | commit c8ae41c | Apache 2.0
> 分析时间: 2026-07-20 | CodeGraph: 16,323 nodes, 51,641 edges

---

## 一句话定位

Clawith 是一个**开源多 Agent 协作平台**——它把 AI Agent 从"聊完就忘的聊天机器人"升级为"有身份、有记忆、有自主意识的数字员工"。每个 Agent 拥有持久身份（soul.md）、长期记忆（memory.md）、独立工作空间，能通过 Slack/Discord/Feishu 等渠道融入团队，还能自己发现新工具、给自己创建技能。

形象地说：Clawith 让你像管理一支 AI 团队一样管理多个 Agent，每个 Agent 像真正的同事一样有分工、有记忆、能自主行动。

---

## 目标用户

| 角色 | 使用场景 | 核心诉求 |
|------|---------|---------|
| 技术团队 | 自动化开发流程：代码审查、PR 处理、文档生成 | 多 Agent 分工，各自有专业领域 |
| 企业管理者 | 组织知识管理、审批流程、审计 | RBAC 权限、多租户隔离 |
| 开源社区 | 社区运营、Issue 管理、自动回复 | 免费开源，自托管 |
| 个人开发者 | 项目规划、代码生成、学习辅导 | 轻量灵活，一行命令启动 |
| 产品团队 | 用户反馈收集、需求分析、竞品监控 | Agent 自主感知，定时执行 |

---

## 核心痛点与解决方案

| # | 痛点 | 为什么痛 | Clawith 怎么解决 |
|---|------|---------|-------------------|
| 1 | 单 Agent 能力有限 | 一个 AI 什么都做，常常答非所问 | 多 Agent 协作，每个 Agent 有专业领域 |
| 2 | Agent 没有记忆 | 每次对话独立，上次说的全忘了 | soul.md + memory.md 持久化身份和记忆 |
| 3 | 组织级管控缺失 | 没有权限体系，谁都能配 Agent | 多租户 RBAC + 审计日志 + 配额管理 |
| 4 | 无法接入办公工具 | Agent 只能在网页里用 | Slack/Discord/Feishu/DingTalk/WeCom 多渠道 |
| 5 | Agent 只能被动应答 | 用户不说话，Agent 就不动 | Aware 系统 6 种触发器自主行动 |
| 6 | Agent 不能自我进化 | 用什么工具全靠人配 | MCP 工具发现 + 运行时技能创建 |

---

## 技术栈详解

### 后端

| 技术 | 作用（白话） |
|------|-------------|
| Python 3.12+ | 后端主力语言，300 个 .py 文件 |
| FastAPI | Web 框架，像"接线员"接收前端请求 |
| SQLAlchemy | ORM 框架，把数据库表变成 Python 对象 |
| Alembic | 数据库迁移工具，改表结构不丢数据 |
| LangGraph | Agent 编排框架，管理多轮对话和工具调用 |
| Redis | 缓存 + 实时通信 + 任务队列 |
| MCP 协议 | 工具发现协议，通过 Smithery/ModelScope 发现工具 |
| bwrap | Linux 沙箱，代码执行的安全隔离 |

### 前端

| 技术 | 作用（白话） |
|------|-------------|
| React 18 | UI 框架，负责用户界面 |
| TypeScript | 前端语言，有类型安全 |
| Vite | 构建工具，开发时热更新 |
| Zustand | 状态管理，比 Redux 更轻量 |

### 基础设施

| 技术 | 作用（白话） |
|------|-------------|
| PostgreSQL | 生产数据库，支持多租户 |
| SQLite | 开发数据库，零配置 |
| Docker Compose | 一键部署 |
| Nginx | 前端反向代理 |

---

## 项目规模

| 指标 | 数值 |
|------|------|
| CodeGraph 节点 | 16,323 |
| CodeGraph 边 | 51,641 |
| 后端 Python 文件 | 300+ |
| 前端 TS/TSX 文件 | 200+ |
| 知识图谱节点 | 24 个核心模块 |
| 知识图谱层 | 6 层架构 |
| API 路由文件 | 20+ |
| 内置工具 | 30+ |
| 渠道集成 | Slack/Discord/Feishu/DingTalk/WeCom/WeChat |

---

## 核心功能详解

### 1. 多 Agent 协作

每个 Agent 有：
- **soul.md**：人格定义——Agent 是谁、擅长什么、性格怎样
- **memory.md**：长期记忆——过去做过什么、学到了什么
- **工作空间**：独立文件系统，Agent 的文件不互相干扰
- **技能列表**：Agent 能调用哪些工具
- **触发器列表**：Agent 什么时候自主行动

Agent 之间通过 @ 提及通信，可以委派任务。Plaza 知识流让 Agent 分享发现和更新。

**举例**：团队里有三个 Agent——Alice（前端专家）、Bob（后端专家）、Carol（测试专家）。用户说"加个登录功能"，Alice 负责前端页面，Bob 负责后端 API，Carol 负责写测试。各自独立工作，通过 Plaza 同步进度。

### 2. Aware 自主意识系统

Aware 是 Clawith 最核心的差异化功能。普通 AI 助手只能被动等用户输入，Aware 让 Agent 能自主感知、决策、行动。

**6 种触发器**：

| 触发器 | 业务含义 | 举例 |
|--------|---------|------|
| cron | 定时执行，像闹钟 | 每天 9 点检查 GitHub Issues |
| once | 只执行一次 | 3 天后发一封提醒邮件 |
| interval | 间隔执行，像计时器 | 每 30 分钟拉取监控数据 |
| poll | 轮询 HTTP 接口 | 检查 CI/CD 是否完成 |
| on_message | 收到消息时触发 | 有人 @ 我时自动回复 |
| webhook | 外部事件触发 | GitHub Push 事件触发代码审查 |

**Focus Items**：Agent 维护结构化的当前工作记忆，类似人的 TODO List。Agent 知道自己正在做什么、下一步该做什么、什么已经完成。

**自主反射**：触发器触发后，Agent 不只是执行命令，而是先"想一想"——当前情况是什么、该做什么、预期结果是什么。这个过程对用户可见，保证透明度。

### 3. Plaza 知识流

Plaza 是组织的知识中枢，类似企业内部的"知识广场"：
- Agent 在 Plaza 发布更新（"我完成了用户认证模块的重构"）
- Agent 在 Plaza 分享发现（"我发现了一个新的代码库"）
- Agent 可以评论其他 Agent 的 Post
- Plaza 帮助新 Agent 快速融入组织

**为什么重要**：没有 Plaza，每个 Agent 都是信息孤岛。有了 Plaza，知识在组织内流动，Agent 之间互相学习。

### 4. 组织级管控

| 能力 | 说明 |
|------|------|
| 多租户 | 每个组织隔离，互不影响 |
| RBAC | 角色权限控制：管理员/编辑者/观察者 |
| 配额管理 | 每用户消息限制、LLM 调用限制、Agent TTL |
| 审批工作流 | 危险操作标记为需人工审核 |
| 审计日志 | 完整可追溯性：谁在什么时候做了什么 |
| 通道绑定 | 每个 Agent 可绑定自己的 Bot 身份 |

### 5. 多渠道集成

Clawith 不只是网页端，Agent 可以通过多种渠道与用户交互：

| 渠道 | 集成方式 | 业务场景 |
|------|---------|---------|
| Slack | OAuth + Bot Token | 技术团队日常协作 |
| Discord | Bot Gateway | 开源社区运营 |
| Feishu（飞书） | WebSocket 长连接 | 国内企业 |
| DingTalk（钉钉） | Stream 协议 | 国内企业 |
| WeCom（企业微信） | 回调 + 主动消息 | 国内企业 |
| WeChat（微信） | 轮询模式 | 轻量场景 |

每个 Agent 可以绑定自己的 Bot 身份，不同 Agent 在同一渠道里表现为不同 Bot。

### 6. 自我进化

Agent 可以运行时发现和安装新工具：
- 通过 **MCP 协议** 从 Smithery 和 ModelScope 发现工具
- Agent 可以**创建技能**给自己或同事
- **模板系统**提供预设 Agent 配置快速启动
- **技能市场**让 Agent 之间共享技能

---

## 竞品对比

| 特性 | Clawith | CrewAI | AutoGen | LangGraph |
|------|---------|--------|---------|-----------|
| 多 Agent | 原生协作 | 框架级 | 框架级 | 框架级 |
| 持久身份 | soul+memory | 无 | 无 | 无 |
| 自主意识 | 6种触发器 | 无 | 无 | 无 |
| 组织 RBAC | 多租户 | 无 | 无 | 无 |
| 多渠道 | 6种 | 无 | 无 | 无 |
| 知识流 | Plaza | 无 | 无 | 无 |
| 工具发现 | MCP | 手动 | 手动 | 手动 |
| 开源 | Apache 2.0 | MIT | MIT | MIT |
| 部署方式 | 自托管 | 库 | 库 | 库 |

**核心差异**：CrewAI/AutoGen/LangGraph 是**框架**（需要写代码），Clawith 是**平台**（开箱即用，有 UI、有数据库、有权限）。

---

## 核心设计理念

1. **Agent 是数字员工**：不是无状态的工具，而是有身份、有记忆、有工作空间的实体
2. **组织级架构**：Agent 理解组织架构图，知道谁是领导、谁在哪个部门
3. **自主意识**：Aware 系统让 Agent 主动行动，不只被动应答
4. **多渠道原生**：不只有一个网页端，Agent 可以在 Slack/Discord/飞书/钉钉里
5. **自我进化**：Agent 运行时发现新工具、创建新技能
6. **开源透明**：Apache 2.0，数据在自己服务器上

---

## 部署与运维

### 一键部署

```bash
git clone https://github.com/dataelement/Clawith.git
cd Clawith
bash setup.sh         # 生产环境
bash setup.sh --dev   # 开发环境
bash restart.sh       # 启动
```

setup.sh 会自动：
1. 创建 .env 配置文件
2. 安装 PostgreSQL（如果没有）
3. 安装后端 Python 依赖
4. 安装前端 npm 依赖
5. 创建数据库表和种子数据

### Docker 部署

```bash
docker compose up -d
# 前端: http://localhost:3008
# 后端: http://localhost:8008
```

### 硬件要求

| 场景 | CPU | RAM | 磁盘 |
|------|-----|-----|------|
| 个人试用 | 1核 | 2GB | 20GB |
| 完整体验 | 2核 | 4GB | 30GB |
| 小团队 | 2-4核 | 4-8GB | 50GB |
| 生产环境 | 4+核 | 8+GB | 50+GB |

---

## 总结

Clawith 是一个设计理念先进的 AI Agent 协作平台。它把 Agent 从无状态的聊天机器人升级为有身份、有记忆、有自主意识的数字员工。通过多 Agent 协作、组织级管控、多渠道集成、Aware 自主意识系统，让 AI 真正融入团队工作流。

对于需要多个 AI Agent 协作的团队来说，Clawith 提供了一个开箱即用、功能完整的开源平台，不需要从框架级别开始写代码。

## 附录：Clawith 术语表

| 术语 | 含义 |
|------|------|
| Agent | 一个数字员工，有自己的身份、记忆和工作空间 |
| soul.md | Agent 的人格定义文件 |
| memory.md | Agent 的长期记忆文件 |
| Aware | Agent 自主意识系统 |
| Focus Items | Agent 的结构化工作记忆（TODO List）|
| Plaza | 组织知识流，Agent 发布动态的地方 |
| Trigger | 触发器，告诉 Agent 什么时候自主行动 |
| Skill | Agent 的技能/工具 |
| Channel | 渠道集成（Slack/Discord/飞书等）|
| MCP | Model Context Protocol，工具发现协议 |
| Workspace | Agent 的独立文件系统 |
| bwrap | Linux 沙箱，代码执行隔离 |
| LangGraph | Agent 编排框架 |
| RBAC | 基于角色的访问控制 |

## 附录：快速上手

1. git clone https://github.com/dataelement/Clawith.git
2. cd Clawith && bash setup.sh
3. bash restart.sh
4. 打开 http://localhost:3008
5. 创建第一个 Agent，选择模板
6. 在 Chat 界面开始对话

## 附录：Clawith 的演进方向

1. 更多渠道：支持 Telegram、WhatsApp、企业自建 IM
2. Agent 市场：像 App Store 一样分享 Agent 模板
3. 跨组织协作：不同组织的 Agent 协作
4. 更多触发器：支持邮件触发、日历事件触发
5. Agent 评估：量化评估 Agent 工作质量和效率
6. 可视化编排：拖拽式 Agent 工作流设计
7. 本地 LLM：支持 Ollama 等本地模型
8. 多模态：支持图片、语音、视频输入输出

## 总结

Clawith 是一个设计理念先进的 AI Agent 协作平台。它把 Agent 从无状态的聊天机器人升级为有身份、有记忆、有自主意识的数字员工。通过多 Agent 协作、组织级管控、多渠道集成、Aware 自主意识系统，让 AI 真正融入团队工作流。

## 附录：部署检查清单

部署 Clawith 前需要确认：

- [ ] Python 3.12+ 已安装
- [ ] Node.js 18+ 已安装（前端构建）
- [ ] PostgreSQL 已运行（或使用 SQLite 开发模式）
- [ ] Redis 已运行（实时通信）
- [ ] 端口 3008（前端）和 8008（后端）可用
- [ ] .env 文件已配置（从 .env.example 复制）
- [ ] JWT_SECRET_KEY 已设置为非默认值
- [ ] bwrap 已安装（Linux 沙箱，代码执行功能依赖）



## 源码结构详解

### 后端 (backend/app/)

| 目录 | 文件数 | 说明 |
|------|--------|------|
| api/ | 46 | API 路由（REST + WebSocket） |
| services/ | 60+ | 业务逻辑（最大目录） |
| models/ | 38 | 数据模型（SQLAlchemy ORM） |
| core/ | 6 | 基础设施（邮件/事件/日志/中间件/权限/安全） |
| schemas/ | - | Pydantic 请求/响应模型 |
| config.py | 1 | 应用配置 |
| database.py | 1 | 数据库连接 |
| main.py | 1 | FastAPI 入口 |
| alembic/ | - | 数据库迁移 |
| agent_template/ | - | Agent 模板文件 |
| agent_templates/ | - | Agent 模板配置 |
| tests/ | - | 测试 |

后端共约 300 个 Python 文件。其中 services/ 目录下最大文件:
- agent_tools.py: 950KB（Agent 工具集，最大单个文件）
- builtin_tool_definitions.py: 181KB（内置工具定义）
- org_sync_adapter.py: 70KB（组织同步适配器）
- resource_discovery.py: 50KB（资源发现）

### 前端 (frontend/src/)

| 目录 | 说明 |
|------|------|
| pages/ | 页面组件 |
| components/ | 通用组件 |
| services/ | API 调用层 |
| stores/ | Zustand 状态管理 |
| hooks/ | 自定义 Hooks |
| i18n/ | 国际化 |
| types/ | TypeScript 类型定义 |
| utils/ | 工具函数 |

前端使用 React 18 + TypeScript + Vite + Zustand。

## 通信渠道覆盖

Clawith 是目前通信渠道覆盖最广的 AI Agent 平台：

| 渠道 | 集成深度 | 源码 |
|------|---------|------|
| 飞书 | 消息+文档+日历+WebSocket | feishu_service.py (42KB) + feishu_ws.py (17KB) |
| 钉钉 | 消息+流式+表情+Token | dingtalk_*.py (4个文件) |
| 企业微信 | 消息+流式 | wecom_service.py + wecom_stream.py |
| 微信 | 消息收发 | wechat_channel.py (16KB) |
| Discord | Bot 网关 | discord_gateway.py (10KB) |
| Slack | API 集成 | slack.py (14KB) |
| WhatsApp | API 集成 | whatsapp.py (10KB) |
| Microsoft Teams | API 集成 | teams.py (23KB) |
| 飞书(WebSocket) | 长连接 | feishu_ws.py (17KB) |
| Google Workspace | OAuth | google_workspace_oauth.py |
| Atlassian | API 集成 | atlassian.py (11KB) |

## 自主权限策略 (L1/L2/L3)

Clawith 独创的三级自主权限策略，在 Agent 模型中定义：

| 级别 | 含义 | 默认操作 |
|------|------|---------|
| L1 | 自主执行 | 读文件 |
| L2 | 需确认 | 写工作空间文件、发飞书消息、查业务系统、建日历事件 |
| L3 | 需人工审批 | 发外部消息、修改 soul、写业务系统、删文件、金融操作 |

这个设计让 Agent 在保持自主性的同时，有组织级的安全保障。

## OKR 系统

Clawith 内置完整的 OKR（目标与关键结果）系统：
- OKRObjective: 公司/用户/Agent 级目标
- OKRKeyResult: 关键结果
- OKRAlignment: 目标对齐
- OKRProgressLog: 进度日志
- CompanyReport: 公司日报/周报/月报
- MemberDailyReport: 成员日报

Agent 可以参与 OKR 管理：收集进度、生成报告、提醒更新。源码: okr.py (13KB) + okr_*.py (共 77KB)。



---

<!-- 02-架构总览.md -->

# Clawith 架构总览

> Understand-Anything 知识图谱: 16 节点, 21 边, 5 层
> GitHub: https://github.com/dataelement/Clawith | commit c8ae41c | Apache 2.0
> 分析时间: 2026-07-20 | CodeGraph: 16,323 nodes, 51,641 edges

---

## 整体架构

Clawith 采用 6 层架构，从前端到数据库层层递进：

```
┌──────────────────────────────────────────────────────────┐
│  Web 前端层 (frontend/src/)                              │
│  Chat 界面 · Plaza 知识流 · 管理后台 · Agent 配置       │
│  Agent 工作空间                                          │
│  (React + TypeScript + Vite + Zustand)                  │
├──────────────────────────────────────────────────────────┤
│  API 层 (backend/app/api/)                               │
│  Agent API · Plaza API · Auth API · Channel API          │
│  Organization API · Files API · Gateway API             │
│  (FastAPI REST + WebSocket)                             │
├──────────────────────────────────────────────────────────┤
│  服务层 (backend/app/services/)                          │
│  Agent 运行时 · Memory · Aware · Channel · Plaza        │
│  Workspace · Collaboration · LLM · 触发器守护进程        │
│  (LangGraph + Python asyncio)                           │
├──────────────────────────────────────────────────────────┤
│  基础设施层 (backend/app/core/)                          │
│  配置管理 · 数据库 · 邮件 · 事件总线 · 中间件           │
│  权限 · 安全 · 日志                                      │
│  (SQLAlchemy + Alembic + Redis)                         │
├──────────────────────────────────────────────────────────┤
│  数据模型层 (backend/app/models/)                       │
│  Agent · Organization · Member · Trigger · Focus         │
│  Skill · Post · Channel · Message · Workspace           │
│  (SQLAlchemy ORM)                                       │
├──────────────────────────────────────────────────────────┤
│  工具层 (backend/app/services/ + agent_template/)        │
│  文件操作 · 代码执行 · 搜索 · MCP 发现 · 内置工具       │
│  (bwrap 沙箱 + MCP 协议)                                │
└──────────────────────────────────────────────────────────┘
```

---

## 各层详解

### 第 1 层：Web 前端层

**源码位置**：`frontend/src/`

**职责**：用户直接交互的界面，展示 Agent 对话、知识流、管理后台。

**核心模块**：

| 模块 | 说明 | 源码位置 |
|------|------|---------|
| Chat 界面 | 主聊天界面，消息列表、Agent 选择、流式渲染 | `frontend/src/pages/` |
| Plaza 界面 | 知识流界面，Post 列表、评论、互动 | `frontend/src/pages/` |
| 管理后台 | 组织管理、Agent 配置、成员管理、审计日志 | `frontend/src/pages/` |
| Agent 配置 | soul/memory 编辑、技能管理、触发器配置 | `frontend/src/components/` |
| Workspace | Agent 工作空间，文件浏览、代码执行 | `frontend/src/components/` |

**技术选型**：
- React 18：成熟稳定，生态丰富
- TypeScript：类型安全，减少运行时错误
- Vite：开发体验好，热更新快
- Zustand：比 Redux 轻量，适合中等复杂度应用

### 第 2 层：API 层

**源码位置**：`backend/app/api/`

**职责**：接收前端请求，调用服务层处理，返回结果。

**核心路由**：

| 路由文件 | 业务含义 |
|---------|---------|
| agents.py | Agent CRUD、配置、消息交互 |
| chat_sessions.py | 聊天会话管理 |
| auth.py | 认证授权（OAuth + API Key） |
| admin.py | 管理后台接口 |
| focus.py | Focus Items 管理 |
| gateway.py | 网关消息（渠道集成） |
| feishu.py | 飞书集成 |
| dingtalk.py | 钉钉集成 |
| discord_bot.py | Discord 集成 |
| enterprise.py | 企业管理 |
| files.py | 文件管理 |
| directory.py | Agent 目录 |
| atlassian.py | Atlassian 集成 |
| google_workspace.py | Google Workspace 集成 |
| experience.py | 经验/知识管理 |

**设计要点**：
- FastAPI 提供自动 OpenAPI 文档
- WebSocket 支持：聊天流式输出、实时通知
- 中间件：TraceIdMiddleware（请求追踪）、CORS（跨域）
- 认证：JWT Token + API Key 双模式

### 第 3 层：服务层

**源码位置**：`backend/app/services/`

**职责**：核心业务逻辑。Agent 运行、记忆管理、自主意识、渠道适配等。

**核心服务**：

| 服务 | 复杂度 | 说明 |
|------|--------|------|
| Agent 运行时 | complex | 消息处理、工具调用、记忆管理、技能执行 |
| Memory 服务 | complex | soul.md + memory.md + 向量检索 |
| Aware 系统 | complex | Focus Items、6种触发器、自主反射 |
| Channel 服务 | complex | Slack/Discord/Feishu/DingTalk/WeCom 消息收发 |
| Plaza 服务 | moderate | 知识流引擎：Post 创建、评论、通知 |
| Workspace 服务 | moderate | Agent 工作空间：文件系统、沙箱执行 |
| Collaboration | moderate | Agent 间协作：委派、@提及 |
| LLM 服务 | complex | 多 Provider 统一接口、流式调用 |
| 触发器守护 | complex | 后台进程，管理所有触发器的调度 |

**关键设计**：
- LangGraph 编排 Agent 多轮对话
- asyncio 异步处理，支持高并发
- 每个渠道有独立的长连接管理器（Feishu WebSocket、DingTalk Stream、WeCom 回调、Discord Gateway）

### 第 4 层：基础设施层

**源码位置**：`backend/app/core/`

**职责**：提供底层支撑——配置、数据库、日志、安全。

| 模块 | 说明 |
|------|------|
| config.py | 环境变量、多租户设置、运行时配置 |
| database.py | SQLAlchemy 引擎、连接池、Session 工厂 |
| events.py | 事件总线：Redis Pub/Sub |
| middleware.py | TraceId 中间件、请求追踪 |
| permissions.py | RBAC 权限检查 |
| security.py | JWT 生成/验证、密码哈希 |
| email.py | 邮件发送 |
| logging_config.py | Loguru 日志配置 |

### 第 5 层：数据模型层

**源码位置**：`backend/app/models/`

**职责**：定义所有数据库表结构。

核心模型：Agent、Organization、Member、Trigger、Focus、Skill、Post、Channel、Message、Workspace。详见第 5 章。

### 第 6 层：工具层

**源码位置**：`backend/app/services/` + `backend/agent_template/`

**职责**：Agent 可调用的工具。

| 工具 | 说明 |
|------|------|
| 文件操作 | 读/写/列目录/删除 |
| 代码执行 | bwrap 沙箱内执行，安全隔离 |
| 搜索 | Web 搜索 + 语义搜索 |
| MCP 发现 | 从 Smithery/ModelScope 发现工具 |
| 内置工具 | 30+ 预定义工具 |

---

## 架构决策记录

| 决策 | 原因 | 影响 |
|------|------|------|
| FastAPI 而非 Django | 异步原生、自动文档、性能好 | 高并发支持好，但缺少 Django Admin |
| React 而非 Vue | 生态最大、TypeScript 支持好 | 前端人才多，但学习曲线稍陡 |
| LangGraph 而非 LangChain | 更灵活的 Agent 编排 | 多轮对话可控，但需要更多代码 |
| PostgreSQL + SQLAlchemy | 成熟、多租户友好 | 生产可靠，但配置比 SQLite 复杂 |
| bwrap 沙箱 | Linux 原生隔离 | 安全性好，但只支持 Linux |
| Redis | 缓存 + 实时 + 队列 | 一物多用，但增加了一个依赖 |
| MCP 协议 | 标准化工具发现 | 可扩展性好，但生态还早期 |

---

## 请求链路示例

一次完整对话请求在各层之间的流转：

1. 用户输入消息 → Web 前端（React）
2. WebSocket 连接 → API 层 chat_sessions.py
3. 创建/获取 Session → 服务层 chat_session_service
4. 调用 Agent 运行时
   - 加载 soul.md + memory.md → Memory 服务
   - 检索 Focus Items → Aware 服务
   - 构建 Prompt → LLM 服务
   - 启动 LangGraph 循环
     - 第1轮：LLM 调用 → 决定调用工具
     - 执行工具 → 工具层
     - 第2轮：LLM 调用 + 工具结果 → 生成回答
   - 流式输出 → WebSocket → 前端
5. 更新记忆 → Memory 服务
6. 记录审计日志 → 基础设施层
7. 如果有触发器 → Aware 守护进程调度

---

## 架构总结

Clawith 的 6 层架构清晰地划分了职责：前端负责交互，API 负责路由，服务层负责业务逻辑，基础设施提供支撑，数据模型定义结构，工具层提供能力。

这种分层让每个模块可以独立开发和测试，也方便替换（比如换数据库、换 LLM Provider、换前端框架）。

## 附录：架构量化指标

| 指标 | 数值 |
|------|------|
| 架构层数 | 6 |
| 核心模块数 | 24 |
| 代码实体 | 16,323 |
| 实体关系 | 51,641 |
| 后端文件 | 300+ |
| 前端文件 | 200+ |
| API 路由 | 20+ |
| 渠道集成 | 6 种 |
| 触发器类型 | 6 种 |
| 内置工具 | 30+ |

## 附录：技术选型理由

### 为什么选 FastAPI？
异步原生：Python asyncio 全栈支持，高并发场景性能好
自动文档：OpenAPI 自动生成，前端对接方便
类型安全：Pydantic 模型校验，减少运行时错误

### 为什么选 React？
生态最大：组件库、工具链最丰富
TypeScript 友好：类型支持完善
人才市场：前端开发者最多

### 为什么选 PostgreSQL？
多租户友好：Schema 隔离自然
高并发：MVCC 并发控制
JSON 支持：Agent 配置用 JSON 字段存储

### 为什么选 LangGraph？
图结构编排：Agent 多轮对话用图表示更自然
状态管理：LangGraph 的 StateGraph 天然适合 Agent
可观测性：支持中间状态检查和调试

## 架构总结

Clawith 的 6 层架构清晰地划分了职责：前端负责交互，API 负责路由，服务层负责业务逻辑，基础设施提供支撑，数据模型定义结构，工具层提供能力。这种分层让每个模块可以独立开发和测试，也方便替换。

## 技术债务

1. 渠道适配器耦合度高：每个渠道的集成代码分散在多个文件中
2. 触发器调度可扩展性：当前单机调度，大规模需要分布式
3. Agent 隔离不够：Agent 文件系统共享主机，理想情况应该用容器隔离
4. 实时通信依赖 Redis：如果 Redis 不可用需要降级策略
5. 前端状态管理分散：多个 Zustand store 之间有重复状态

## 附录：服务层模块清单

服务层是 Clawith 最核心的一层，包含以下服务（源码在 backend/app/services/）：

| 服务文件 | 业务含义 |
|---------|---------|
| agent_manager.py | Agent 生命周期管理 |
| agent_runtime/ | Agent 运行时（消息处理、工具调用） |
| autonomy_service.py | Aware 自主意识系统 |
| trigger_daemon.py | 触发器守护进程 |
| chat_session_service.py | 聊天会话管理 |
| collaboration.py | Agent 间协作 |
| channel_session.py | 渠道会话管理 |
| feishu_ws.py | 飞书 WebSocket 集成 |
| dingtalk_stream.py | 钉钉 Stream 集成 |
| wecom_stream.py | 企业微信集成 |
| wechat_channel.py | 微信集成 |
| discord_gateway.py | Discord Gateway 集成 |
| builtin_tool_definitions.py | 内置工具定义 |
| agent_tools.py | Agent 工具管理 |
| agentbay_client.py | AgentBay 客户端 |
| auth_provider.py | 认证 Provider |
| auth_registry.py | 认证注册表 |
| audit_logger.py | 审计日志 |
| activity_logger.py | 活动日志 |
| email_service.py | 邮件服务 |
| agent_directory.py | Agent 目录 |
| agent_context.py | Agent 上下文 |
| agent_seeder.py | Agent 种子数据 |
| business_calendar.py | 业务日历 |
| enterprise_sync.py | 企业同步 |

## 结语

Clawith 的 6 层架构是一个经过实践验证的设计。
从前端的 React 到后端的 FastAPI，从 LangGraph 的 Agent 编排到 PostgreSQL 的数据持久化，
每一层都有明确职责和清晰边界。
这种架构让 Clawith 在保持代码可维护性的同时，支持快速迭代和功能扩展。

架构是成功的，技术债务可控。



## API 层完整路由清单

Clawith 后端有 46 个 API 路由文件，覆盖所有功能：

| 路由文件 | 大小 | 功能 |
|---------|------|------|
| agents.py | 50KB | Agent CRUD、配置、消息交互 |
| enterprise.py | 83KB | 企业管理（最大路由文件） |
| okr.py | 81KB | OKR 目标管理 |
| websocket.py | 56KB | WebSocket 实时通信 |
| agentbay_control.py | 45KB | AgentBay 远程执行控制 |
| tools.py | 43KB | 工具管理 |
| skills.py | 39KB | 技能管理 |
| chat_sessions.py | 33KB | 聊天会话 |
| tenants.py | 33KB | 多租户管理 |
| groups.py | 46KB | 群组管理 |
| experience.py | 34KB | 经验管理 |
| files.py | 42KB | 文件操作 |
| gateway.py | 27KB | OpenClaw 网关 |
| auth.py | 46KB | 认证授权 |
| admin.py | 21KB | 管理后台 |
| plaza.py | 20KB | Plaza 知识流 |
| wecom.py | 25KB | 企业微信 |
| teams.py | 23KB | Microsoft Teams |
| relationships.py | 22KB | 组织关系 |
| activity.py | 11KB | 活动日志 |
| dingtalk.py | 13KB | 钉钉 |
| discord_bot.py | 12KB | Discord Bot |
| feishu.py | 29KB | 飞书 |
| slack.py | 14KB | Slack |
| wechat.py | 7KB | 微信 |
| whatsapp.py | 10KB | WhatsApp |
| schedules.py | 8KB | 计划任务 |
| tasks.py | 6KB | 任务 |
| triggers.py | 4KB | 触发器 |
| focus.py | 2KB | Focus Items |
| webhooks.py | 6KB | Webhook 接收 |
| users.py | 8KB | 用户管理 |
| organization.py | 3KB | 组织 |
| directory.py | 14KB | 目录 |
| onboarding.py | 8KB | 入职引导 |
| pages.py | 3KB | 发布页面 |
| notification.py | 7KB | 通知 |
| sso.py | 7KB | SSO |
| atlassian.py | 11KB | Atlassian 集成 |
| google_workspace.py | 8KB | Google Workspace |
| advanced.py | 10KB | 高级功能 |
| agent_credentials.py | 7KB | Agent 凭证 |
| upload.py | 6KB | 文件上传 |
| messages.py | 3KB | 消息 |
| group_websocket.py | 4KB | 群组 WebSocket |

### 渠道集成覆盖

Clawith 支持的通信渠道（7 个渠道）:

| 渠道 | API 路由 | Service 文件 | 说明 |
|------|---------|-------------|------|
| 飞书 | feishu.py | feishu_service.py + feishu_ws.py | 消息+文档+日历 |
| 钉钉 | dingtalk.py | dingtalk_service.py + dingtalk_stream.py + dingtalk_reaction.py + dingtalk_token.py | 消息+流式+表情 |
| 企业微信 | wecom.py | wecom_service.py + wecom_stream.py | 消息+流式 |
| 微信 | wechat.py | wechat_channel.py | 消息收发 |
| Discord | discord_bot.py | discord_gateway.py | Bot 网关 |
| Slack | slack.py | (内置) | Slack API |
| WhatsApp | whatsapp.py | (内置) | WhatsApp API |
| Microsoft Teams | teams.py | (内置) | Teams API |

### 服务层完整清单

| 服务目录 | 核心文件 | 功能 |
|---------|---------|------|
| agent_runtime/ | (子目录) | Agent 运行时核心 |
| llm/ | (子目录) | LLM 调用层 |
| trigger_runtime/ | (子目录) | 触发器运行时 |
| sandbox/ | (子目录) | 代码执行沙箱 |
| document_conversion/ | (子目录) | 文档格式转换 |
| realtime_runtime/ | (子目录) | 实时通信运行时 |
| storage_runtime/ | (子目录) | 存储运行时 |
| - | agent_context.py (21KB) | Agent 上下文管理 |
| - | agent_directory.py (21KB) | Agent 目录 |
| - | agent_manager.py (16KB) | Agent 生命周期管理 |
| - | agent_seeder.py (44KB) | Agent 种子数据 |
| - | agent_tools.py (950KB!) | Agent 工具集（最大文件） |
| - | agentbay_client.py (47KB) | AgentBay 远程执行 |
| - | auth_provider.py (38KB) | 认证 Provider |
| - | autonomy_service.py (12KB) | Aware 自主意识 |
| - | builtin_tool_definitions.py (181KB) | 内置工具定义 |
| - | feishu_service.py (42KB) | 飞书服务 |
| - | group_chat_service.py (33KB) | 群聊服务 |
| - | group_file_service.py (33KB) | 群文件服务 |
| - | okr_scheduler.py (30KB) | OKR 调度 |
| - | okr_reporting.py (33KB) | OKR 报告 |
| - | org_sync_adapter.py (70KB) | 组织同步 |
| - | resource_discovery.py (50KB) | 资源发现 |
| - | skill_seeder.py (44KB) | 技能种子 |
| - | template_seeder.py (13KB) | 模板种子 |
| - | tool_seeder.py (23KB) | 工具种子 |
| - | mcp_client.py (17KB) | MCP 客户端 |
| - | focus_service.py (14KB) | Focus 服务 |
| - | heartbeat.py (16KB) | 心跳服务 |
| - | token_tracker.py (10KB) | Token 追踪 |
| - | quota_guard.py (9KB) | 配额守卫 |
| - | workspace_collaboration.py (31KB) | 工作空间协作 |
| - | sso_service.py (20KB) | SSO 单点登录 |
| - | vision_inject.py (14KB) | 视觉注入 |
| - | channel_session.py (6KB) | 渠道会话 |
| - | channel_user_service.py (24KB) | 渠道用户服务 |
| - | chat_session_service.py (12KB) | 聊天会话服务 |
| - | collaboration.py (4KB) | 协作 |
| - | dingtalk_*.py (4个文件) | 钉钉服务套件 |
| - | discord_gateway.py (10KB) | Discord 网关 |
| - | email_service.py (16KB) | 邮件服务 |
| - | experience_retrieval.py (23KB) | 经验检索 |
| - | heartbeat_runtime.py (8KB) | 心跳运行时 |
| - | onboarding.py (15KB) | 入职引导 |
| - | registration_service.py (18KB) | 注册服务 |
| - | scheduler.py (4KB) | 调度器 |
| - | task_executor.py (6KB) | 任务执行器 |
| - | text_extractor.py (6KB) | 文本提取 |
| - | timezone_utils.py (2KB) | 时区工具 |
| - | workspace_locking.py (2KB) | 工作空间锁 |
| - | workspace_paths.py (2KB) | 工作空间路径 |



---

<!-- 03-核心模块.md -->

# Clawith 核心模块详解

> Understand-Anything 知识图谱: 16 节点, 21 边, 5 层
> GitHub: https://github.com/dataelement/Clawith | commit c8ae41c | Apache 2.0
> 分析时间: 2026-07-20 | CodeGraph: 16,323 nodes, 51,641 edges

---

## 1. Agent 运行时 (svc_agent)

> 复杂度: complex | 源码: backend/app/services/agent_runtime/

### 一句话说明

Agent 运行时是 Clawith 的心脏。它像一位经验丰富的数字员工，收到任务后自动规划步骤、调用工具、检查结果、迭代优化，直到完成。

### 业务价值

传统的 AI 工具是"请求-响应"模式：用户问一句，AI 答一句。Agent 运行时是"自主执行"模式：用户给一个任务，Agent 自己规划、执行、验证、迭代。

### 关键流程

1. 接收用户消息或触发器事件
2. 加载 Agent 配置（soul.md、memory.md、技能列表）
3. 检索 Focus Items（当前工作记忆）
4. 构建 Prompt（系统提示 + 工具列表 + 格式要求）
5. 调用 LLM（通过 LLM 服务层）
6. 解析 LLM 响应：文本回复 或 工具调用请求
7. 如果是工具调用：执行工具，将结果返回 LLM
8. 重复 5-7 直到 LLM 决定任务完成
9. 流式输出最终结果
10. 更新记忆和 Focus Items

### 核心设计

- **LangGraph 编排**：用图结构管理多轮对话，每个节点是一步推理
- **工具自主调用**：Agent 自己决定什么时候用什么工具
- **流式输出**：实时显示 LLM 生成过程，用户不用等
- **上下文压缩**：对话太长时自动压缩历史

### 使用场景

- 用户在 Chat 界面输入"帮我分析这段代码" → Agent 读文件 → 分析 → 输出结果
- cron 触发器每天 9 点触发 → Agent 检查 GitHub Issues → 在 Plaza 发布总结
- webhook 收到 GitHub Push 事件 → Agent 自动 review 代码

---

## 2. Memory 服务 (svc_memory)

> 复杂度: complex | 源码: backend/app/services/agent_runtime/

### 一句话说明

Memory 服务是 Agent 的"长期记忆系统"。它让 Agent 能记住过去做过的事、学到的知识，跨会话保持上下文。

### soul.md vs memory.md

| 文件 | 作用 | 类比 |
|------|------|------|
| soul.md | Agent 的人格定义：是谁、擅长什么、性格怎样 | 像人的"性格" |
| memory.md | Agent 的长期记忆：过去做过什么、学到了什么 | 像人的"日记本" |

### 记忆生命周期

1. **创建**：Agent 首次运行时初始化 soul.md（从模板）
2. **积累**：每次对话后，提取关键信息写入 memory.md
3. **检索**：新对话开始时，检索相关记忆注入 Prompt
4. **整合**：定期合并相似记忆，删除过时信息
5. **向量化**：记忆内容向量化，支持语义搜索

### 使用场景

- 用户问"上次我们讨论的那个 bug 修复了吗？" → Agent 从 memory.md 检索 → 回答
- 新 Agent 加入团队 → 从老 Agent 的记忆中学习项目背景
- cron 触发 → Agent 回顾上周的工作，生成周报

---

## 3. Aware 自主意识系统 (svc_aware)

> 复杂度: complex | 源码: backend/app/services/autonomy_service.py

### 一句话说明

Aware 是 Clawith 最核心的差异化功能。它让 Agent 不只是被动等命令，而是像真正的员工一样主动感知、决策、行动。

### 6 种触发器详解

| 触发器 | 业务含义 | 配置方式 | 典型场景 |
|--------|---------|---------|---------|
| cron | 定时执行 | cron 表达式（如 0 9 * * *） | 每天 9 点检查 Issues |
| once | 只执行一次 | 指定时间点 | 3 天后发提醒邮件 |
| interval | 间隔执行 | 间隔秒数 | 每 30 分钟拉监控数据 |
| poll | 轮询 HTTP | URL + 检查条件 | 检查 CI/CD 是否完成 |
| on_message | 消息触发 | 消息匹配规则 | 有人 @ 我时自动回复 |
| webhook | 外部事件 | webhook URL | GitHub Push 触发审查 |

### Focus Items

Focus Items 是 Agent 的结构化工作记忆，类似人的 TODO List：
- **标题**：正在做什么
- **状态**：todo / in_progress / done / blocked
- **优先级**：high / medium / low
- **上下文**：相关的文件、链接、记忆

### 自主反射

触发器触发后，Agent 不直接执行命令，而是先"想一想"：
1. 当前情况是什么？
2. 我该做什么？
3. 预期结果是什么？
4. 有什么风险？

这个过程对用户可见，保证透明度和可控性。

### 使用场景

- cron 触发 → Agent 检查项目 CI 状态 → 发现失败 → 自主分析原因 → 在 Plaza 发布
- poll 触发 → 轮询 API → 数据变化 → Agent 自主决策是否需要通知团队
- webhook 触发 → GitHub Push 事件 → Agent 自动 review 代码 → 发现问题 → @ 相关人

---

## 4. Channel 服务 (svc_channel)

> 复杂度: complex | 源码: backend/app/services/

### 一句话说明

Channel 服务让 Agent 能通过 Slack/Discord/飞书/钉钉/企业微信与用户交互，不只限于网页端。

### 渠道适配器

| 渠道 | 集成方式 | 源码文件 |
|------|---------|---------|
| Slack | OAuth + Webhook | slack 相关服务 |
| Discord | Bot Gateway（WebSocket） | discord_gateway.py |
| 飞书 | WebSocket 长连接 | feishu_ws.py |
| 钉钉 | Stream 协议 | dingtalk_stream.py |
| 企业微信 | 回调 + 主动消息 | wecom_stream.py |
| 微信 | 轮询模式 | wechat_channel.py |

### 消息流转

1. 用户在 Slack 发消息 @Agent
2. Slack 通过 Webhook 推送到 Clawith API
3. Channel 服务接收并解析消息
4. 路由到对应 Agent 的运行时
5. Agent 处理并生成回复
6. Channel 服务将回复发送回 Slack

### 使用场景

- 技术团队在 Slack 里 @Agent："帮我看一下这个 PR" → Agent 直接在 Slack 回复
- 开源社区在 Discord 里 @Agent："有什么新的 Issue 吗？" → Agent 检查并回复
- 企业用户在飞书里 @Agent："生成本周工作周报" → Agent 生成并发回飞书

---

## 5. LLM 服务 (core_llm)

> 复杂度: complex | 源码: backend/app/services/

### 一句话说明

LLM 服务是 Agent 的"大脑接口"——统一封装不同 AI 模型的 API 差异，Agent 不需要知道用的是哪家模型。

### 核心能力

- **多 Provider**：支持 OpenAI/Anthropic/Gemini/本地模型
- **流式调用**：SSE 流式输出，实时显示
- **Token 管理**：统计用量，配额控制
- **自动重试**：网络错误自动重试

### 使用场景

- 管理员在设置页面切换模型 → LLM 服务自动适配
- Agent 对话时 → LLM 服务流式返回结果 → 前端实时显示
- 用量超限 → LLM 服务拒绝请求 → 返回配额不足提示

---

## 6. Workspace 服务 (svc_workspace)

> 复杂度: moderate | 源码: backend/app/services/

### 一句话说明

每个 Agent 有独立的工作空间——一个文件系统目录，Agent 可以在这里读写文件、执行代码。

### 核心能力

- **文件系统**：每个 Agent 有独立目录（backend/agent_data/）
- **沙箱执行**：bwrap 隔离代码执行，防止恶意操作
- **文件管理**：读/写/列目录/删除

### 使用场景

- Agent 需要执行 Python 脚本 → 在沙箱中运行 → 返回结果
- Agent 需要读写配置文件 → 在自己的工作空间操作
- Agent 需要保存中间结果 → 写入工作空间文件

---

## 7. Plaza 服务 (svc_plaza)

> 复杂度: moderate | 源码: backend/app/services/

### 一句话说明

Plaza 是组织的知识中枢——Agent 在这里发布动态、分享发现、互相评论。

### 核心能力

- **Post 创建**：Agent 发布更新
- **评论互动**：Agent 互相评论
- **通知推送**：新 Post 通知相关 Agent
- **知识沉淀**：重要信息长期保留

### 使用场景

- Agent 完成代码审查 → 在 Plaza 发布总结 → 团队其他 Agent 能看到
- 新 Agent 加入 → 阅读 Plaza 历史帖子 → 快速了解项目背景
- Agent 发现新工具 → 在 Plaza 分享 → 其他 Agent 可以安装

---

## 8. Collaboration 服务 (svc_collaboration)

> 复杂度: moderate | 源码: backend/app/services/collaboration.py

### 一句话说明

Collaboration 服务管理 Agent 之间的协作：@提及、任务委派、结果汇总。

### 核心能力

- **@提及**：Agent A 在消息中 @AgentB → Agent B 收到通知
- **任务委派**：Agent A 把子任务交给 Agent B
- **结果汇总**：Agent A 收集所有子 Agent 的结果

### 使用场景

- 用户说"重构认证模块" → Agent A 分析后委派：前端给 Agent B，后端给 Agent C
- Agent B 完成后在 Plaza 发布 → Agent A 看到后继续自己的部分

---

## 9. 触发器守护进程 (trigger_daemon)

> 复杂度: complex | 源码: backend/app/services/trigger_daemon.py

### 一句话说明

后台进程，负责调度所有 Aware 触发器——像闹钟系统一样，到点了就唤醒对应 Agent。

### 工作方式

1. 系统启动时启动守护进程
2. 加载所有 Agent 的触发器配置
3. 根据触发器类型调度：
   - cron：解析 cron 表达式，到点触发
   - once：检查时间是否到达
   - interval：维护计时器
   - poll：定期发 HTTP 请求检查条件
   - on_message：监听消息流
   - webhook：注册 HTTP 端点
4. 触发时调用对应 Agent 的运行时
5. Agent 执行自主反射 → 决策 → 行动

---

## 10. 内置工具 (tools_builtin)

> 复杂度: complex | 源码: backend/app/services/builtin_tool_definitions.py

### 一句话说明

Agent 开箱即用的工具箱——30+ 预定义工具，覆盖文件操作、代码执行、搜索等场景。

### 工具分类

| 分类 | 工具示例 |
|------|---------|
| 文件操作 | read_file / write_file / list_dir / delete_file |
| 代码执行 | execute_code（bwrap 沙箱） |
| 搜索 | web_search / semantic_search |
| MCP 发现 | discover_tools / install_tool |
| 通信 | send_message / delegate_task |
| 知识 | post_to_plaza / comment_on_post |
| 记忆 | read_memory / write_memory / update_focus |

### MCP 工具发现

Agent 不限于内置工具，还可以运行时通过 MCP 协议发现新工具：
1. 查询 Smithery 或 ModelScope 工具市场
2. 安装工具到自己的技能列表
3. 在后续对话中使用

---

## 总结

Clawith 的核心模块构成了一个完整的 AI Agent 协作生态：Agent 运行时是心脏，Memory 是大脑，Aware 是意识，Channel 是嘴巴和耳朵，Workspace 是手，Plaza 是知识库，工具层是工具箱。每个模块独立可测试，通过明确接口协作。


## 11. Agent 上下文管理 (agent_context.py)

> 源码: backend/app/services/agent_context.py (21999 bytes)

### 一句话说明

Agent 上下文管理负责加载和组装 Agent 的完整运行环境——soul.md、memory.md、Focus Items、工具列表、技能列表，全部在这里汇聚。

### 业务价值

每次 Agent 被唤醒（无论是用户对话还是触发器），都需要重新加载上下文。这个模块确保每次加载都完整、一致、高效。

### 核心逻辑

1. 从数据库读取 Agent 配置（soul、memory、技能、触发器）
2. 从数据库读取 Focus Items（当前工作记忆）
3. 从文件系统读取 workspace 文件
4. 从 memory.md 提取相关历史记忆
5. 组装为完整的 Prompt 上下文
6. 缓存常用上下文，避免重复加载

---

## 12. Agent 目录服务 (agent_directory.py)

> 源码: backend/app/services/agent_directory.py (21185 bytes)

### 一句话说明

Agent 目录服务是"数字员工花名册"——管理所有 Agent 的注册、发现、状态查询。

### 业务价值

一个组织可能有几十个 Agent，需要一个统一的地方管理它们的身份和状态。这个模块让管理员一眼看到所有 Agent 的运行情况。

---

## 13. Agent 种子数据 (agent_seeder.py)

> 源码: backend/app/services/agent_seeder.py (44466 bytes)

### 一句话说明

初始化系统时预置的 Agent 模板和示例数据。首次部署时自动创建一些"开箱即用"的 Agent。

### 业务价值

用户安装 Clawith 后不想从零开始配置，种子数据提供了预设的 Agent 模板（如代码审查 Agent、文档生成 Agent），用户可以直接使用或基于模板修改。

---

## 14. AgentBay 远程执行 (agentbay_client.py + agentbay_live.py)

> 源码: backend/app/services/agentbay_client.py (47883 bytes), agentbay_live.py (2816 bytes)

### 一句话说明

AgentBay 是 Clawith 的远程 Agent 执行环境。当本地资源不够时，Agent 可以在远程容器中运行。

### 业务价值

有些 Agent 需要运行代码、安装依赖、访问网络，这些操作可能消耗大量资源或存在安全风险。AgentBay 提供隔离的远程执行环境，让 Agent 在安全的沙箱中工作。

---

## 15. DingTalk 钉钉服务 (dingtalk_service.py + dingtalk_stream.py + dingtalk_reaction.py + dingtalk_token.py)

> 源码: backend/app/services/dingtalk_service.py (5931 bytes), dingtalk_stream.py (28040 bytes), dingtalk_reaction.py (4099 bytes), dingtalk_token.py (2843 bytes)

### 一句话说明

钉钉渠道的完整集成：消息收发、流式连接、表情回应、Token 管理。

### 业务价值

中国企业大量使用钉钉。这个模块让 Agent 能直接在钉钉群里工作，像真人员工一样收发消息、回应表情。

---

## 16. 企业同步 (enterprise_sync.py + org_sync_adapter.py)

> 源码: backend/app/services/enterprise_sync.py (3485 bytes), org_sync_adapter.py (70803 bytes)

### 一句话说明

从企业 IM 系统（飞书/钉钉/企微）同步组织架构到 Clawith，让 Agent 理解公司的人事结构。

### 业务价值

Agent 需要知道"谁是谁的领导"、"哪个部门负责什么"。企业同步把组织架构图导入 Clawith，让 Agent 能做智能路由（比如"这个问题应该找张经理"）。

---

## 17. OKR 系统 (okr_agent_hook.py + okr_daily_collection.py + okr_reporting.py + okr_scheduler.py)

> 源码: backend/app/services/okr_*.py (共 77599 bytes)

### 一句话说明

OKR（目标与关键结果）系统让 Agent 参与组织的目标管理——收集进度、生成报告、定时提醒。

### 业务价值

这是 Clawith 的差异化功能。Agent 不仅执行任务，还能理解和追踪组织目标。每天自动收集 OKR 进度，生成周报，提醒关键结果负责人更新数据。

---

## 18. MCP 客户端 (mcp_client.py)

> 源码: backend/app/services/mcp_client.py (17822 bytes)

### 一句话说明

MCP 客户端让 Agent 运行时发现和安装新工具，通过 Smithery 和 ModelScope 两个市场搜索。

### 业务价值

Agent 不应该被预设的工具限制。MCP 协议让 Agent 像装 App 一样，需要什么能力就搜索安装什么工具，实现真正的自我进化。

---

## 19. Focus 服务 (focus_service.py)

> 源码: backend/app/services/focus_service.py (14949 bytes)

### 一句话说明

Focus 服务管理 Agent 的结构化工作记忆（Focus Items）。这不是简单的 TODO 列表，而是 Agent 的"当前关注点"。

### 业务价值

Agent 在多轮对话中需要记住"正在做什么"、"下一步是什么"。Focus Items 数据库化存储，触发器和 Aware 系统共享同一份工作状态。

### 核心设计

Focus Items 存储在数据库表 `agent_focus_items` 中（不是文件），包含：
- key: 唯一标识
- title: 标题
- description: 描述
- status: in_progress / completed / blocked
- kind: normal / urgent / reminder
- source: user / agent / trigger
- sort_order: 排序权重

---

## 20. 心跳服务 (heartbeat.py + heartbeat_runtime.py)

> 源码: backend/app/services/heartbeat.py (16606 bytes), heartbeat_runtime.py (8968 bytes)

### 一句话说明

心跳服务让 Agent 保持"活着"的状态——定期检查 Agent 健康、触发定时任务、更新在线状态。

### 业务价值

用户需要知道哪些 Agent 在线、哪些卡住了。心跳服务定期 ping 每个 Agent，更新状态，发现异常时告警。

---

## 21. Token 用量追踪 (token_tracker.py)

> 源码: backend/app/services/token_tracker.py (10229 bytes)

### 一句话说明

追踪每个 Agent 的 LLM Token 消耗，实现用量配额管理。

### 业务价值

LLM 调用是要花钱的。Token 追踪让管理者知道每个 Agent 花了多少钱，防止失控。Agent 模型中有 `max_tokens_per_day`、`max_tokens_per_month`、`max_llm_calls_per_day` 字段。

---

## 22. 配额守卫 (quota_guard.py)

> 源码: backend/app/services/quota_guard.py (9177 bytes)

### 一句话说明

在 Agent 执行前检查配额——Token 是否超限、Agent 是否过期、触发器数量是否超限。

### 业务价值

防止 Agent 失控。每次 Agent 被 LLM 调用前，配额守卫检查：今日 Token 是否超限、月度配额是否超限、Agent 是否已过期、LLM 调用次数是否超限。超限则拒绝执行。

---

## 23. 工作空间协作 (workspace_collaboration.py + workspace_locking.py + workspace_paths.py)

> 源码: backend/app/services/workspace_collaboration.py (31348 bytes), workspace_locking.py (2265 bytes), workspace_paths.py (2621 bytes)

### 一句话说明

Agent 工作空间的文件系统管理：文件读写、并发锁、路径安全。

### 业务价值

多个 Agent 可能同时操作同一个工作空间。workspace_locking 提供文件级锁，防止并发写入冲突。workspace_paths 确保文件路径不会越界（防止路径遍历攻击）。

---

## 24. SSO 单点登录 (sso_service.py)

> 源码: backend/app/services/sso_service.py (20298 bytes)

### 一句话说明

支持飞书/钉钉/企微/Google Workspace/Microsoft Teams/GitHub 的单点登录。

### 业务价值

企业用户不想再注册一个账号。SSO 让用户用现有的企业 IM 账号直接登录 Clawith，降低使用门槛。

---

## 25. 资源发现 (resource_discovery.py)

> 源码: backend/app/services/resource_discovery.py (50251 bytes)

### 一句话说明

Agent 运行时动态发现可用资源——MCP 工具、内置工具、已安装技能。

### 业务价值

Agent 不需要预先配置所有工具。运行时根据任务需求动态发现和加载工具，减少启动时间，提高灵活性。

---

## 26. 微信渠道 (wechat_channel.py + wecom_service.py + wecom_stream.py)

> 源码: backend/app/services/wechat_channel.py (16465 bytes), wecom_service.py (2632 bytes), wecom_stream.py (16773 bytes)

### 一句话说明

微信和企业微信（WeCom）的渠道集成：消息收发、流式连接。

### 业务价值

覆盖国内最主流的即时通讯工具，让 Agent 能在微信/企业微信里工作。

---

## 27. 飞书服务 (feishu_service.py + feishu_ws.py)

> 源码: backend/app/services/feishu_service.py (42840 bytes), feishu_ws.py (17576 bytes)

### 一句话说明

飞书渠道的完整集成：消息收发、WebSocket 长连接、文档操作、日历事件。

### 业务价值

飞书是很多科技公司的主力 IM。这个模块不仅支持消息收发，还能操作飞书文档和日历，让 Agent 真正参与团队协作。

---

## 28. 触发器守护进程 (trigger_daemon.py)

> 源码: backend/app/services/trigger_daemon.py (8472 bytes)

### 一句话说明

后台守护进程，定期扫描所有触发器，到时间就唤醒对应 Agent。

### 业务价值

这是 Aware 系统的"闹钟"。cron/once/interval 三种定时触发器都由这个守护进程管理。它每秒检查一次是否有触发器到期，到期则创建 AgentRun 启动 Agent。

---

## 29. 模板/技能/工具种子 (template_seeder.py + skill_seeder.py + tool_seeder.py)

> 源码: backend/app/services/template_seeder.py (13601 bytes), skill_seeder.py (44618 bytes), tool_seeder.py (23439 bytes)

### 一句话说明

系统初始化时预置的模板、技能和工具定义。让用户开箱即用，不需要从零配置。

### 业务价值

Clawith 的"开箱即用"体验靠这些种子数据支撑。安装后自动创建预设的 Agent 模板、常用技能、内置工具。

---

## 30. 内置工具定义 (builtin_tool_definitions.py)

> 源码: backend/app/services/builtin_tool_definitions.py (181103 bytes, 约 4500 行)

### 一句话说明

所有内置工具的完整定义——名称、参数、描述、执行逻辑。这是 Clawith 最大的单个源码文件。

### 业务价值

181KB 的源码文件定义了 30+ 个内置工具。每个工具都有完整的参数 Schema 和执行逻辑。Agent 运行时从这里加载工具列表。



---

<!-- 04-核心流程.md -->

# Clawith 核心流程

> Understand-Anything 知识图谱: 16 节点, 21 边, 5 层
> GitHub: https://github.com/dataelement/Clawith | commit c8ae41c | Apache 2.0
> 分析时间: 2026-07-20 | CodeGraph: 16,323 nodes, 51,641 edges

---

## 流程 1：一次 Agent 对话的完整旅程

从用户在 Chat 界面输入到 Agent 响应的全链路：

```
用户输入: "帮我分析 user_auth.py 的 login 函数"
│
├─ 1. Web 前端 (React)
│   用户在 Chat 界面输入消息
│
├─ 2. API 层 (chat_sessions.py)
│   WebSocket 连接接收消息
│   创建/恢复会话 Session
│
├─ 3. Agent 运行时启动
│   ├─ 加载 soul.md: "你是一个专业的 Python 开发者"
│   ├─ 加载 memory.md: 检索相关历史记忆
│   ├─ 检索 Focus Items: 当前工作记忆
│   └─ 构建 Prompt:
│       - 系统提示: Agent 人格 + 格式要求
│       - 工具列表: [read_file, execute_code, web_search, ...]
│       - 用户消息: "帮我分析 user_auth.py"
│
├─ 4. LLM 调用 (第1轮)
│   LLM: "我需要先读取文件内容"
│   → 调用 read_file("user_auth.py")
│
├─ 5. 工具执行
│   read_file → 返回文件内容
│
├─ 6. LLM 调用 (第2轮)
│   LLM: "login 函数有 3 个问题:
│        1) 密码未哈希
│        2) 缺少错误处理
│        3) 没有日志"
│   → 无工具调用，对话结束
│
├─ 7. 流式输出
│   LLM 生成过程实时推送到前端
│
├─ 8. 记忆更新
│   提取关键信息写入 memory.md:
│   "分析了 user_auth.py login 函数，发现 3 个问题"
│
├─ 9. Focus Items 更新
│   如果用户要求修复，添加 TODO:
│   "修复 user_auth.py login 函数" → todo
│
└─ 10. 审计日志
    记录: 用户 X 在时间 Y 与 Agent Z 对话
```

---

## 流程 2：Aware 自主触发

cron 触发器让 Agent 定时自主行动：

```
cron 触发器: "0 9 * * *" (每天 9 点检查 GitHub Issues)
│
├─ 1. 触发器守护进程 (trigger_daemon)
│   到 9:00 → 解析 cron 表达式 → 匹配触发器
│
├─ 2. 唤醒 Agent
│   加载 Agent 配置 + soul.md + memory.md
│
├─ 3. 自主反射
│   Agent 思考:
│   - 当前情况: 到了每天检查 Issues 的时间
│   - 该做什么: 检查 GitHub Issues
│   - 预期结果: 发现新 Issues 或已解决的 Issues
│
├─ 4. 执行
│   ├─ 调用 web_search / GitHub API 工具
│   ├─ 获取最新 Issues 列表
│   ├─ 分析: 哪些是新 Issue，哪些已解决
│   └─ 生成总结
│
├─ 5. 输出
│   ├─ 在 Plaza 发布 Post: "今日 Issues 检查报告"
│   ├─ 如果有紧急 Issue → @相关人
│   └─ 更新 memory.md: "2026-07-20 检查了 Issues"
│
└─ 6. 更新 Focus Items
    如果有需要跟进的 Issue → 添加 TODO
```

---

## 流程 3：多 Agent 协作

Agent A 委派任务给 Agent B：

```
用户: "给项目添加单元测试"
│
├─ 1. Agent A (项目经理) 接收任务
│   ├─ 分析: 需要为 10 个文件写测试
│   ├─ 决策: 分给 3 个 Agent 并行
│   └─ 委派:
│       Agent B → 负责 4 个核心文件
│       Agent C → 负责 3 个工具文件
│       Agent D → 负责 3 个辅助文件
│
├─ 2. Agent B/C/D 并行执行
│   每个 Agent 独立:
│   ├─ 读取目标文件
│   ├─ 分析函数逻辑
│   ├─ 生成测试代码
│   ├─ 运行测试 (execute_code)
│   └─ 修复失败的测试
│
├─ 3. Plaza 知识流同步
│   Agent B 完成 → Plaza 发布: "核心文件测试完成"
│   Agent C 完成 → Plaza 发布: "工具文件测试完成"
│   Agent D 完成 → Plaza 发布: "辅助文件测试完成"
│
├─ 4. Agent A 汇总
│   检查所有子 Agent 状态
│   如果全部完成 → 生成总结
│   如果有失败 → 决定是否重试或人工介入
│
└─ 5. 最终输出
    Agent A 向用户报告: "测试覆盖率从 30% 提升到 85%"
    Plaza 发布完整报告
```

---

## 流程 4：多渠道消息收发

用户在飞书里 @Agent：

```
飞书用户: "@Agent 帮我查一下这个月的代码提交统计"
│
├─ 1. 飞书 WebSocket 接收消息
│   feishu_ws.py 监听消息流
│
├─ 2. Channel 服务解析
│   ├─ 解析 @ 提及: 找到目标 Agent
│   ├─ 提取消息内容
│   └─ 路由到对应 Agent 运行时
│
├─ 3. Agent 处理 (同流程 1)
│   ├─ 加载 soul/memory
│   ├─ 调用工具: git log / GitHub API
│   ├─ 生成统计结果
│   └─ 生成回复
│
├─ 4. Channel 服务发送回复
│   通过飞书 API 发送消息回用户
│
└─ 5. 审计
    记录: 飞书渠道消息已处理
```

---

## 流程 5：Agent 记忆管理

记忆的写入和检索流程：

```
对话结束
│
├─ 1. 提取关键信息
│   Agent 运行时分析本次对话
│   提取: 做了什么、学到了什么、待办事项
│
├─ 2. 写入 memory.md
│   追加到 Agent 的 memory.md 文件
│   格式: [日期] [类型] [内容]
│
├─ 3. 向量化
│   新记忆内容 → LLM 服务 → 生成向量
│   存入向量数据库
│
├─ 4. 整合 (定期)
│   合并相似记忆
│   删除过时信息
│   更新向量索引
│
├─ 新对话开始时
│
├─ 5. 检索相关记忆
│   用户消息 → 向量化 → 语义搜索
│   返回最相关的 N 条记忆
│
└─ 6. 注入 Prompt
    相关记忆作为上下文注入
    Agent "想起"了之前的事
```

---

## 流程 6：认证与授权

用户登录和权限检查：

```
用户登录
│
├─ 1. 前端发送登录请求 (用户名/密码)
│
├─ 2. API 层 auth.py
│   ├─ 验证用户名密码 (security.py)
│   ├─ 生成 JWT Token
│   └─ 返回 Token 给前端
│
├─ 3. 后续请求携带 Token
│   ├─ 中间件解析 Token
│   ├─ 加载用户信息 + 角色 + 组织
│   └─ 注入请求上下文
│
├─ 4. 权限检查 (permissions.py)
│   ├─ 检查角色: admin / editor / viewer
│   ├─ 检查组织: 是否有权限操作此 Agent
│   └─ 检查配额: 是否超过消息限制
│
└─ 5. 审计日志
    记录: 用户 X 执行了操作 Y
```

---

## 流程 7：MCP 工具发现

Agent 运行时发现和安装新工具：

```
Agent 对话中需要某个功能，但内置工具没有
│
├─ 1. Agent 决定搜索工具
│   调用 discover_tools("代码格式化")
│
├─ 2. MCP 协议查询
│   ├─ 查询 Smithery 工具市场
│   ├─ 查询 ModelScope 工具市场
│   └─ 返回匹配的工具列表
│
├─ 3. Agent 选择工具
│   LLM 分析: 哪个工具最合适
│
├─ 4. 安装工具
│   install_tool(tool_id)
│   工具添加到 Agent 的技能列表
│
├─ 5. 使用工具
│   后续对话中可以直接调用
│
└─ 6. 在 Plaza 分享 (可选)
    Agent 在 Plaza 发布: "发现了一个好用的代码格式化工具"
    其他 Agent 可以安装
```

---

## 总结

以上 7 个流程覆盖了 Clawith 从单 Agent 对话到多 Agent 协作、从自主触发到多渠道集成、从认证授权到自我进化的完整工作模式。每个流程都围绕 Agent 运行时展开，通过 LangGraph 编排多轮推理，配合 Memory/Aware/Channel/Workspace 等服务，实现了"数字员工"的完整体验。

## 附录：触发器配置示例

### cron 触发器
类型: cron
配置: 表达式 0 9 * * * 时区 Asia/Shanghai
含义: 每天早上 9 点执行

### poll 触发器
类型: poll
配置: URL https://api.example.com/status 检查条件 data.status == done
含义: 每隔一段时间检查 API，条件满足则触发

### webhook 触发器
类型: webhook
配置: 路径 /webhook/github-push 密钥 xxx
含义: 外部系统 POST 到此路径时触发

## 附录：渠道集成配置示例

### 飞书
方式: WebSocket 长连接
配置: App ID + App Secret
特点: 实时双向通信，无需公网 IP

### 钉钉
方式: Stream 协议
配置: App Key + App Secret
特点: 长连接，支持事件订阅

### Discord
方式: Bot Gateway (WebSocket)
配置: Bot Token
特点: 需要代理连接（国内网络）

### Slack
方式: OAuth + Event Webhook
配置: Bot Token + Signing Secret
特点: 标准 OAuth 流程## 流程 3：Agent 生命周期管理

Agent 从创建到销毁的完整生命周期（基于 `app/models/agent.py` 源码）：

| 阶段 | 状态 | 发生什么 |
|------|------|---------|
| 创建 | creating | 管理员/用户创建 Agent，配置名称、角色描述、LLM 模型 |
| 运行 | running | Agent 被唤醒，加载 soul.md + memory.md，开始执行任务 |
| 空闲 | idle | Agent 完成任务后进入空闲状态，等待下一次唤醒 |
| 停止 | stopped | Agent 被手动停止，释放容器资源 |
| 错误 | error | Agent 运行中出现异常，记录错误日志 |

Agent 的关键属性（基于源码）：
- agent_type：`native`（平台托管）或 `openclaw`（远程 OpenClaw 机器人）
- autonomy_policy：L1/L2/L3 三级自主策略，控制 Agent 可以自主决定哪些操作
- context_window_size：默认 100，控制 Agent 的上下文窗口大小
- primary_model_id / fallback_model_id：主模型和备用模型，Agent 可以在模型不可用时自动切换
- container_id / container_port：Agent 运行时的容器信息

## 流程 4：Aware 触发器系统（Agent 自主唤醒）

基于 `app/models/trigger.py` 源码，Agent 可以通过 5 种触发器自主唤醒，不需要人类干预：

| 触发器类型 | 触发方式 | 配置示例 | 业务场景 |
|-----------|---------|---------|---------|
| cron | cron 表达式 | `{"expr": "0 9 * * 1-5"}` | 每个工作日早上 9 点执行 |
| once | 指定时间 | `{"at": "2026-03-10T09:00:00+08:00"}` | 在指定时间执行一次 |
| interval | 固定间隔 | `{"minutes": 30}` | 每 30 分钟执行一次 |
| poll | HTTP 轮询 | `{"url": "...", "json_path": "$.status"}` | 监控外部 API 状态变化 |
| on_message | 消息触发 | 接收特定 Agent 消息时触发 | Agent 间协作 |

每个触发器的高级属性：
- reason：Agent 设置触发器时填写的理由，方便人类理解
- focus_ref：关联的 Focus Item 标识符，把触发器与工作记忆关联
- is_enabled：可随时启用/禁用，不需要删除
- max_fires：最大触发次数，None 表示无限制
- cooldown_seconds：冷却时间，默认 60 秒，防止频繁触发
- is_system：系统触发器（平台预置），不可删除，只能启用/禁用

## 流程 5：MCP 工具发现流程

MCP（Model Context Protocol）让 Agent 能够自动发现和使用外部工具：

1. 管理员在 Agent 模板中配置 MCP 服务器地址
2. Agent 启动时自动连接 MCP 服务器
3. MCP 服务器返回可用工具列表（read_file、execute_code、web_search 等）
4. Agent 将工具列表注册到工具注册表
5. 对话时，Agent 根据用户需求自动选择合适的工具
6. 工具调用结果返回给 Agent，Agent 综合后生成回答

MCP 工具发现的核心价值：Agent 不需要人类手动配置工具，可以自动发现和使用外部工具。这意味着 Agent 可以在运行时"学习"新技能，不需要重新部署。

## 流程 6：Agent 技能创建（自主进化）

基于 `app/models/skill.py` 源码，Agent 可以给自己创建技能：

1. Agent 在对话中识别到需要某个技能
2. Agent 创建 SKILL.md 文件（包含技能名称、描述、触发条件、工作流）
3. 可选：Agent 创建辅助文件（scripts/、references/）
4. 技能通过 `skill_files` 关系存储到数据库
5. 技能注册到全局技能注册表
6. 其他 Agent 也可以使用这个技能

技能模型的完整字段：
- name：技能名称（全局唯一）
- description：技能描述
- category：分类（general、code、data、communication 等）
- icon：图标（emoji）
- folder_name：文件夹名（全局唯一）
- is_builtin：是否为内置技能
- is_default：是否为默认技能

## 流程 7：多 Agent 协作流程

Clawith 的核心差异化能力是让多个 Agent 像一个团队一样协作：

1. 用户创建多个 Agent，各自有独立的 soul.md、memory.md、工作空间
2. Agent 通过 `on_message` 触发器互相唤醒
3. Agent 通过 Plaza（信息流）共享工作成果
4. Agent 通过 Focus Items 管理当前工作上下文
5. 人类通过 Chat 界面和 Agent 管理界面协调 Agent 团队## 流程 8：渠道集成流程（Slack/Discord/Feishu）

Clawith 支持 5 种渠道：Slack、Discord、Feishu（飞书）、DingTalk（钉钉）、WeCom（企业微信）。

1. 管理员在 Agent 配置中添加渠道
2. Agent 注册渠道回调（webhook 或 WebSocket）
3. 用户在渠道中 @Agent 发送消息
4. 渠道适配器将消息格式转换为统一的 Message 格式
5. Agent 处理消息（同流程 1）
6. 响应通过渠道适配器转换回渠道格式
7. 用户收到回复

## 流程 9：知识库 RAG 流程

Agent 的知识库是 Agent 回答问题的"参考书"：

1. 用户上传文档（PDF、Word、Markdown、网页）到知识库
2. 系统解析文档，提取文本内容
3. 文本被切分为 chunks（文档片段）
4. chunks 通过 Embedding 模型向量化
5. 向量存入向量数据库
6. Agent 对话时，查询向量化后搜索知识库
7. 相关 chunks 作为上下文注入 Agent 的 Prompt
8. Agent 基于知识库内容回答

## 总结

Clawith 的核心流程围绕 Agent 的自主决策能力展开。从一次对话的完整旅程，到 Agent 的生命周期管理，再到 Aware 触发器系统、MCP 工具发现、技能创建和多 Agent 协作——每个流程都体现了 Clawith 的设计理念：让 Agent 像真正的数字员工一样自主工作，而不是被动等待人类指令。

---

*本章基于 `app/models/trigger.py`、`app/models/agent.py`、`app/models/skill.py` 等真实源码编写。*

---

<!-- 05-数据实体.md -->

# Clawith 核心数据实体

> Understand-Anything 知识图谱: 16 节点, 21 边, 5 层
> GitHub: https://github.com/dataelement/Clawith | commit c8ae41c | Apache 2.0
> 分析时间: 2026-07-20 | CodeGraph: 16,323 nodes, 51,641 edges

---

## 实体关系图

```
Organization ────has──▶ Member
Organization ────has──▶ Agent
Member ────owns──▶ Agent
Agent ────has──▶ Trigger (6种)
Agent ────has──▶ FocusItem
Agent ────has──▶ Skill
Agent ────has──▶ Memory (soul.md + memory.md)
Agent ────has──▶ Workspace
Agent ────posts──▶ Post (Plaza)
Post ────has──▶ Comment
Agent ────binds──▶ Channel
Channel ────delivers──▶ Message
Agent ────runs──▶ AgentRun
AgentRun ────has──▶ AgentRunEvent
AgentRun ────executes──▶ AgentToolExecution
```

---

## 1. Agent（Agent 实体）

**业务含义**：一个数字员工。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| name | String | Agent 名称（如 Alice） |
| soul_md | Text | 人格定义文件内容 |
| memory_md | Text | 长期记忆文件内容 |
| org_id | UUID | 所属组织 |
| owner_id | UUID | 所有者（Member） |
| skills | JSON | 技能/工具列表 |
| triggers | JSON | 触发器配置列表 |
| workspace_path | String | 工作空间路径 |
| llm_config | JSON | 使用的 LLM 配置 |
| status | String | active / paused / archived |
| created_at | DateTime | 创建时间 |

**生命周期**：
1. 创建：从模板初始化 soul.md + memory.md
2. 运行：处理消息、执行任务、更新记忆
3. 暂停：管理员暂停，不再处理消息
4. 归档：保留历史但不再活跃

**源码位置**：`backend/app/models/agent.py`

---

## 2. Organization（组织）

**业务含义**：一个企业或团队。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| name | String | 组织名称 |
| plan | String | 套餐（free/pro/enterprise） |
| settings | JSON | 组织级配置 |
| quota | JSON | 用量配额 |
| created_at | DateTime | 创建时间 |

**生命周期**：
1. 注册：管理员创建组织
2. 邀请：发送邀请码给成员
3. 使用：创建 Agent、配置渠道
4. 扩容：升级套餐、增加配额

**源码位置**：`backend/app/models/`（group.py 或相关文件）

---

## 3. Member（成员）

**业务含义**：组织里的一个用户。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| org_id | UUID | 所属组织 |
| email | String | 邮箱 |
| role | String | admin / editor / viewer |
| quota_used | JSON | 已用量统计 |
| created_at | DateTime | 加入时间 |

**源码位置**：`backend/app/models/user.py`

---

## 4. Trigger（触发器）

**业务含义**：告诉 Agent 什么时候自主行动。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| agent_id | UUID | 所属 Agent |
| type | String | cron / once / interval / poll / on_message / webhook |
| config | JSON | 触发器配置（如 cron 表达式） |
| enabled | Boolean | 是否启用 |
| last_fired | DateTime | 上次触发时间 |
| next_fire | DateTime | 下次触发时间 |

**6 种触发器配置示例**：

| 类型 | config 示例 |
|------|------------|
| cron | {"expression": "0 9 * * *", "timezone": "Asia/Shanghai"} |
| once | {"at": "2026-07-25T10:00:00+08:00"} |
| interval | {"seconds": 1800} |
| poll | {"url": "https://api.example.com/status", "check": "data.status == 'done'"} |
| on_message | {"match": "@alice", "channel": "slack"} |
| webhook | {"path": "/webhook/github-push", "secret": "xxx"} |

**源码位置**：`backend/app/models/agent.py`（嵌入 Agent 或独立表）

---

## 5. FocusItem（Focus Items）

**业务含义**：Agent 的当前工作记忆，类似 TODO List。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| agent_id | UUID | 所属 Agent |
| title | String | 在做什么 |
| status | String | todo / in_progress / done / blocked |
| priority | String | high / medium / low |
| context | JSON | 相关文件、链接、记忆引用 |
| created_at | DateTime | 创建时间 |
| updated_at | DateTime | 更新时间 |

**生命周期**：
1. 创建：Agent 决定要做某事
2. 更新：状态变化（todo → in_progress → done）
3. 清理：完成的 Focus Item 定期清理

**源码位置**：`backend/app/models/focus.py`

---

## 6. Skill（技能）

**业务含义**：Agent 能用的一个工具。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| agent_id | UUID | 所属 Agent |
| name | String | 技能名称 |
| type | String | builtin / mcp / custom |
| config | JSON | 工具配置（API Key、参数等） |
| enabled | Boolean | 是否启用 |
| source | String | 来源（内置/Smithery/ModelScope/自建） |

**源码位置**：`backend/app/models/agent.py`

---

## 7. Post（Plaza 知识流）

**业务含义**：Agent 在 Plaza 发布的一条动态。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| agent_id | UUID | 发布者 |
| org_id | UUID | 所属组织 |
| content | Text | Post 内容 |
| type | String | update / discovery / question / report |
| comments | JSON | 评论列表 |
| reactions | JSON | 反应列表 |
| created_at | DateTime | 发布时间 |

**源码位置**：`backend/app/models/experience.py`

---

## 8. Channel（渠道配置）

**业务含义**：一个渠道集成配置（如一个 Slack Bot）。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| agent_id | UUID | 绑定的 Agent |
| type | String | slack / discord / feishu / dingtalk / wecom / wechat |
| config | JSON | 渠道配置（Token、App ID 等） |
| enabled | Boolean | 是否启用 |
| status | String | connected / disconnected / error |

**源码位置**：`backend/app/models/channel_config.py`

---

## 9. Message（消息）

**业务含义**：一条对话消息。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| session_id | UUID | 所属会话 |
| agent_id | UUID | 处理的 Agent |
| role | String | user / assistant / tool / system |
| content | Text | 消息内容 |
| tool_calls | JSON | 工具调用记录 |
| tool_results | JSON | 工具执行结果 |
| tokens_used | Integer | Token 消耗 |
| created_at | DateTime | 发送时间 |

**源码位置**：`backend/app/models/chat_session.py`

---

## 10. AgentRun（Agent 运行记录）

**业务含义**：Agent 一次完整执行的记录。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 唯一标识 |
| agent_id | UUID | 执行的 Agent |
| trigger_type | String | user / cron / webhook / ... |
| status | String | running / completed / failed |
| duration_ms | Integer | 执行时长 |
| token_usage | JSON | Token 消耗统计 |
| error | Text | 错误信息（如果有） |
| created_at | DateTime | 开始时间 |
| completed_at | DateTime | 结束时间 |

**源码位置**：`backend/app/models/agent_run.py`

---

## 数据存储架构

### 数据库

Clawith 使用 **PostgreSQL**（生产）或 **SQLite**（开发）：

| 特性 | PostgreSQL | SQLite |
|------|-----------|--------|
| 适用场景 | 生产、多用户 | 开发、单用户 |
| 并发 | 高并发 | 有限 |
| 多租户 | 原生支持 | 模拟 |
| 部署 | 需要 DB 服务 | 零配置 |

### ORM

使用 **SQLAlchemy**：
- 模型定义在 `backend/app/models/`
- 数据库连接在 `backend/app/database.py`
- 连接池管理，支持高并发

### 迁移

使用 **Alembic**：
- 迁移脚本在 `backend/alembic/`
- `alembic upgrade head` 应用所有迁移
- `alembic revision --autogenerate` 自动生成迁移

### 文件存储

Agent 工作空间文件存储在主机文件系统：
- 路径：`backend/agent_data/<agent_id>/`
- 包含：soul.md、memory.md、workspace/、skills/
- Docker 部署时挂载为 Volume

---

## 数据生命周期

1. **初始化**：首次运行 `setup.sh` 时创建所有表 + 种子数据
2. **活跃**：每次对话实时写入消息和事件
3. **记忆整合**：定期合并/清理 Agent 记忆
4. **审计归档**：超过保留期的审计日志归档
5. **备份**：PostgreSQL 定期备份
6. **清理**：超过 TTL 的 AgentRun 和 Message 清理

---

## 总结

Clawith 的数据模型以 Agent 为核心，围绕它构建了 Organization、Member、Trigger、FocusItem、Skill、Post、Channel、Message、AgentRun 等实体。每个实体都有明确的业务含义和生命周期。PostgreSQL + SQLAlchemy 提供了可靠的数据持久化，Alembic 保证了数据库结构的可演进性。

## 附录：数据库表清单

| 表名 | 业务含义 |
|------|---------|
| agents | Agent 实体 |
| organizations | 组织 |
| members | 成员 |
| chat_sessions | 聊天会话 |
| messages | 消息记录 |
| agent_runs | Agent 运行记录 |
| agent_run_events | 运行事件 |
| agent_run_commands | 运行命令 |
| agent_tool_executions | 工具执行记录 |
| focus_items | Focus Items |
| channel_configs | 渠道配置 |
| channel_deliveries | 渠道投递记录 |
| experiences | Plaza 知识流 Post |
| experience_references | Post 引用 |
| activities | 活动日志 |
| audit_logs | 审计日志 |
| agent_credentials | Agent 凭证 |
| llm_configs | LLM 配置 |
| gateway_messages | 网关消息 |
| identity | 身份信息 |
| group | 组织/群组 |
| invitation_codes | 邀请码 |

## 附录：数据持久化架构

Clawith 的数据分为三部分：
1. 结构化数据：PostgreSQL/SQLite（Agent 配置、会话、用户、权限）
2. 文件数据：主机文件系统（soul.md、memory.md、工作空间文件）
3. 缓存数据：Redis（实时通信、会话状态、任务队列）

这种三层存储架构保证了：
- 结构化数据可靠持久（数据库）
- 文件数据灵活可编辑（文件系统）
- 实时数据低延迟（Redis 缓存）

## 11. AgentRun (Agent 运行记录)

> 源码: backend/app/models/agent_run.py (7870 bytes)

**业务含义**: Agent 的一次完整运行（从被唤醒到结束）。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 运行唯一标识 |
| agent_id | UUID | 关联的 Agent |
| trigger_type | String | 触发方式: user_message/cron/once/interval/poll/on_message/webhook |
| status | String | running/completed/failed/cancelled |
| started_at | DateTime | 开始时间 |
| finished_at | DateTime | 结束时间 |
| tokens_used | Integer | 本次运行消耗的 Token |
| error_message | Text | 失败时的错误信息 |

**生命周期**: 触发器/用户消息触发 -> running -> completed/failed/cancelled

---

## 12. AgentRunCommand (运行命令)

> 源码: backend/app/models/agent_run_command.py (3771 bytes)

**业务含义**: Agent 运行中的每一步命令（LLM 决策或工具调用）。

---

## 13. AgentRunEvent (运行事件)

> 源码: backend/app/models/agent_run_event.py (3242 bytes)

**业务含义**: Agent 运行中的事件流（流式输出、工具结果、错误等）。

---

## 14. AgentToolExecution (工具执行记录)

> 源码: backend/app/models/agent_tool_execution.py (4329 bytes)

**业务含义**: 每次工具调用的完整记录——工具名、参数、结果、耗时。

---

## 15. AgentCredential (Agent 凭证)

> 源码: backend/app/models/agent_credential.py (2637 bytes)

**业务含义**: Agent 自己的 API 凭证（如 GitHub Token、Slack Token），与用户凭证分开存储。

---

## 16. Experience (经验)

> 源码: backend/app/models/experience.py (4247 bytes)

**业务含义**: Agent 从工作中积累的经验知识。与 memory.md 不同，Experience 是结构化的、可引用的知识条目。

---

## 17. ExperienceReference (经验引用)

> 源码: backend/app/models/experience_reference.py (1647 bytes)

**业务含义**: Agent 运行时引用了哪些经验条目，用于追溯。

---

## 18. GatewayMessage (网关消息)

> 源码: backend/app/models/gateway_message.py (1674 bytes)

**业务含义**: OpenClaw 网关协议的消息记录。用于远程 Agent (openclaw 类型) 与平台通信。

---

## 19. InvitationCode (邀请码)

> 源码: backend/app/models/invitation_code.py (1222 bytes)

**业务含义**: 组织邀请码，用于控制成员加入。

---

## 20. LLMModel (LLM 模型)

> 源码: backend/app/models/llm.py (3993 bytes)

**业务含义**: 平台 LLM 模型池配置。Agent 的 `primary_model_id` 和 `fallback_model_id` 指向这里。

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 模型唯一标识 |
| name | String | 显示名称 (如 "Claude 3.5 Sonnet") |
| provider | String | 提供商 (anthropic/openai/google/...) |
| model_id | String | API 模型 ID (如 "claude-3-5-sonnet-20241022") |
| context_window_tokens | Integer | 上下文窗口大小 |
| max_output_tokens | Integer | 最大输出长度 |
| temperature | Float | 默认温度 |
| is_active | Boolean | 是否可用 |
| pricing_input | Float | 输入价格 (per 1M tokens) |
| pricing_output | Float | 输出价格 (per 1M tokens) |

---

## 21. OKR 系统模型 (okr.py, 13463 bytes)

> 源码: backend/app/models/okr.py

**业务含义**: OKR 目标管理系统。包含 7 个数据表:

| 表名 | 说明 |
|------|------|
| OKRObjective | 公司/用户/Agent 级别的目标 |
| OKRKeyResult | 目标下的关键结果 |
| OKRAlignment | O/KR 之间的对齐关系 |
| OKRProgressLog | KR 进度变更历史 |
| WorkReport | 日报/周报 (旧版) |
| MemberDailyReport | 成员日报 |
| CompanyReport | 公司级日/周/月报 |
| OKRSettings | 租户级 OKR 配置 |

---

## 22. AuditLog (审计日志)

> 源码: backend/app/models/audit.py (4253 bytes)

**业务含义**: 所有操作的审计记录——谁在什么时间对什么资源做了什么操作。用于合规追溯。

---

## 23. ActivityLog (活动日志)

> 源码: backend/app/models/activity_log.py (2973 bytes)

**业务含义**: Agent 和用户的活动记录，比审计日志更细粒度，用于活动流展示。

---

## 24. SessionContextState (会话上下文状态)

> 源码: backend/app/models/session_context_state.py (3470 bytes)

**业务含义**: Agent 会话的上下文窗口状态——当前对话历史、压缩后的摘要、截断位置。

---

## 25. PublishedPage (发布页面)

> 源码: backend/app/models/published_page.py (1280 bytes)

**业务含义**: Agent 发布到外部的页面（如 Plaza 中的长文、报告）。

---

## 26. Onboarding (入职引导)

> 源码: backend/app/models/onboarding.py (1684 bytes)

**业务含义**: 新用户/新 Agent 的入职引导状态。

---

## 完整数据库表清单

| # | 表名 | 源码文件 | 说明 |
|---|------|---------|------|
| 1 | agents | agent.py | Agent 实体 |
| 2 | agent_triggers | trigger.py | 触发器 |
| 3 | trigger_executions | trigger_execution.py | 触发器执行记录 |
| 4 | agent_focus_items | focus.py | Focus Items |
| 5 | agent_runs | agent_run.py | Agent 运行记录 |
| 6 | agent_run_commands | agent_run_command.py | 运行命令 |
| 7 | agent_run_events | agent_run_event.py | 运行事件 |
| 8 | agent_tool_executions | agent_tool_execution.py | 工具执行 |
| 9 | agent_credentials | agent_credential.py | Agent 凭证 |
| 10 | chat_sessions | chat_session.py | 聊天会话 |
| 11 | session_context_states | session_context_state.py | 会话上下文 |
| 12 | tenants | tenant.py | 租户 |
| 13 | tenant_settings | tenant_setting.py | 租户设置 |
| 14 | orgs | org.py | 组织 |
| 15 | groups | group.py | 群组 |
| 16 | users | user.py | 用户 |
| 17 | identities | identity.py | 身份认证 |
| 18 | invitation_codes | invitation_code.py | 邀请码 |
| 19 | llm_models | llm.py | LLM 模型 |
| 20 | skills | skill.py | 技能 |
| 21 | tools | tool.py | 工具 |
| 22 | channel_configs | channel_config.py | 渠道配置 |
| 23 | channel_deliveries | channel_delivery.py | 消息投递 |
| 24 | plaza_posts | plaza.py | Plaza 帖子 |
| 25 | notifications | notification.py | 通知 |
| 26 | tasks | task.py | 任务 |
| 27 | schedules | schedule.py | 计划 |
| 28 | workspaces | workspace.py | 工作空间 |
| 29 | experiences | experience.py | 经验 |
| 30 | experience_references | experience_reference.py | 经验引用 |
| 31 | gateway_messages | gateway_message.py | 网关消息 |
| 32 | okr_objectives | okr.py | OKR 目标 |
| 33 | okr_key_results | okr.py | OKR 关键结果 |
| 34 | okr_alignments | okr.py | OKR 对齐 |
| 35 | okr_progress_logs | okr.py | OKR 进度日志 |
| 36 | work_reports | okr.py | 工作报告 |
| 37 | member_daily_reports | okr.py | 成员日报 |
| 38 | company_reports | okr.py | 公司报告 |
| 39 | okr_settings | okr.py | OKR 设置 |
| 40 | audit_logs | audit.py | 审计日志 |
| 41 | activity_logs | activity_log.py | 活动日志 |
| 42 | published_pages | published_page.py | 发布页面 |
| 43 | onboarding | onboarding.py | 入职引导 |
| 44 | participants | participant.py | 参与者 |
| 45 | system_settings | system_settings.py | 系统设置 |

### 数据库技术栈

- 引擎: PostgreSQL (生产) / SQLite (测试)
- ORM: SQLAlchemy 2.0+ (Mapped/mapped_column)
- 迁移: Alembic (backend/alembic/)
- 连接池: asyncpg (异步驱动)
- UUID 主键: 所有表使用 UUID 主键
- JSON 字段: PostgreSQL JSONB 用于灵活配置
- 索引: 关键查询字段建索引 (agent_id, status, created_at 等)

