# 03-06 · SquillaRouter 路由层 — 深度剖析

> **文件数**：34 个文件 | **核心类**：SquillaRouter (predictor.py:257) | **技术**：LightGBM 机器学习模型 | **学术论文**：arxiv.org/abs/2607.11399 | **codegraph 符号**：71 个

---

## 定位

SquillaRouter 是 OpenSquilla 的**智能调度员**。每次用户发消息，路由器先判断任务难度，然后决定用哪个模型——简单问题用便宜模型，复杂问题才用贵模型。这能大幅降低 AI 使用成本。

核心原理：本地运行一个 LightGBM 分类模型，每次推理只需几毫秒，不消耗网络请求。

---

## 生活化比喻

> SquillaRouter 像**滴滴的智能派单系统**。你去楼下便利店（简单任务），系统派一辆共享单车（便宜模型）；你要去机场（复杂任务），系统派一辆专车（贵模型）。系统通过分析你的历史行程（历史对话）、当前时间地点（上下文特征）、目的地（任务复杂度），自动做出最优决策。

---

## 一、机器学习模型架构

### 1.1 模型类型：LightGBM 梯度提升树

SquillaRouter 使用 LightGBM 作为分类器，将每个用户请求分类为 4 个难度层级之一。模型文件：lgbm_model.bin（predictor.py:286）

### 1.2 模型版本管理

路由模型通过 model_dir/version.json 声明版本。当前支持 v4 版本，SquillaRouter 工厂类自动检测版本并分发到 V4Router（predictor.py:264-273）：

```
SquillaRouter.__new__()
  ↓ 读取 version.json
  ↓ version == "v4" → V4Router(model_dir)
  ↓ 否则 → 默认 SquillaRouter 初始化
```

---

## 二、特征工程：5 通道特征提取

FeatureExtractor (features.py) 从用户消息中提取 5 个通道的特征，拼接成最终特征向量：

### 通道 1：手工特征 (HANDCRAFTED_DIMS = 51 维)

| 特征类别 | 示例特征 | 维度 |
|----------|---------|------|
| 文本统计 | 字数、句子数、平均句长、问号数 | ~10 |
| 代码特征 | 是否含代码块、代码语言类型、代码行数 | ~10 |
| 关键词匹配 | 搜索/分析/写/实现/修复等关键词计数 | ~15 |
| 复杂度指标 | 大写字母比例、特殊字符密度、URL 数量 | ~10 |
| 格式特征 | 列表项数、标题层级、引用块数 | ~6 |

### 通道 2：上下文特征 (CONTEXT_DIMS = 10 维)

ContextMetadata (features.py:164) 从会话上下文中提取：

| 字段 | 说明 |
|------|------|
| turn_index | 当前是第几轮对话 |
| context_tokens_est | 估算上下文 token 数 |
| n_tools | 可用工具数量 |
| tool_result_length | 上次工具结果长度 |
| has_code_block | 历史是否含代码块 |
| has_file_reference | 是否引用了文件 |
| has_url | 是否包含 URL |
| has_tool_results | 是否包含工具结果 |

### 通道 3：历史特征 (HIST_DIMS = 16 维)

extract_hist_features() (features.py:108) 从历史对话中提取 16 维向量：

```
Layout:
  [0] prev_route_idx      # 上一轮的难度层级
  [1] prev_difficulty      # 上一轮的难度分数
  [2] prev_margin          # 上一轮的分类置信度边距
  [3] max_route_idx        # 历史最高难度层级
  [4] turn_index           # 当前轮次
  [5] history_len          # 历史长度
  [6] dominant_route       # 历史主导层级（出现次数最多的层级）
  [7] switches             # 层级切换次数
  [8..15] trajectory one-hot  # 8 种对话轨迹类型的 one-hot 编码
```

### 通道 4：TF-IDF 特征 (降维后)

对文本做 TF-IDF 向量化，再通过 SVD 降维（_TFIDF_SVD_DIMS 维度），捕捉文本的主题分布。

### 通道 5：BGE 嵌入特征 (降维后)

使用 BGE 模型（BAAI General Embedding）将文本编码为语义向量，再通过 PCA 降维（_BGE_PCA_DIMS 维度），捕捉深层语义。

---

## 三、推理流程：predict() 方法（8 步）

predict() 方法 (predictor.py:292-332) 是路由器的核心推理入口：

**步骤 1：轨迹分类** — classify_trajectory(history) 分析历史对话模式，输出 8 种轨迹类型之一（NEW_TOPIC/CONTINUATION/REFINEMENT/DEEPENING/...）

**步骤 2：特征提取** — self._extractor.transform(text, context, history, trajectory) 提取 5 通道特征，拼接为 1 维向量

**步骤 3：维度校验** — 检查特征维度是否与模型期望匹配（expected_dims vs actual_dims），不匹配则抛出 ValueError

**步骤 4：LightGBM 推理** — self._model.predict(features) 输出 4 分类原始概率（raw_probs）

**步骤 5：概率标准化** — 将 raw_probs 映射为 R0/R1/R2/R3 的概率字典（probs）

**步骤 6：置信度计算** — 最高概率与次高概率之差（margin），判断分类是否可靠

**步骤 7：难度分数** — difficulty_score = sum(i * p_i)，加权平均量化任务难度

**步骤 8：后处理** — apply_post_processing(raw_probs, text, config, context, history) 应用规则后处理，输出最终层级和标志位

---

## 四、CascadeRouter：两级级联路由

CascadeRouter (predictor.py:339) 是更复杂的路由策略，使用 3 个 LightGBM 模型进行两级分类：

```
输入文本
  ↓
Stage 1: 轻量/重量分类 (cascade_stage1.bin)
  ├── 轻量 (s1_prob <= 0.4)
  │     ↓
  │   Stage 2a: R0 vs R1 分类 (cascade_stage2a.bin)
  │     ├── s2a_prob > 0.5 → R1
  │     └── s2a_prob <= 0.5 → R0
  │
  └── 重量 (s1_prob > 0.4)
        ↓
      Stage 2b: R2 vs R3 分类 (cascade_stage2b.bin)
        ├── s2b_prob > 0.5 → R3
        └── s2b_prob <= 0.5 → R2
```

级联优势：Stage 1 阈值偏向"重量"（0.4），避免简单任务被错误路由到弱模型。

---

## 五、4 级难度分类映射

| 层级 | 难度 | 路由决策 | 适用任务 | 默认模型 |
|------|------|---------|---------|---------|
| R0 (C0) | 简单 | 最便宜模型 | 闲聊、简单问答、格式转换 | deepseek-v4-flash |
| R1 (C1) | 中等 | 中档模型 | 代码编写、文档生成、数据分析 | deepseek-v4-pro |
| R2 (C2) | 困难 | 高档模型 | 复杂算法、架构设计、多文件工程 | kimi-k2.7-code |
| R3 (C3) | 极难 | 最强模型 | 需要最强推理能力的任务 | glm-5.2 |

---

## 六、RoutingResult 输出结构

每次推理返回 RoutingResult 对象 (predictor.py:324-332)：

| 字段 | 类型 | 说明 |
|------|------|------|
| route_class | str | 最终路由层级 (R0/R1/R2/R3) |
| probabilities | dict | 4 分类概率 |
| difficulty_score | float | 难度分数 (0-3) |
| margin | float | 置信度边距 |
| flags | dict | 后处理标志位 |
| tier | str | 模型层级 ID |
| thinking_mode | str | 思考模式 (auto/on/off) |
| prompt_policy | str | 提示词策略 |
| prompt_hint | str | 提示词提示 |
| selected_model | str | 选中的模型名称 |
| trajectory | str | 对话轨迹类型 |
| model_version | str | 模型版本 |

---

## 七、模型训练管线

1. 从真实代理流量中收集标注数据（JSONL 格式）
2. 每条样本包含：text + session_context + tool_context + route_class 标签
3. 特征提取 (FeatureExtractor.fit) 训练 TF-IDF 向量化器和 PCA 降维器
4. BGE 嵌入 (BGE_PCA_DIMS) 训练 PCA 降维
5. LightGBM 训练 (lgb.train) 多分类模型
6. 模型导出为 lgbm_model.bin + features/ 目录
7. 验证集评估准确率和 margin 分布

> 来源：codegraph explore squilla_router, predictor.py L257-L332, L339-L407, features.py L91-L189, cascade config


---

## 七、predict() 后处理详解（5 个子函数）

predict() 在 LightGBM 输出 4 类概率后，通过 5 个后处理函数做最终决策：

### 7.1 _apply_flag_overrides(raw_probs, flags)
```
根据 flags 覆盖原始概率：
  - force_auto → 忽略路由，使用自动模式
  - force_tier:X → 强制使用指定层级
  - no_ensemble → 禁用 ensemble 模式
源码：predictor.py _apply_flag_overrides()
```

### 7.2 _derive_thinking_mode(config, tier)
```
根据层级和配置推导思考模式：
  - C0 (Flash)：无思考模式
  - C1 (Pro)：轻量思考
  - C2 (Code)：标准思考
  - C3 (GLM)：深度思考
源码：predictor.py _derive_thinking_mode()
```

### 7.3 _derive_prompt_policy(config, tier)
```
根据层级推导提示词策略：
  - 系统提示词模板选择
  - 工具定义压缩策略
  - 上下文窗口分配
源码：predictor.py _derive_prompt_policy()
```

### 7.4 _get_prompt_hint(tier, context)
```
根据层级和上下文生成提示词提示：
  - 低层级：简洁提示
  - 高层级：详细指令
源码：predictor.py _get_prompt_hint()
```

### 7.5 _select_model(tier, config)
```
根据层级和配置选择具体模型：
  - 读取 tier 配置中的 model 字段
  - 检查模型可用性
  - 返回模型 ID
源码：predictor.py _select_model()
```

> 来源：codegraph explore squilla_router — predict() calls 列表包含这 5 个函数

---

## 八、路由器控制工具 (router_control)

用户可以手动切换模型层级，通过 router_control 工具 (tools/builtin/router_control.py)：

| 操作 | 参数 | 说明 |
|------|------|------|
| set_hold | action="set_hold", target_id="tier:c2", evidence="..." | 临时切换到 C2 层级 |
| clear_hold | action="clear_hold" | 恢复自动路由 |

Hold 配置：
- DEFAULT_HOLD_TTL_SECONDS：默认保持时间
- DEFAULT_HOLD_TURNS：默认保持轮次

可用层级：tier:c0 (deepseek-v4-flash) / tier:c1 (deepseek-v4-pro) / tier:c2 (kimi-k2.7-code) / tier:c3 (glm-5.2)

> 来源：tools/builtin/router_control.py:16-52, router_control.py set_hold()
