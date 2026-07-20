# 06-Embedding 服务（Embedding Service）

> 把文字变成"数学坐标"——DeepTutor 的 Embedding 服务就像一个翻译官，把人类能读懂的文字翻译成计算机能计算的数字向量，让 AI 可以"理解"文本之间的相似度。

---

## 1. 整体概述

Embedding 服务是 DeepTutor 的"向量化引擎"。它的核心任务是把文本（一段话、一篇文章、一个问题）转换成一串数字（向量），这串数字就是文本的"数学指纹"。相似的文本会有相似的"指纹"，这样计算机就能通过比较向量之间的距离来判断"这两段话是不是在说同一件事"。

这个服务主要服务于 RAG（检索增强生成）——当用户问问题时，系统先把问题转成向量，然后在知识库中找向量最接近的文档片段，作为 AI 回答的参考资料。

**核心文件：**

| 文件 | 路径 | 职责 |
|------|------|------|
| `client.py` | `deeptutor/services/embedding/client.py` | Embedding 客户端，对外暴露的统一接口 |
| `config.py` | `deeptutor/services/embedding/config.py` | Embedding 配置管理 |
| `validation.py` | `deeptutor/services/embedding/validation.py` | 向量质量验证 |
| `adapters/__init__.py` | `deeptutor/services/embedding/adapters/__init__.py` | 适配器注册表 |
| `adapters/base.py` | `deeptutor/services/embedding/adapters/base.py` | 适配器抽象基类和数据模型 |

**源码位置：** `/Users/a1021500266/Documents/learn/DeepTutor/deeptutor/services/embedding/`

---

## 2. 数据模型（adapters/base.py）

*文件：`deeptutor/services/embedding/adapters/base.py`*

### 2.1 `EmbeddingRequest`（Embedding 请求）

*文件位置：base.py，第 31-69 行*

```python
@dataclass
class EmbeddingRequest:
    texts: List[str]                    # 要向量化的文本列表
    model: str                          # 使用的模型名称
    dimensions: Optional[int] = None    # 输出向量维度
    input_type: Optional[str] = None    # 输入类型提示（如 "search_document"）
    encoding_format: Optional[str] = "float"  # 输出格式
    truncate: Optional[bool] = True     # 是否截断超长文本
    normalized: Optional[bool] = True   # 是否 L2 归一化
    late_chunking: Optional[bool] = False  # 延迟分块（Jina v3）
    contents: Optional[List[Dict]] = None  # 多模态内容（图文混合）
    enable_fusion: Optional[bool] = None   # DashScope 多模态融合
```

**业务含义：** EmbeddingRequest 是"向量化订单"——你告诉系统"我要把这些文本转成向量，用这个模型，向量维度是多少，有什么特殊要求"。

**关键字段：**
- `texts`：要向量化的文本列表。可以一次传多条，批量处理更高效
- `model`：使用哪个 Embedding 模型。不同模型产生不同"指纹"，更换模型需要重新索引知识库
- `dimensions`：输出向量的维度（如 768、1024、1536）。维度越高，表达能力越强，但计算成本也越大
- `input_type`：告诉模型这段文本的用途——是"要搜索的文档"还是"用户的搜索查询"。某些模型（如 Cohere）会根据用途调整向量
- `contents`：多模态内容，支持文本+图片混合输入。例如 `[{"text": "一段话"}, {"image": "data:image/png;base64,..."}]`
- `truncate`：如果文本太长，是否自动截断

**大白话解释：** 这是"翻译订单"——告诉翻译官，要把哪些文字翻译成什么格式的数学语言。

### 2.2 `EmbeddingResponse`（Embedding 响应）

*文件位置：base.py，第 72-79 行*

```python
@dataclass
class EmbeddingResponse:
    embeddings: List[List[float]]  # 向量列表
    model: str                     # 实际使用的模型
    dimensions: int                # 实际向量维度
    usage: Dict[str, Any]          # 用量信息（token 消耗等）
```

**业务含义：** EmbeddingResponse 是"翻译结果"——系统返回每个文本对应的向量，以及用了多少 token。

**大白话解释：** 这是"翻译好的密码本"——每段文本对应一串数字，还告诉你了这次翻译花了多少成本。

### 2.3 `EmbeddingProviderError`（Embedding 错误）

*文件位置：base.py，第 82-121 行*

```python
class EmbeddingProviderError(RuntimeError):
    def __init__(self, message, *, status, body, model, url, provider):
        ...
```

**业务含义：** 当 Embedding 服务调用失败时，抛出这个错误。它携带了 HTTP 状态码、响应内容、模型名、请求 URL 等详细信息，方便排查问题。

**大白话解释：** 这是"翻译失败报告"——详细记录了为什么翻译失败，是网络问题还是模型报错，方便技术人员排查。

### 2.4 `BaseEmbeddingAdapter`（Embedding 适配器基类）

*文件位置：base.py，第 123-180 行*

```python
class BaseEmbeddingAdapter(ABC):
    def __init__(self, config: Dict[str, Any]):
        self.api_key = config.get("api_key")
        self.base_url = config.get("base_url")
        self.model = config.get("model")
        ...
```

**业务含义：** 这是所有 Embedding 提供商的"统一接口"。不同的 Embedding 服务（OpenAI、Cohere、Jina、Ollama 等）都有各自的 API 格式，适配器负责把它们翻译成统一的 `EmbeddingRequest` → `EmbeddingResponse` 流程。

**抽象方法：**
- `embed(self, request)`：执行向量化
- `get_model_info(self)`：返回模型信息（名称、维度、是否支持多模态等）

**`looks_like_multimodal_embedding_model(model_name)`（第 14-28 行）：**
- **业务做什么：** 根据模型名称判断是否支持多模态（图文混合）输入
- **大白话解释：** 看模型名字猜它能不能处理图片——名字里带 "vl-embedding"、"vision-embedding" 的基本都能

---

## 3. 适配器注册表（adapters/__init__.py）

*文件：`deeptutor/services/embedding/adapters/__init__.py`*

### 3.1 `ADAPTER_BACKENDS`（适配器后端注册表）

*文件位置：adapters/__init__.py，第 16-23 行*

```python
ADAPTER_BACKENDS: dict[str, type[BaseEmbeddingAdapter]] = {
    "openai_compat":    OpenAICompatibleEmbeddingAdapter,   # OpenAI 兼容格式
    "openai_sdk":       OpenAISDKEmbeddingAdapter,         # OpenAI 官方 SDK
    "cohere":           CohereEmbeddingAdapter,            # Cohere
    "jina":             JinaEmbeddingAdapter,              # Jina AI
    "ollama":           OllamaEmbeddingAdapter,             # 本地 Ollama
    "dashscope_native": DashScopeMultiModalEmbeddingAdapter, # 阿里云 DashScope
}
```

**业务含义：** 这是一个"适配器黄页"——系统根据配置中的 `binding` 字段（如 `"openai_compat"`），从这个字典里找到对应的适配器类。

**大白话解释：** 就像快递公司的"配送网点"——寄到不同地区用不同的网点，这里根据不同 Embedding 服务选不同的适配器。

---

## 4. 配置管理（config.py）

*文件：`deeptutor/services/embedding/config.py`*

### 4.1 `EmbeddingConfig`（Embedding 配置）

*文件位置：config.py，第 10-27 行*

```python
@dataclass
class EmbeddingConfig:
    model: str                  # 模型名称
    api_key: str                # API 密钥
    base_url: str | None        # API 基础 URL
    effective_url: str | None   # 实际使用的 URL
    binding: str = "openai"     # 适配器绑定
    provider_name: str = "openai"  # 提供商名称
    dim: int = 0                # 向量维度
    send_dimensions: bool | None = None  # 是否发送维度参数
    request_timeout: int = 60   # 请求超时（秒）
    batch_size: int = 10        # 每批处理多少条文本
    batch_delay: float = 0.0    # 批次间延迟（秒）
```

**业务含义：** EmbeddingConfig 集中管理所有 Embedding 相关的配置参数。

**关键字段说明：**
- `batch_size`：每批处理多少条文本。不是越大越好——某些提供商有上限（如 SiliconFlow Qwen3 系列最多 32 条，DashScope 最多 20 条）
- `batch_delay`：批次间延迟。如果调用太频繁，提供商会限流（rate limit），加延迟可以避免被"封"
- `dim`：向量维度。0 表示用模型的默认维度
- `send_dimensions`：是否显式告诉 API 要多少维的向量。有些模型支持可变维度（如 OpenAI text-embedding-3），有些不支持

**`get_embedding_config()`（第 30-62 行）：**
- **业务做什么：** 从系统配置中加载 Embedding 设置
- **关键逻辑：** 会验证模型是否已设置、URL 是否有效、API 密钥是否配置（非本地模式）
- **大白话解释：** 去"设置中心"读取 Embedding 的配置，检查配置是否完整

---

## 5. Embedding 客户端（client.py）

*文件：`deeptutor/services/embedding/client.py`*

### 5.1 `EmbeddingClient`（Embedding 客户端）

*文件位置：client.py，第 34-243 行*

```python
class EmbeddingClient:
    def __init__(self, config: Optional[EmbeddingConfig] = None):
        ...
```

**业务含义：** EmbeddingClient 是 Embedding 服务的"前台"。所有需要向量化的地方都通过它来调用。它负责选择合适的适配器、管理批量处理、验证结果质量。

**`__init__(self, config)`（第 37-64 行）：**
- **业务做什么：** 初始化客户端，创建适配器实例
- **关键逻辑：**
  1. 加载配置
  2. 验证端点 URL 是否有效
  3. 根据配置中的 `binding` 找到对应的适配器类
  4. 实例化适配器，传入 API 密钥、URL、模型名等配置
- **大白话解释：** 翻译官上岗——先检查翻译工具是否正常，再确认用哪个翻译服务

**`embed(self, texts, progress_callback)`（第 66-152 行）：**
- **业务做什么：** 把一批文本转成向量（核心方法）
- **输入：** `texts`（文本列表）、`progress_callback`（进度回调函数）
- **输出：** `List[List[float]]`（向量列表，每个向量是一串浮点数）

**执行流程（大白话解释）：**

1. **空列表检查（第 67-68 行）：** 如果文本列表为空，直接返回空列表
2. **批次大小限制（第 76-83 行）：** 从提供商配置中读取最大批次限制，如果设置的批次大小超过了限制，就自动缩小。例如你设置每批 100 条，但 SiliconFlow 最多 32 条，系统会自动调整为 32 条
3. **分批处理（第 89-152 行）：**
   - 把文本列表切成多个批次
   - 对每个批次，调用适配器的 `embed()` 方法
   - 验证返回的向量质量（数量是否正确、维度是否一致、值是否合法）
   - 如果启用了进度回调，每处理完一批就汇报进度
   - 批次间加上延迟（如果配置了），防止被限流
4. **维度一致性检查（第 123-133 行）：** 如果不同批次返回的向量维度不一致，说明提供商有问题，直接报错
5. **错误处理（第 98-113 行）：** 如果某批处理失败，记录详细的错误信息（批次索引、文本长度等），方便排查

**`embed_sync(self, texts)`（第 224-236 行）：**
- **业务做什么：** 同步版本的 embed（不需要 async/await）
- **大白话解释：** 给不想用异步代码的调用方提供的简化版

**`get_embedding_func(self)`（第 238-242 行）：**
- **业务做什么：** 返回一个可以直接调用的 Embedding 函数（用于传递给 LlamaIndex 等框架）
- **大白话解释：** 把翻译官包装成一个简单的函数，方便传给其他工具使用

**`supports_multimodal_contents(self)`（第 154-164 行）：**
- **业务做什么：** 检查当前配置的模型是否支持多模态（图文混合）输入
- **大白话解释：** 问翻译官"你能处理图片吗？"

**`embed_contents(self, contents, progress_callback)`（第 166-222 行）：**
- **业务做什么：** 把多模态内容（文本+图片）转成向量
- **输入：** `contents`（如 `[{"text": "..."}, {"image": "data:..."}]`）
- **关键逻辑：** 先检查模型是否支持多模态，不支持就报错
- **大白话解释：** 给翻译官图文混合的材料，让他一起翻译成向量

**`_resolve_adapter_class(binding)`（第 18-31 行）：**
- **业务做什么：** 根据绑定名称找到对应的适配器类
- **大白话解释：** 根据配置中的"用哪个服务"，找到对应的翻译官

**`get_embedding_client(config)`（第 248-253 行）：**
- **业务做什么：** 获取全局单例的 EmbeddingClient
- **大白话解释：** 获取唯一的翻译官实例

---

## 6. 向量验证（validation.py）

*文件：`deeptutor/services/embedding/validation.py`*

### 6.1 `validate_embedding_batch()`（批量向量验证）

*文件位置：validation.py，第 37-128 行*

```python
def validate_embedding_batch(
    embeddings: Any,
    *,
    expected_count: int,       # 期望的向量数量
    binding: str | None = None,  # 提供商名称
    model: str | None = None,    # 模型名称
    batch_index: int | None = None,  # 批次索引
    total_batches: int | None = None,  # 总批次数
    start_index: int = 0,       # 起始索引
) -> list[list[float]]:
```

**业务含义：** 这是 Embedding 服务的"质检员"。它逐条检查提供商返回的向量是否符合要求，确保后续的相似度计算不会因为"脏数据"而崩溃。

**检查项目（逐条验证）：**

1. **类型检查（第 61-70 行）：** 返回的是不是列表？不能是字符串或空值
2. **数量检查（第 72-79 行）：** 返回的向量数量是否和输入文本数量一致？少了说明提供商"偷工减料"了
3. **空值检查（第 84-85 行）：** 每个向量不能是 null
4. **元素类型检查（第 86-90 行）：** 每个向量必须是数字序列，不能是字符串
5. **空向量检查（第 92-93 行）：** 向量不能为空（长度为 0）
6. **单维度检查（第 96-116 行）：** 逐个检查每个维度的值：
   - 不能是 `None`
   - 不能是布尔值（必须是真正的数字）
   - 必须是有限值（不能是 `NaN`、`Infinity`）
7. **维度一致性检查（第 120-127 行）：** 同一批次中所有向量的维度必须一致

**大白话解释：** 翻译官交回来的密码本，质检员要一页一页检查——密码位数对不对？有没有缺页？有没有乱码？

---

## 7. 适配器详解

DeepTutor 支持 6 种 Embedding 适配器，覆盖了主流的 Embedding 服务：

### 7.1 `OpenAICompatibleEmbeddingAdapter`（OpenAI 兼容）

*文件：`adapters/openai_compatible.py`*

**业务含义：** 支持所有兼容 OpenAI Embedding API 格式的服务。这是最通用的适配器，适用于 OpenAI、Azure OpenAI、SiliconFlow、DeepSeek 等。

**大白话解释：** 这是"通用翻译官"——只要你的服务说的是 OpenAI 的"语言"，就能用这个适配器。

### 7.2 `OpenAISDKEmbeddingAdapter`（OpenAI 官方 SDK）

*文件：`adapters/openai_sdk.py`*

**业务含义：** 使用 OpenAI 官方 Python SDK（`AsyncOpenAI`）调用 Embedding API。这是为旧配置和测试保留的适配器，新配置推荐使用 `openai_compat`。

**关键方法：**
- `_should_send_dimensions(model_name)`（第 30-47 行）：判断是否应该向 API 发送维度参数。`text-embedding-3` 系列和 `qwen3-embedding` 系列支持可变维度
- `_build_client()`（第 49-63 行）：构建 AsyncOpenAI 客户端
- `embed()`（第 65-143 行）：调用 `client.embeddings.create()` 并处理各种错误

### 7.3 `CohereEmbeddingAdapter`（Cohere）

*文件：`adapters/cohere.py`*

**业务含义：** 支持 Cohere 的 Embedding API。Cohere 的特殊之处在于它的 `input_type` 参数——你需要告诉它这段文本是"搜索文档"还是"搜索查询"，它会据此调整向量。

### 7.4 `JinaEmbeddingAdapter`（Jina AI）

*文件：`adapters/jina.py`*

**业务含义：** 支持 Jina AI 的 Embedding API。Jina 的特色是支持 `late_chunking`（延迟分块）——可以先对长文本分段，最后再统一向量化，保持语义连贯性。

### 7.5 `OllamaEmbeddingAdapter`（本地 Ollama）

*文件：`adapters/ollama.py`*

**业务含义：** 支持本地运行 Ollama 提供的 Embedding 模型。完全离线，不需要 API 密钥，适合隐私敏感场景。

### 7.6 `DashScopeMultiModalEmbeddingAdapter`（阿里云 DashScope）

*文件：`adapters/dashscope_native.py`*

**业务含义：** 支持阿里云 DashScope 的多模态 Embedding 模型。这是唯一支持图文混合向量化的适配器，可以对图片和文本一起生成向量。

---

## 8. 数据流总览

```
RAG 索引 / 知识库检索
        │
        ▼
   EmbeddingClient.embed(texts=["什么是微积分", "微积分的应用"])
        │
        ├─ 加载配置 (EmbeddingConfig)
        ├─ 根据 binding 选择适配器
        │     ├─ openai_compat → OpenAICompatibleEmbeddingAdapter
        │     ├─ cohere → CohereEmbeddingAdapter
        │     └─ ...
        │
        ├─ 计算批次大小（clamp 到提供商限制）
        │
        ├─ 分批处理 ──────────────────────────┐
        │  批次 1: ["什么是微积分"]            │
        │  │   ├─ adapter.embed(request)       │
        │  │   │   └─ HTTP POST → 提供商 API   │
        │  │   ├─ validate_embedding_batch()   │
        │  │   │   ├─ 检查数量是否正确          │
        │  │   │   ├─ 检查维度是否一致          │
        │  │   │   └─ 检查值是否合法            │
        │  │   └─ 报告进度 (progress_callback) │
        │  │                                   │
        │  批次 2: ["微积分的应用"]             │
        │  │   └─ ... (同上)                   │
        │  │                                   │
        │  └─ 批次间延迟 (batch_delay)         │
        │                                       │
        ▼
   返回: [[0.123, -0.456, ...], [0.789, -0.012, ...]]
        │
        └─ 用于相似度计算 / 向量检索
```

---

## 9. 一句话总结

Embedding 服务是 DeepTutor 的"数学翻译官"——它把人类语言翻译成计算机能计算的数字向量，让 AI 可以"理解"文本之间的相似关系，是实现智能检索和知识库问答的基础设施。

