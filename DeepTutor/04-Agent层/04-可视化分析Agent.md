# 04 — 可视化分析 Agent（Visualize Pipeline）

> **源码位置**：`deeptutor/agents/visualize/`
> **核心文件**：`pipeline.py`（91行）、`capability.py`（574行）、`models.py`（89行）、`utils.py`（190行）
> **子 Agent**：`agents/analysis_agent.py`（103行）、`agents/code_generator_agent.py`（119行）、`agents/review_agent.py`（75行）
> **一句话总结**：可视化分析 Agent 是 DeepTutor 的"画图工具"——根据用户描述，自动判断该画什么类型的图，然后生成对应的代码（SVG、Chart.js、Mermaid、HTML），最后校验并修复。

---

## 目录

1. [整体架构](#整体架构)
2. [核心数据模型](#核心数据模型)
3. [VisualizeCapability — 能力入口](#visualizecapability--能力入口)
4. [VisualizePipeline — 三阶段编排](#visualizepipeline--三阶段编排)
5. [AnalysisAgent — 分析阶段](#analysisagent--分析阶段)
6. [CodeGeneratorAgent — 代码生成阶段](#codegeneratoragent--代码生成阶段)
7. [ReviewAgent — 修复阶段](#reviewagent--修复阶段)
8. [本地校验器 — validate_visualization](#本地校验器--validate_visualization)
9. [Manim 分支](#manim-分支)
10. [总结](#总结)

---

## 整体架构

如果把可视化 Agent 比作一家"设计工作室"，架构如下：

| 文件 | 角色比喻 | 职责 |
|------|----------|------|
| `capability.py` | 工作室前台 | 接收客户需求，调度整体流程，区分文本渲染和 Manim 动画两条路径 |
| `pipeline.py` | 项目经理 | 串联三个阶段：分析 → 代码生成 → 修复 |
| `models.py` | 设计规范文档 | 定义输入/输出的数据结构（分析结果、修复结果） |
| `utils.py` | 质检工具 | 本地校验代码是否合法，无需调用 AI |
| `agents/analysis_agent.py` | 需求分析师 | 理解用户意图，决定画什么类型的图 |
| `agents/code_generator_agent.py` | 画图师傅 | 根据分析结果生成代码 |
| `agents/review_agent.py` | 审图员 | 代码有问题时进行修复 |

**支持的 6 种渲染类型**：

| 渲染类型 | 产出物 | 应用场景 |
|----------|--------|----------|
| `svg` | 原始 SVG 代码 | 矢量示意图、流程图、结构图 |
| `chartjs` | Chart.js 配置 JSON | 柱状图、折线图、饼图等数据图表 |
| `mermaid` | Mermaid 语法文本 | 流程图、时序图、思维导图、类图 |
| `html` | 自包含 HTML 页面 | 交互式可视化、动画演示 |
| `manim_video` | Manim 动画视频 | 数学概念动画演示 |
| `manim_image` | Manim 静态图片 | 数学公式/图形渲染 |

---

## 核心数据模型

**源码位置**：`models.py`

### VisualizationAnalysis（可视化分析结果）

**源码位置**：第10-73行

```python
class VisualizationAnalysis(BaseModel):
    render_type: Literal["svg", "chartjs", "mermaid", "html", "manim_video", "manim_image"]
    description: str = ""           # 高层描述：这个图要展示什么
    data_description: str = ""      # 数据描述：有哪些数据/元素
    chart_type: str = ""            # 子类型：bar/line/flowchart/mindmap 等
    visual_elements: list[str] = [] # 视觉元素清单：颜色、形状、标签等
    rationale: str = ""             # 选择理由：为什么选这个渲染类型
    visual_genre: Literal[...] = "" # 教学风格标签：flowchart/structural/illustrative/chart/stepper 等
```

**大白话解释**：这是"需求分析师"提交的设计方案。它不仅说了"画什么类型的图"，还说了"为什么要画这种图"、"图上应该有哪些元素"、"教学风格是什么"。这个分析结果会原封不动地传给代码生成器。

**visual_genre 教学风格标签**（第52-73行）：

| 标签 | 含义 | 典型场景 |
|------|------|----------|
| `flowchart` | 流程图 | 展示步骤/流程 |
| `structural` | 结构图 | 展示概念关系/层级 |
| `illustrative` | 插图 | "X 是如何工作的"空间比喻 |
| `chart` | 图表 | 定量数据可视化 |
| `stepper` | 分步演示 | 循环或阶段性演示 |
| `interactive` | 交互式 | 可操作的 HTML 体验 |
| `mockup` | 模型图 | UI/界面展示 |
| `art` | 艺术图 | 创意表达 |

### ReviewResult（修复结果）

**源码位置**：第76-89行

```python
class ReviewResult(BaseModel):
    optimized_code: str    # 修复后的代码
    changed: bool = False  # 是否做了修改
    review_notes: str = "" # 修复说明
```

---

## VisualizeCapability — 能力入口

**源码位置**：`capability.py` 第50-573行

### 类定义

```python
class VisualizeCapability(BaseCapability):
    manifest = CapabilityManifest(
        name="visualize",
        description="Generate SVG, Chart.js, Mermaid, interactive HTML, or Manim animation/storyboard visualizations.",
        stages=["analyzing", "generating", "reviewing", "concept_analysis",
                "concept_design", "code_generation", "code_retry",
                "summary", "render_output"],
        tools_used=[],  # 可视化不需要外部工具
        cli_aliases=["visualize", "viz"],
    )
```

**基本信息**：
- **能力名称**：`visualize`
- **命令行别名**：`visualize` 或 `viz`
- **可用工具**：无（可视化是纯代码生成，不需要搜索/知识库等工具）
- **阶段**：共 9 个阶段，但一次请求只用到其中一部分

### run() — 核心调度

**源码位置**：第63-276行

**流程**：

```
用户输入："画一个展示牛顿第二定律的示意图"
         │
         ▼
┌─────────────────────────────────────┐
│  Stage 1: 分析 (analyzing)          │
│  AnalysisAgent 判断渲染类型          │
│  输出：VisualizationAnalysis         │
└──────────────┬──────────────────────┘
               │
               ├─ render_type 是 manim_video / manim_image？
               │   yes → 走 Manim 子管线（_run_manim_path）→ 结束
               │
               └─ no（svg/chartjs/mermaid/html）
                   │
                   ▼
┌─────────────────────────────────────┐
│  Stage 2: 生成 (generating)         │
│  CodeGeneratorAgent 生成代码         │
│  输出：代码字符串                     │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  Stage 3: 校验/修复 (reviewing)      │
│  先本地校验 → 通过？直接使用          │
│  → 不通过？ReviewAgent 修复 → 再校验  │
│  输出：最终代码                       │
└──────────────┬──────────────────────┘
               │
               ▼
        推送结果给前端
```

**关键设计特点**：

1. **先本地校验，再 AI 修复**（第146-169行）：不浪费 AI 调用。先用 `validate_visualization` 做零成本的本地检查，只有不通过时才调用 ReviewAgent。

2. **HTML 特殊处理**（第170-190行）：HTML 文档通常 8-16k tokens，太大不适合走修复回路。如果 HTML 校验失败，直接使用一个兜底 HTML 模板（`build_fallback_html`），而不是让 AI 修复。

3. **结果信封**（第262-276行）：最终推送一个结构化结果，包含 `render_type`（前端用这个决定用哪个渲染器）、`code`（代码内容和语言）、`analysis`（分析结果）、`review`（修复结果）。

---

## VisualizePipeline — 三阶段编排

**源码位置**：`pipeline.py` 第13-91行

这是一个轻量级的编排器，职责很简单：把三个子 Agent 串起来。

### __init__()

**源码位置**：第14-41行

初始化时创建三个子 Agent：
- `AnalysisAgent`：分析需求，决定渲染类型
- `CodeGeneratorAgent`：根据分析结果生成代码
- `ReviewAgent`：修复有问题的代码

每个 Agent 都共享相同的 API 配置和语言设置。

### 三个运行方法

| 方法 | 对应阶段 | 输入 | 输出 |
|------|----------|------|------|
| `run_analysis()` | 分析 | 用户输入、历史上下文、渲染模式、附件 | `VisualizationAnalysis` |
| `run_code_generation()` | 代码生成 | 用户输入、历史上下文、分析结果 | `str`（代码） |
| `run_repair()` | 修复 | 用户输入、分析结果、原代码、错误信息 | `ReviewResult` |

---

## AnalysisAgent — 分析阶段

**源码位置**：`agents/analysis_agent.py` 第13-103行

### 业务场景

用户说"帮我画一个柱状图对比三国 GDP"——AnalysisAgent 的工作是判断：这个需求应该用 Chart.js 的 bar 图来实现。

### process() 方法

**源码位置**：第30-103行

**输入**：
- `user_input`：用户说的话
- `history_context`：对话历史
- `render_mode`：渲染模式偏好
- `attachments`：用户上传的附件

**输出**：`VisualizationAnalysis` 对象

**渲染模式的处理逻辑**：

| render_mode | 含义 | 处理方式 |
|-------------|------|----------|
| `manim_video` / `manim_image` | 用户明确要求 Manim | 直接返回 stub 分析结果，跳过 LLM 调用 |
| `svg` / `chartjs` / `mermaid` / `html` | 用户指定了渲染类型 | 使用 `system_fixed` 提示词，告诉 AI 类型已定 |
| `figure` | 图书插图模式 | 使用 `system_figure` 提示词，AI 只能从 svg/chartjs/mermaid 中选（排除 html） |
| `auto`（默认） | 自动判断 | 使用通用提示词，AI 从全部 6 种类型中选择 |

**大白话解释**：
- `render_mode` 就像是用户对设计师说"我要一幅油画"（指定类型）或"你看着办"（自动判断）。
- 当用户明确指定了类型（如 `chartjs`），系统会跳过 AI 的判断过程，直接告诉 AI "用 Chart.js 格式画"。
- `figure` 模式是给图书模块用的，它排除了 HTML（因为图书里不能嵌入交互式 HTML），AI 只能从静态图类型中选。

**防御性编程**（第96-102行）：如果 AI 在 `figure` 模式下选了不在允许范围内的类型，强制改为 `svg`。

---

## CodeGeneratorAgent — 代码生成阶段

**源码位置**：`agents/code_generator_agent.py` 第14-119行

### 业务场景

AnalysisAgent 已经决定了"画一个 Chart.js 的柱状图，数据是三国 GDP 对比"。CodeGeneratorAgent 的工作是：写出具体的 Chart.js 配置代码。

### process() 方法

**源码位置**：第31-119行

**输入**：
- `user_input`：用户原始输入
- `history_context`：对话历史
- `analysis`：VisualizationAnalysis 分析结果

**输出**：`str`（代码字符串）

**提示词组装策略**（第44-49行）：

```
system_prompt = system_base（通用基础规则）
              + rules_general（通用代码规则）
              + rules_{render_type}（类型专属规则，如 rules_svg / rules_chartjs / rules_mermaid / rules_html）
```

**大白话解释**：提示词采用"基础 + 通用规则 + 类型专属规则"的分层结构。这样每次调用只加载该类型需要的规则，不会把四种类型的规则全塞进 prompt，节省 token。

**代码提取逻辑**（第78-119行）：

1. 尝试从 AI 输出中提取代码块（```svg ... ```）
2. 如果提取不到，检查是否是裸 HTML 文档（以 `<!doctype` 或 `<html` 开头）
3. 对 SVG 做额外清理：裁剪到 `<svg>...</svg>` 范围
4. 对 HTML 做额外清理：裁剪到 `<!doctype...` 到 `</html>` 范围

**为什么需要这些清理？** AI 有时会在代码块外面加一些"解说文字"（如"好的，这是您要的 SVG：……"），或者把代码块边界写得不标准。这些清理步骤确保传给前端的是纯净的代码。

---

## ReviewAgent — 修复阶段

**源码位置**：`agents/review_agent.py` 第14-75行

### 业务场景

CodeGeneratorAgent 生成的 SVG 代码可能缺少 `viewBox` 属性，或者 Chart.js 配置不是合法的 JSON。ReviewAgent 接收具体的错误信息，进行针对性修复。

### process() 方法

**源码位置**：第31-75行

**输入**：
- `user_input`：用户原始输入
- `analysis`：分析结果
- `code`：有问题的代码
- `error`：具体的错误描述（来自本地校验器）

**输出**：`ReviewResult` 对象

**关键设计**：
- ReviewAgent **只在本地校验失败后才被调用**，不是无条件执行的
- 它接收的是**具体错误**（如"SVG must contain a root `<svg>` element"），而不是泛泛地"检查代码质量"
- 输出格式是 JSON Object，包含修复后的代码和修复说明
- 修复深度只有 1 次（不是循环修复），如果修复后还是有问题，就标记但不阻塞

---

## 本地校验器 — validate_visualization

**源码位置**：`utils.py` 第111-181行

这是整个可视化流程的"质量关卡"。它不调用 AI，完全在本地运行，零成本。

### 函数签名

```python
def validate_visualization(code: str, render_type: str) -> tuple[bool, str]:
```

**输入**：代码字符串 + 渲染类型

**输出**：`(是否通过, 错误信息)` 元组。通过时错误信息为空字符串。

### 各类型校验规则

| 渲染类型 | 校验内容 | 大白话解释 |
|----------|----------|------------|
| 通用 | 代码不能为空 | 最起码要有内容 |
| `svg` | 1. 包含 `<svg` 标签<br>2. 是合法的 XML<br>3. 根元素是 `<svg>`<br>4. 必须有 `viewBox` 属性（注意大小写！） | SVG 必须有正确的根标签和 viewBox，否则浏览器无法正确缩放 |
| `chartjs` | 1. 是严格的 JSON（双引号、不能有注释）<br>2. 是一个 JSON 对象<br>3. 包含 `type` 和 `data` 字段 | Chart.js 配置必须是合法的 JSON，JS 注释和单引号都不行 |
| `mermaid` | 第一行必须以合法的 Mermaid 关键字开头 | Mermaid 语法规定第一行必须是 `graph`、`flowchart`、`sequenceDiagram` 等关键字 |
| `html` | 包含 `<html`、`<!doctype`、`<body` 或 `<div` 标签 | 至少看起来像一个 HTML 文档 |

**SVG viewBox 的大小写陷阱**（第137-141行）：

这是代码注释里特别强调的一个细节：SVG 标准规定 `viewBox` 必须是**驼峰命名**（camelCase）。如果 AI 不小心写成了小写的 `viewbox`，浏览器会忽略这个属性，导致图形塌缩成一团。本地校验器会专门检查这个。

### 兜底 HTML 模板

**源码位置**：`utils.py` 第59-101行

当 HTML 生成失败时，系统不会给用户一个空白页面，而是渲染一个美观的兜底卡片：

```html
一个带有渐变背景、白色卡片、蓝色标题的页面，显示标题和描述信息。
如果 AI 提供了错误说明，还会显示一个黄色警告条。
```

---

## Manim 分支

**源码位置**：`capability.py` 第278-492行

### 业务场景

当 AnalysisAgent 判断需要画数学动画（如"展示傅里叶变换的过程"）时，渲染类型会被设为 `manim_video` 或 `manim_image`。这时走的是完全不同的管线。

### _run_manim_path() 方法

**流程**：

```
concept_analysis（概念分析）
    → concept_design（概念设计）
    → code_generation（Manim 代码生成）
    → code_retry（渲染 + 自动重试）
    → summary（生成总结文本）
    → render_output（输出渲染产物）
```

**关键特点**：

1. **依赖检查**（第297-302行）：如果 Manim 库没有安装，直接抛出友好的错误提示。
2. **自动重试**（第376-407行）：Manim 渲染可能因为代码错误而失败，系统会自动重试，每次重试都会把错误信息反馈给 AI 让其修复代码。
3. **进度回调**（第389-406行）：渲染过程通过 `on_render_progress` 和 `on_retry` 回调向前端推送进度。
4. **产物输出**：最终输出包含视频文件或图片文件，以及源代码。

---

## 总结

### 数据流全景图

```
用户输入："画一个牛顿第二定律的示意图"
         │
         ▼
VisualizeCapability.run()
         │
         ▼
AnalysisAgent.process()
  → AI 判断：这是示意图，用 SVG 画
  → 输出：VisualizationAnalysis(render_type="svg", ...)
         │
         ▼
CodeGeneratorAgent.process()
  → AI 生成 SVG 代码
  → 输出：<svg viewBox="...">...</svg>
         │
         ▼
validate_visualization(code, "svg")
  → 检查：有 <svg> 标签？是合法 XML？有 viewBox？
  ├─ 通过 → 直接使用 ✓
  └─ 不通过 → ReviewAgent.process(code, error)
       → AI 修复代码
       → validate_visualization 再检查
         │
         ▼
推送结果给前端：
  { render_type: "svg", code: { language: "svg", content: "..." }, ... }
```

### 关键设计决策

| 决策 | 原因 |
|------|------|
| 分析→生成→校验三阶段 | 分离关注点：先判断类型，再写代码，最后检查质量 |
| 先本地校验，再 AI 修复 | 节省 AI 调用成本。大多数情况代码是合法的，不必调用 ReviewAgent |
| 按类型分层提示词 | 每次只加载一个类型的规则，节省 token，提高针对性 |
| HTML 兜底模板 | HTML 文档太长不适合 AI 修复，但用户不能看到空白页 |
| Manim 独立分支 | Manim 是完整的子管线，有自己的分析→设计→生成→渲染→总结流程 |
| SVG 大小写敏感检查 | 一个字母的大小写错误可能导致整个图无法渲染，这是实际踩过的坑 |
| 防御性代码清理 | AI 输出常有"多余的话"，需要裁剪到纯代码 |
