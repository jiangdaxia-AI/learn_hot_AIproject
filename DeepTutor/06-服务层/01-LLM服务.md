# LLM 服务层源码剖析

> Understand-Anything 知识图谱: 26 节点, 25 边, 7 层
> **源码路径**：`deeptutor/services/llm/`
> **核心入口**：`__init__.py`（重新导出工厂函数）、`factory.py`（统一工厂）、`provider_core/openai_compat_provider.py`（OpenAI 兼容提供者）、`provider_core/anthropic_provider.py`（Anthropic 提供者）、`local_provider.py`（本地模型提供者）
> **阅读对象**：产品经理 / 运营 / 非技术人员

---

## 一、概述：LLM 服务层是什么？

LLM 服务层是 DeepTutor 与所有大语言模型（LLM）对话的"翻译官"和"调度员"。它把上层 Agent 的调用请求（"帮我问问模型这个问题的答案"）统一翻译成各模型提供商能理解的 API 请求，然后把响应再翻译回 DeepTutor 能处理的格式。

**一句话理解**：不管你的模型是 OpenAI 的 GPT-4o、Anthropic 的 Claude、本地部署的 Qwen，还是通过 OpenRouter 调用的第三方模型，DeepTutor 的上层代码都不需要关心差别——LLM 服务层帮你全部搞定。

### 核心职责

| 职责 | 说明 |
|------|------|
| **提供商适配** | 支持 OpenAI、Anthropic、Azure、DeepSeek、硅基流动、本地模型等 20+ 种提供商 |
| **流式/非流式** | 同时支持两种模式，流式模式下逐字推送给前端 |
| **Token 管理** | 上下文窗口守卫、token 用量追踪、上下文超限自动裁剪 |
| **能力检测** | 自动检测每个模型是否支持工具调用、视觉输入、JSON 格式输出等 |
| **流量控制** | 并发限制 + 速率限制（Token Bucket 算法），防止 API 超额 |
| **故障恢复** | 指数退避重试、流式失败自动降级为非流式、响应格式错误自动回退 |

---

## 二、架构全景图

```
┌─────────────────────────────────────────────────────┐
│                   Agent 层                           │
│  (ChatAgent, SolveAgent, 等)                        │
└──────────────────────┬──────────────────────────────┘
                       │ complete() / stream()
                       ▼
┌─────────────────────────────────────────────────────┐
│              LLM 工厂 (factory.py)                   │
│  ┌─────────────────────────────────────────────┐    │
│  │  resolve_provider_spec()                    │    │
│  │  根据 binding / model / base_url 确定提供商  │    │
│  └─────────────────────────────────────────────┘    │
└──────────┬──────────────────────┬───────────────────┘
           │                      │
           ▼                      ▼
┌──────────────────┐   ┌──────────────────────┐
│  OpenAICompat-   │   │  LocalProvider       │
│  Provider        │   │  (local_provider.py) │
│  (openai_compat) │   │  aiohttp → LM Studio │
│                  │   │  Ollama, vLLM 等     │
└────────┬─────────┘   └──────────┬───────────┘
         │                        │
         ▼                        ▼
┌──────────────────────────────────────────────────┐
│              能力检测 (capabilities.py)           │
│  supports_tools() / supports_vision() / ...      │
│  PROVIDER_CAPABILITIES + MODEL_OVERRIDES         │
└──────────────────────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────────────┐
│            流量控制 (traffic_control.py)           │
│  TrafficController: 并发信号量 + Token Bucket    │
└──────────────────────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────────────────┐
│          上下文窗口 (context_window.py)            │
│  模型窗口大小推算 + 90% 安全边界守卫              │
└──────────────────────────────────────────────────┘
```

---

## 三、提供商体系：支持哪些模型？

### 3.1 提供商注册表（provider_registry.py）

DeepTutor 通过 `deeptutor/services/provider_registry.py` 维护一个中心化的提供商注册表。每个提供商（Provider）都是一条记录，包含：

- **name**：唯一标识，如 `"openai"`、`"deepseek"`、`"siliconflow"`
- **binding**：SDK 绑定类型，决定用哪个底层 SDK 发起请求
- **default_api_base**：默认 API 地址
- **env_key**：存储 API Key 的环境变量名
- **default_models**：该提供商可用的模型列表
- **supports_prompt_caching**：是否支持提示词缓存（节省 token）
- **strip_model_prefix**：是否需要去掉模型名前缀（如 OpenRouter 的 `openai/gpt-4o` → `gpt-4o`）

**源码位置**：`deeptutor/services/provider_registry.py`

### 3.2 两大类提供商

#### A. OpenAI 兼容型（OpenAICompatProvider）

**源码位置**：`deeptutor/services/llm/provider_core/openai_compat_provider.py`（856 行）

这是最核心的提供者。任何遵循 OpenAI Chat Completions API 格式的提供商都走这个通道。

**支持的提供商**（部分列举）：

| 提供商 | binding | 说明 |
|--------|---------|------|
| OpenAI | `openai` | GPT-4o, GPT-5, o1, o3, o4 等 |
| Azure OpenAI | `azure_openai` | 企业级 Azure 部署 |
| GitHub Copilot | `github_copilot` | 开发者常用的 Copilot 模型 |
| DeepSeek | `deepseek` | DeepSeek-V3, DeepSeek-R1 等 |
| 硅基流动 | `siliconflow` | 国产模型托管平台 |
| Moonshot/Kimi | `moonshot` | 月之暗面 Kimi 系列 |
| MiniMax | `minimax` | MiniMax 系列模型 |
| OpenRouter | `openrouter` | 聚合多家模型的代理平台 |
| 自定义端点 | `custom` | 用户自建 OpenAI 兼容服务 |

**关键设计决策：Responses API 自动切换**

OpenAI 新推出的 Responses API 与旧的 Chat Completions API 不同。对于需要推理能力的模型（gpt-5、o1、o3、o4），`OpenAICompatProvider` 会自动尝试使用 Responses API；如果失败，则自动回退到 Chat Completions API。

这个切换逻辑由 `_should_use_responses_api()` 方法控制（第 289 行），并且内置了**熔断器**（Circuit Breaker）：连续失败 2 次后，5 分钟内不再尝试 Responses API，避免持续浪费请求。

> **业务理解**：你可以把这想象成"智能路由"——如果新高速路堵了，自动走老路，并且记住这条路堵了，5 分钟内不再尝试。

#### B. Anthropic 提供者（AnthropicProvider）

**源码位置**：`deeptutor/services/llm/provider_core/anthropic_provider.py`（565 行）

专门为 Anthropic Claude 系列模型（Claude 3.5 Sonnet、Claude 4 Opus 等）编写的提供者。使用 Anthropic 官方 SDK。

与 OpenAI 兼容型的关键区别：

| 特性 | OpenAI 兼容型 | Anthropic |
|------|:---:|:---:|
| System Prompt 位置 | 放在 messages 数组里 | 独立参数 |
| 响应格式 | 支持 response_format | 不支持 |
| 视觉输入 | 支持 URL + base64 | 仅支持 base64 |
| 提示词缓存 | ephemeral 类型 | 独立 cache_control |

#### C. 本地提供者（LocalProvider）

**源码位置**：`deeptutor/services/llm/local_provider.py`（390 行）

为本地部署的模型设计。使用 aiohttp 而非 httpx（因为 httpx 对某些本地服务器如 LM Studio 有已知的 502 兼容性问题）。

**支持的本地服务器**：
- **LM Studio**：桌面端本地模型运行工具
- **Ollama**：命令行本地模型管理工具
- **vLLM**：高性能推理引擎
- **llama.cpp**：轻量级 CPU 推理

**特殊处理**：
- 超时设为 300 秒（5 分钟），因为本地推理可能很慢
- 自动检测 Ollama 并调用 `/api/tags` 获取模型列表
- 流式模式下自动处理 ` thinking` 标签（推理模型如 Qwen 的思考过程）
- 流式失败自动降级为非流式

---

## 四、流式 vs 非流式：两种对话模式

### 4.1 业务场景

| 模式 | 使用场景 | 用户体验 |
|------|---------|---------|
| **流式（Stream）** | 聊天对话、Agent 循环 | 逐字输出，类似 ChatGPT 打字效果 |
| **非流式（Complete）** | 后台任务、记忆合并、RAG 查询生成 | 等待完整响应后一次性返回 |

### 4.2 流式实现细节

**源码位置**：`openai_compat_provider.py` 的 `chat_stream()` 方法（第 561 行）

流式模式下，每个 LLM 返回的文本块（chunk）会通过回调函数 `on_content_delta` 实时推送到前端。同时，推理内容（reasoning_content，即模型的"思考过程"）通过 `on_reasoning_delta` 单独推送。

**AgentLoop 层面的流式处理**（`agent_loop.py` 第 369 行）：

```python
# 每个 chunk 到来时：
if content:
    text_parts.append(content)
    # 检查是否有  thinking 标签，分离思考内容和用户可见内容
    await _emit_segments(think_filter.feed(content))
```

`InlineThinkFilter` 是一个增量解析器，它实时检测 ` thinking` / `</thinking>` 标签：
- 标签内的内容 → 推送到 thinking 通道（侧边栏展示）
- 标签外的内容 → 推送到 content 通道（主对话区展示）

> **业务理解**：这就像把模型的自言自语和正式回答分开——思考过程放侧边栏供调试，正式回答放主聊天区。

### 4.3 非流式实现细节

**源码位置**：`openai_compat_provider.py` 的 `chat()` 方法（第 499 行）

非流式模式直接调用 `client.chat.completions.create(stream=False)`，等待完整响应后解析。解析逻辑包括：
- 提取文本内容
- 提取工具调用（tool_calls）并解析 JSON 参数
- 提取推理内容
- 提取 token 用量统计

---

## 五、Token 管理

### 5.1 上下文窗口守卫

**源码位置**：`deeptutor/services/llm/context_window.py`（80 行）

每个模型有一个最大上下文窗口（如 GPT-4o 是 128K tokens）。DeepTutor 在发送请求前会估算当前消息的总 token 数，如果超过窗口的 90%，则触发守卫逻辑：

**守卫策略**（`agentic_pipeline.py` 第 1073 行 `_guard_context_window()`）：

```
预算 = 模型上下文窗口 × 0.9
如果 当前消息 token 数 > 预算:
    从最早的工具结果开始，逐个替换为"[内容已裁剪]"
    直到 token 数回到预算以内
```

> **业务理解**：就像写信时纸张有限，如果写不下了，就把之前查阅的资料（工具结果）摘要化，只保留结论，为新内容腾出空间。

**源码位置**：`agentic_pipeline.py` 第 1073-1103 行

### 5.2 Token 用量追踪

**源码位置**：`deeptutor/core/agentic/usage.py`（通过 `UsageTracker` 追踪）

每次 LLM 调用后，DeepTutor 记录：
- **prompt_tokens**：输入消耗的 token
- **completion_tokens**：输出消耗的 token
- **total_tokens**：总计

这些数据用于：
- 成本核算（不同模型价格不同）
- 用量监控（防止异常消耗）
- 前端展示（"本次对话消耗 xxx tokens"）

### 5.3 上下文窗口大小推算

**源码位置**：`context_window.py` 第 65 行 `resolve_effective_context_window()`

当模型没有明确声明上下文窗口大小时，DeepTutor 会智能推算：

| 模型系列 | 默认窗口 |
|---------|---------|
| GPT-4o / Claude / Gemini / DeepSeek / Kimi 等 | 65,536 tokens |
| 其他模型 | 16,384 tokens |
| 用户手动配置 | 取配置值（上限 1,000,000） |

---

## 六、能力检测系统

**源码位置**：`deeptutor/services/llm/capabilities.py`（562 行）

### 6.1 能力矩阵

DeepTutor 维护了一个两层的能力配置表：

**第一层：提供商级别**（`PROVIDER_CAPABILITIES`，第 30 行）

| 能力 | 说明 | 示例 |
|------|------|------|
| `supports_response_format` | 是否支持 JSON 结构化输出 | OpenAI ✅ / DeepSeek ❌ |
| `supports_streaming` | 是否支持流式输出 | 几乎所有 ✅ |
| `supports_tools` | 是否支持函数调用/工具 | 主流模型 ✅ |
| `supports_vision` | 是否支持图片输入 | GPT-4o ✅ / DeepSeek ❌ |
| `system_in_messages` | System Prompt 是否放在 messages 里 | OpenAI ✅ / Anthropic ❌ |
| `has_thinking_tags` | 输出是否包含 ` thinking` 标签 | DeepSeek ✅ / GPT-4o ❌ |
| `requires_api_version` | 是否需要 API 版本参数 | Azure ✅ / 其他 ❌ |
| `vision_url_supported` | 视觉输入是否支持 URL（而非仅 base64） | OpenAI ✅ / Anthropic ❌ |

**第二层：模型级别**（`MODEL_OVERRIDES`，第 90 行起）

针对特定模型系列的特殊覆盖：

```python
"gpt-5": {"forced_temperature": 1.0},  # 推理模型只支持 temperature=1.0
"o1": {"forced_temperature": 1.0},
"o3": {"forced_temperature": 1.0},
"deepseek-r1": {"has_thinking_tags": True},
"qwen/qwen3-vl": {"has_thinking_tags": True, "supports_vision": True},
"gemma": {"supports_vision": False, "supports_response_format": False},
```

### 6.2 能力查询 API

```python
# 检查是否支持工具调用
supports_tools(binding="deepseek", model="deepseek-chat")  # → True

# 检查是否支持视觉输入
supports_vision(binding="openai", model="gpt-4o")  # → True

# 获取有效温度值（推理模型强制为 1.0）
get_effective_temperature(binding="openai", model="o1", requested_temp=0.7)  # → 1.0
```

> **业务理解**：这就像每个模型都有一张"能力身份证"，Agent 在调用前先查一下这张证，确认模型能不能做某件事，避免发出无效请求。

### 6.3 运行时动态禁用

**源码位置**：`capabilities.py` 第 513 行 `disable_response_format_at_runtime()`

某些模型在静态配置中说支持 JSON 格式，但实际运行时却拒绝（如 LM Studio 上的某些 Gemma 变体）。DeepTutor 设计了运行时缓存机制：

```python
# 当运行时发现 (binding, model) 不支持 response_format 时
disable_response_format_at_runtime("openai", "gemma-4-e2b")

# 后续调用自动跳过 response_format
supports_response_format("openai", "gemma-4-e2b")  # → False
```

**这个缓存在进程重启后清空**，确保如果模型升级后支持了该能力，不会被永久禁用。

---

## 七、流量控制（TrafficController）

**源码位置**：`deeptutor/services/llm/traffic_control.py`（125 行）

### 7.1 设计目标

防止两种情况：
1. **本地资源耗尽**：同时发起太多请求，系统内存/连接数不够
2. **API 速率限制**：超过提供商的 RPM（每分钟请求数）限制，被暂时封禁

### 7.2 双重保护机制

```
TrafficController
├── 并发控制（Semaphore）
│   └── 默认同时最多 20 个请求
│   └── 超时 30 秒后拒绝新请求
└── 速率限制（Token Bucket）
    └── 默认 600 RPM（每秒 10 个）
    └── 令牌不足时等待，不拒绝
```

### 7.3 Token Bucket 算法

简单理解：桶里初始有 600 个令牌，每秒补充 10 个。每个请求消耗 1 个令牌。如果令牌不够，请求等待直到有令牌可用。

> **业务理解**：就像医院挂号——同时只能有 20 个人在候诊区（并发限制），每小时最多挂 600 个号（速率限制）。超过并发限制的直接拒绝，超过速率限制的排队等待。

---

## 八、配置切换：如何换模型？

### 8.1 配置入口

**源码位置**：`deeptutor/services/llm/config.py` 的 `LLMConfig`

用户在前端设置页面（或配置文件）中配置以下字段：

| 配置项 | 说明 | 示例 |
|--------|------|------|
| `binding` | 提供商绑定 | `"openai"`, `"deepseek"`, `"anthropic"` |
| `model` | 模型名称 | `"gpt-4o"`, `"deepseek-chat"` |
| `api_key` | API 密钥 | `"sk-xxx"` |
| `base_url` | API 地址（可选） | `"https://api.openai.com/v1"` |
| `api_version` | API 版本（Azure 专用） | `"2024-02-15-preview"` |
| `reasoning_effort` | 推理深度 | `"low"`, `"medium"`, `"high"` |
| `context_window` | 上下文窗口覆盖 | `131072` |
| `max_tokens` | 最大输出 token | `4096` |

### 8.2 提供商解析流程

**源码位置**：`factory.py` 第 82 行 `_resolve_provider_spec()`

```
1. 用户指定了 binding（如 "deepseek"）
   → 直接用 find_by_name("deepseek") 查找注册表
   → 如果同时配了网关，优先用网关

2. 用户没指定 binding，但指定了 model
   → 用 find_by_model("deepseek-chat") 反向查找

3. 用户只给了 base_url
   → 检测是否是本地服务器（localhost:11434 → Ollama）
   → 否则用默认的 openai

4. 什么都没给
   → 回退到 openai
```

### 8.3 模型切换的业务含义

**对于产品经理**：用户可以在设置中自由切换模型，DeepTutor 会自动适配。例如：
- 切换到 DeepSeek → 自动禁用 JSON 结构化输出，自动处理 ` thinking` 标签
- 切换到 Claude → System Prompt 自动改为独立参数
- 切换到本地模型 → 超时自动延长到 5 分钟，失败自动降级

---

## 九、关键方法速查

| 方法 | 源码位置 | 功能 |
|------|---------|------|
| `complete()` | `factory.py` | 非流式 LLM 调用 |
| `stream()` | `factory.py` | 流式 LLM 调用 |
| `get_llm_config()` | `config.py` | 获取当前 LLM 配置 |
| `supports_tools()` | `capabilities.py` | 检查是否支持工具调用 |
| `supports_vision()` | `capabilities.py` | 检查是否支持图片输入 |
| `resolve_effective_context_window()` | `context_window.py` | 推算有效上下文窗口 |
| `TrafficController.__aenter__()` | `traffic_control.py` | 获取流量控制令牌 |
| `OpenAICompatProvider.chat()` | `openai_compat_provider.py:499` | OpenAI 兼容型非流式调用 |
| `OpenAICompatProvider.chat_stream()` | `openai_compat_provider.py:561` | OpenAI 兼容型流式调用 |
| `LocalProvider.complete()` | `local_provider.py:65` | 本地模型非流式调用 |
| `LocalProvider.stream()` | `local_provider.py:130` | 本地模型流式调用 |
| `_resolve_provider_spec()` | `factory.py:82` | 解析提供商规格 |
| `_should_use_responses_api()` | `openai_compat_provider.py:289` | 判断是否使用 Responses API |
| `_guard_context_window()` | `agentic_pipeline.py:1073` | 上下文窗口守卫 |

---

## 十、故障处理与容错

### 10.1 指数退避重试

**源码位置**：`factory.py` 第 67 行 `_build_retry_delays()`

```
默认配置：最多重试 3 次，基础延迟 1 秒，指数退避
第 1 次重试：等待 1 秒
第 2 次重试：等待 2 秒
第 3 次重试：等待 4 秒
单次延迟上限：120 秒
```

### 10.2 流式降级

**源码位置**：`local_provider.py` 第 230 行

如果流式请求失败（如网络中断），本地提供者会自动降级为非流式请求，确保用户至少能拿到结果。

### 10.3 响应格式回退

**源码位置**：`openai_compat_provider.py` 第 520 行

如果请求时带了 `response_format` 参数但模型不支持，自动去掉该参数重新请求。

### 10.4 工具格式错误回退

**源码位置**：`openai_compat_provider.py` 第 533 行

某些端点（如 DashScope Coding Plan）在非流式模式下对工具调用参数的 JSON 格式校验很严格，出错时自动切换到流式模式重试。

---

## 十一、总结

LLM 服务层是 DeepTutor 的"模型中立层"。它的核心价值在于：

1. **一套代码，适配所有模型**：20+ 种提供商，从云端到本地，从 OpenAI 到 Anthropic
2. **智能容错**：熔断器、自动降级、指数退避，确保用户不会因为模型问题而中断使用
3. **透明切换**：用户切换模型时，所有能力差异自动适配，无需手动配置
4. **资源保护**：流量控制防止 API 超额，上下文窗口守卫防止 token 超限

**一句话总结**：LLM 服务层让 DeepTutor 成为一个"模型无关"的 AI 教学平台——不管底层用什么模型，上层体验一致。

