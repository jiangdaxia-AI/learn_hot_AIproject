# Book 学习系统 — AI 驱动的互动教材

> Understand-Anything 知识图谱: 26 节点, 25 边, 7 层
## 一句话说明

Book 学习系统是 DeepTutor 的"AI 教材工厂 + 互动阅读器"，它把传统的静态教材 PDF 变成由 AI 自动生成、可交互、可答题、可追踪学习进度的智能学习材料。

---

## 业务价值

- **AI 自动生成教材**：输入学习意图，AI 自动规划大纲、撰写章节、插入互动元素，几分钟就能生成一本完整的互动教材。
- **互动式学习**：教材不只是"读"的，每个页面里嵌入了测验题、代码块、闪卡、时间线、交互图表等 Block，学生可以边学边练。
- **个性化深度探索**：学生在阅读时可以对任意知识点发起"深度探索"（Deep Dive），AI 会生成专门的扩展页面。
- **分页追踪**：每个学生的学习进度（读了哪些页面、答了多少题、做对了多少）都被记录，教师可以精确了解学生的掌握情况。
- **与 AI 对话联动**：每个页面都有一个专属的聊天面板，学生可以针对页面内容提问，AI 结合教材上下文回答。

---

## 源码位置

| 文件 | 路径 | 行数 |
|------|------|------|
| Book 主页 | `web/app/(workspace)/book/page.tsx` | 607 |
| 书架列表 | `web/app/(workspace)/book/components/BookLibrary.tsx` | ~200+ |
| 教材创建器 | `web/app/(workspace)/book/components/BookCreator.tsx` | ~400+ |
| 大纲编辑器 | `web/app/(workspace)/book/components/SpineEditor.tsx` | ~300+ |
| 侧边栏 | `web/app/(workspace)/book/components/BookSidebar.tsx` | ~280+ |
| 页面阅读器 | `web/app/(workspace)/book/components/PageReader.tsx` | ~500+ |
| Block 渲染器 | `web/app/(workspace)/book/components/blocks/BlockRenderer.tsx` | ~200+ |
| 聊天面板 | `web/app/(workspace)/book/components/BookChatPanel.tsx` | ~300+ |
| 进度时间线 | `web/app/(workspace)/book/components/BookProgressTimeline.tsx` | ~100+ |
| 健康状态横幅 | `web/app/(workspace)/book/components/BookHealthBanner.tsx` | ~100+ |
| 页面大纲导航 | `web/app/(workspace)/book/components/PageOutlineNav.tsx` | ~100+ |
| Book API 层 | `web/lib/book-api.ts` | ~200+ |
| Book 类型定义 | `web/lib/book-types.ts` | ~200+ |
| Book 进度管理 | `web/lib/book-progress.ts` | ~100+ |

---

## 教材生命周期的四个阶段

Book 系统采用**四阶段流程**，就像真实的教材编写过程：

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ ① 创建   │ → │ ② 大纲   │ → │ ③ 编译   │ → │ ④ 阅读   │
│ Creator  │   │  Spine   │   │  Pages   │   │  Reader  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
  输入意图       确认章节        逐页生成        互动学习
  生成提案       调整结构        AI 撰写        深度探索
```

### 阶段一：创建（Creator）

**对应 View**：`view === "creator"`

**业务场景**：教师或学生想创建一本新教材，输入学习目标，AI 生成教材提案。

**交互流程**：
1. **输入学习意图**：在创建器中输入一段描述，如"我想学习微积分基础，从极限到导数"。
2. **选择上下文**：可选地引用现有对话、知识库、笔记本、题库中的题目作为参考素材。
3. **AI 生成提案**：后端根据意图生成一个 `BookProposal`（教材提案），包含：
   - 教材标题
   - 目标受众
   - 学习目标
   - 建议的章节结构
   - 预估的页面数
4. **确认提案**：学生可以编辑提案（修改标题、调整章节等），确认后进入大纲阶段。

**源码路径**：`web/app/(workspace)/book/page.tsx` 第 265-285 行（`handleCreate`）、第 507-522 行（Creator 渲染）

**关键 API**：
- `bookApi.create(payload)` — 创建教材并生成提案
- `bookApi.confirmProposal(bookId, editedProposal)` — 确认提案

---

### 阶段二：大纲编辑（Spine Editor）

**对应 View**：`view === "spine"`

**业务场景**：学生确认提案后，进入大纲编辑阶段。这里可以调整章节结构、增减章节、修改章节标题。

**交互**：
- **章节列表**：展示所有章节，每个章节包含标题和描述。
- **拖拽排序**：可拖拽调整章节顺序。
- **增删章节**：添加新章节或删除不需要的章节。
- **确认大纲**：确认后，AI 开始逐页生成教材内容。

**源码路径**：`web/app/(workspace)/book/page.tsx` 第 302-318 行（`handleConfirmSpine`）、第 524-534 行（SpineEditor 渲染）

**关键 API**：
- `bookApi.confirmSpine(bookId, spine, autoCompile)` — 确认大纲，`autoCompile=true` 表示自动开始编译第一页

---

### 阶段三：编译（Page Compilation）

**业务场景**：AI 根据大纲逐页生成教材内容，每页由多个 Block 组成。

**实现**：
- 这是一个**异步过程**，通过 WebSocket 实时推送进度。
- 每页生成时，状态从 `pending` → `planning` → `generating` → `ready`。
- 前端通过 WebSocket 订阅 `bookId` 的实时事件，更新进度时间线。

**源码路径**：`web/app/(workspace)/book/page.tsx` 第 137-165 行（WebSocket 订阅）、第 320-336 行（`compilePage`）

**关键 API**：
- `bookApi.compilePage(bookId, pageId, force)` — 编译指定页面
- `openBookSocket(callback)` — 订阅教材的实时事件流

---

### 阶段四：阅读（Reader）

**对应 View**：`view === "reader"`

**业务场景**：学生阅读已生成的教材，与内容互动。

**页面布局**：
```
┌──────────────────────────────────────────────────┐
│ [侧边栏] │         [阅读区]          │ [聊天面板] │
│          │ ┌──────────────────────┐  │ (可折叠)   │
│ 章节列表  │ │  页面标题             │  │           │
│ ○ 第1章  │ │  ┌────────────────┐  │  │ 当前页面   │
│ ● 第2章  │ │  │ 文本 Block     │  │  │ 专属聊天   │
│ ○ 第3章  │ │  └────────────────┘  │  │           │
│          │ │  ┌────────────────┐  │  │           │
│          │ │  │ 测验 Block     │  │  │           │
│          │ │  │ [A] [B] [C] [D]│  │  │           │
│          │ │  └────────────────┘  │  │           │
│          │ │  ┌────────────────┐  │  │           │
│          │ │  │ 代码 Block     │  │  │           │
│          │ │  └────────────────┘  │  │           │
│          │ └──────────────────────┘  │           │
└──────────────────────────────────────────────────┘
```

**源码路径**：`web/app/(workspace)/book/page.tsx` 第 536-571 行（Reader 渲染）

---

## 核心组件详解

### 1. 书架列表 (`BookLibrary.tsx`)

**作用**：展示所有已创建的教材，类似"书架"。

**交互**：
- 以卡片网格形式展示每本教材，显示标题、状态、页数、更新时间。
- 点击进入教材。
- 支持删除教材。

**教材状态**：
- `draft`：草稿，尚未确认提案
- `spine_ready`：大纲已确认，等待编译
- `ready`：已编译完成，可以阅读
- `partial`：部分页面编译完成
- `error`：编译出错

---

### 2. 侧边栏 (`BookSidebar.tsx`)

**路径**：`web/app/(workspace)/book/components/BookSidebar.tsx`（第 1-280+ 行）

**作用**：教材的"目录"，展示所有页面，支持折叠/展开。

**交互**：
- **展开模式**：显示完整页面列表，每页显示标题、状态标签和页码。
- **折叠模式**：只显示圆形页码按钮，节省空间，鼠标悬停显示标题。
- **页面状态指示**：每个页面用不同颜色/图标标记状态：
  - 🟢 `ready`：已编译完成
  - 🟡 `generating`：正在编译
  - ⚪ `pending`：等待编译
  - 🔴 `error`：编译失败
  - 🟠 `partial`：部分完成
- **返回书架**：点击返回按钮回到书架列表。
- **重建教材**：点击重建按钮，使用当前章节结构重新生成所有页面。

**源码位置**：`BookSidebar.tsx` 第 34-280+ 行

---

### 3. 页面阅读器 (`PageReader.tsx`)

**路径**：`web/app/(workspace)/book/components/PageReader.tsx`（第 1-500+ 行）

**作用**：教材页面的核心阅读组件，渲染页面中的所有 Block，并提供各种交互。

**交互功能**：

#### 3.1 折叠式标题
- 页面上方显示标题，向下滚动时标题自动折叠，节省阅读空间。
- 滚动回顶部时标题自动展开。
- 也可手动点击按钮折叠/展开。

#### 3.2 Block 操作
每个 Block 都有以下操作（鼠标悬停时显示）：
- **重新生成**：让 AI 重新生成这个 Block 的内容。
- **删除**：删除这个 Block。
- **上移/下移**：调整 Block 在页面中的顺序。
- **更改类型**：将 Block 从一种类型改为另一种（如把"文本"改为"提示框"）。
- **深度探索**：对 Block 中的知识点发起 Deep Dive，生成扩展页面。

#### 3.3 插入 Block
- 页面底部有"+"按钮，点击展开 Block 类型菜单。
- 选择类型后插入一个空 Block，AI 自动生成内容。

#### 3.4 页面大纲导航
- 右侧（或底部）显示页面内的大纲导航，快速跳转到页面中的各个标题。

**源码位置**：`PageReader.tsx` 第 49-500+ 行

---

### 4. Block 类型一览

**定义位置**：`PageReader.tsx` 第 16-28 行

| Block 类型 | 标识 | 业务含义 | 交互方式 |
|-----------|------|---------|---------|
| 文本 | `text` | 普通文本内容（Markdown 渲染） | 阅读 |
| 提示框 | `callout` | 重点提示、警告、笔记等 | 阅读 |
| 测验 | `quiz` | 选择题/填空题 | 学生答题，即时反馈 |
| 代码 | `code` | 代码示例（带语法高亮） | 阅读/复制 |
| 时间线 | `timeline` | 历史事件时间线 | 交互式浏览 |
| 闪卡 | `flash_cards` | 知识点闪卡（正面问题/背面答案） | 翻转查看 |
| 图表 | `figure` | 图片/图表 | 查看 |
| 交互式 | `interactive` | 交互式组件（如 GeoGebra 图形） | 操作 |
| 动画 | `animation` | 数学动画 | 播放 |
| 深度探索 | `deep_dive` | 扩展学习入口 | 点击生成新页面 |
| 用户笔记 | `user_note` | 学生自己的笔记 | 编辑 |

**Block 渲染器**：`web/app/(workspace)/book/components/blocks/BlockRenderer.tsx` 根据 Block 类型分发到不同的渲染组件。

---

### 5. 深度探索（Deep Dive）

**业务场景**：学生在阅读教材时，对某个知识点特别感兴趣，想深入了解。

**交互流程**：
1. 在 Block 上点击"深度探索"按钮。
2. AI 根据该 Block 的内容和选中的知识点生成一个新的扩展页面。
3. 新页面自动插入到当前页面之后，侧边栏自动更新。
4. 学生被导航到新页面。

**源码路径**：`page.tsx` 第 402-419 行（`handleDeepDive`）

**关键 API**：
- `bookApi.deepDive({ book_id, parent_page_id, topic, block_id })` — 创建深度探索页面

---

### 6. 测验交互

**业务场景**：教材中嵌入了测验 Block，学生在阅读过程中可以随时自测。

**交互**：
- 选择题：点击选项，即时判断对错。
- 填空题：输入答案，提交后判断。
- **答错补救**：如果学生答错，系统自动调用 `supplement` API，AI 针对这个知识点生成补充说明，追加到页面中。
- **答题记录**：每次答题结果都记录到后端。

**源码路径**：`page.tsx` 第 422-447 行（`handleQuizAttempt`）

**关键 API**：
- `bookApi.recordQuizAttempt(...)` — 记录答题结果
- `bookApi.supplement(bookId, pageId, topic)` — 生成补充材料

---

### 7. 页面聊天面板 (`BookChatPanel.tsx`)

**作用**：每个页面专属的 AI 对话面板，学生可以针对页面内容提问。

**交互**：
- 点击右下角的"Chat"按钮展开聊天面板。
- 面板从右侧滑入，与页面内容并排显示。
- 聊天会话与页面绑定：每个页面有一个独立的 `page_chat_session_id`。
- 切换页面时，聊天面板自动切换到对应页面的会话。
- AI 在回答时自动引用当前页面的内容作为上下文。

**源码路径**：`page.tsx` 第 582-603 行（Chat 按钮和面板）、第 449-464 行（`handlePageChatSession`）

---

### 8. 进度时间线 (`BookProgressTimeline.tsx`)

**作用**：展示教材编译过程中 AI 的实时工作进度。

**交互**：
- **迷你模式**：在阅读器右上角显示一个紧凑的进度指示器（一个小圆点）。
- **完整模式**：在创建器中显示完整的进度时间线，包括每个阶段的耗时和状态。

**源码路径**：`web/app/(workspace)/book/components/BookProgressTimeline.tsx`

**数据来源**：WebSocket 实时推送的 `BookEngine` 事件，通过 `reduceBookEvent` reducer 累积状态。

---

### 9. 健康状态横幅 (`BookHealthBanner.tsx`)

**作用**：检查教材中是否有编译失败的页面，提示用户修复。

**交互**：
- 如果教材中有 `error` 状态的页面，在阅读器顶部显示警告横幅。
- 点击"重新编译"按钮可以重新编译失败的页面。

**源码路径**：`web/app/(workspace)/book/components/BookHealthBanner.tsx`

---

### 10. 教材重建

**业务场景**：学生对教材的章节结构做了调整（增删章节），或者想用新的模型重新生成所有内容。

**交互**：
- 在侧边栏点击"重建"按钮。
- 弹出确认对话框："将使用当前章节结构重建教材，已有页面会被替换。"
- 确认后，AI 重新编译所有页面。

**源码路径**：`page.tsx` 第 242-263 行（`handleRebuildBook`）

**关键 API**：
- `bookApi.rebuild(bookId, autoCompile)` — 重建教材

---

## 实时通信机制

**WebSocket 订阅**：
- 当学生进入某本教材时，前端自动打开一个 WebSocket 连接到 `/book/{bookId}` 端点。
- 后端推送的每个事件都被 `reduceBookEvent` reducer 处理，更新进度状态。
- 关键事件类型：`block_ready`、`block_error`、`page_compiled`、`page_planned`、`spine_ready`。
- 收到这些事件后，前端自动刷新教材详情，获取最新数据。

**源码路径**：`page.tsx` 第 137-165 行

---

## 数据流总结

```
用户操作 → bookApi → 后端 BookEngine → WebSocket 推送事件 → 前端更新 UI
                                                              ↓
                                                         progress reducer
                                                              ↓
                                                    BookProgressTimeline
                                                    BookHealthBanner
                                                    侧边栏状态更新
```

---

## 与其他模块的联动

- **知识库**：创建教材时可以引用知识库中的文档作为素材。
- **笔记本**：教材中的测验结果可以关联到学生的错题本。
- **Chat 界面**：页面聊天面板本质上是嵌入的 Chat 组件，共享会话管理。
- **题库**：创建教材时可以引用题库中的题目。

---

## 比喻总结

如果把传统教材比作一本**纸质书**，那 Book 系统就是一本**活的魔法书**：

- 📝 **创建阶段**：你对魔法书说"我想学微积分"，它自己就写出了目录和内容。
- ✏️ **大纲阶段**：你可以调整目录，说"第三章太长了，拆成两章"，魔法书照做。
- 🏗️ **编译阶段**：魔法书一页一页地"长出来"，每页都是量身定制的。
- 📖 **阅读阶段**：你不只是读，还能在书里答题、翻闪卡、看动画、点"我想了解更多"就自动生成新章节。
- 💬 **聊天阶段**：遇到不懂的，直接问魔法书，它会结合当前页面的内容给你解释。

Book 系统把"教材"从静态的文本变成了一个**可对话、可交互、可生长的学习伴侣**。

