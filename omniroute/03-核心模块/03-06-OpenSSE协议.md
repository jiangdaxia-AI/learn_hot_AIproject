# 03-06 · OpenSSE 协议 — 深度剖析

> **文件数**：open-sse/ | **核心**：60+ 个执行器 | **功能**：Web 客户端模拟 + OpenAI 兼容 API

---

## 定位

OpenSSE 是 OmniRoute 的"万能钥匙"。它通过模拟 Web 浏览器访问 AI 服务的网页版，让用户免费使用原本需要付费的 AI 服务。

> 来源：open-sse/ 目录

---

## 生活化比喻

> OpenSSE 像**免费试听课的预约系统**：
> - 很多 AI 服务（ChatGPT、Claude、Gemini）提供网页版免费使用
> - 但网页版只能手动操作，不能通过 API 调用
> - OpenSSE 模拟一个"机器人"自动打开网页、输入问题、读取回答
> - 用户通过 API 调用，背后是机器人在操作网页

---

## 一、60+ 个执行器

每个执行器模拟一个 AI 服务的 Web 客户端。实际有 **60+ 个执行器**（`open-sse/executors/` 目录），以下是代表性示例：

| 执行器 | 模拟的服务 | 大白话 |
|--------|-----------|--------|
| chatgpt-web | ChatGPT 网页版 | 免费用 ChatGPT |
| claude-web | Claude 网页版 | 免费用 Claude |
| codex | OpenAI Codex | 免费用 Codex 编程助手 |
| cursor | Cursor AI | 使用 Cursor 的免费额度 |
| deepseek-web | DeepSeek 网页版 | 免费用 DeepSeek |
| grok-web | Grok 网页版 | 免费用 Grok |
| perplexity-web | Perplexity 网页版 | 免费用 Perplexity 搜索 |
| gemini-web | Google Gemini 网页版 | 免费用 Gemini |
| kimi-web | 月之暗面 Kimi 网页版 | 免费用 Kimi |
| qwen-web | 通义千问网页版 | 免费用通义千问 |
| ... | 还有 50+ 个 | 覆盖几乎所有主流 AI 服务 |

> 来源：open-sse/executors/ 目录

---

## 二、Cursor 会话管理器 — 保持登录状态

### 问题

Web 客户端模拟需要保持登录状态（Cookie/Token），但会话会过期。

### 解决方案

CursorSessionManager 管理会话生命周期：创建、复用、过期清理。

### 内部算法：5 步会话管理

```
步骤 1：open() — 创建新会话
  - 获取浏览器 Cookie/Token
  大白话：打开浏览器，登录 AI 服务

步骤 2：enforceMaxSessions() — 限制最大会话数
  - 防止创建太多会话被服务商封禁
  大白话：控制同时登录的人数

步骤 3：attachCloseHandlers() — 注册关闭回调
  - 会话过期时自动清理
  大白话：设置"自动退出"时间

步骤 4：get() — 获取活跃会话
  - 优先复用已有会话
  大白话：如果已经登录了，直接用

步骤 5：close() — 关闭会话
  - 释放资源
  大白话：退出登录
```

> 来源：open-sse/services/cursorSessionManager.ts:79

---

## 三、Codex Device Code 认证 — 5 步状态机

### 问题

Codex 使用 Device Code 流程认证，需要用户在浏览器中授权。

### 解决方案

CommandCode 状态机管理整个 Device Code 流程：

| 状态 | 英文 | 大白话 |
|------|------|--------|
| 等待授权 | PENDING | 生成了一个授权码，等用户在浏览器中确认 |
| 已收到 | RECEIVED | 用户在浏览器中确认了 |
| 已完成 | COMPLETED | 授权成功，可以开始使用 |
| 已过期 | EXPIRED | 授权码过期了，需要重新生成 |
| 已拒绝 | REJECTED | 用户拒绝了授权 |

> 来源：src/lib/db/commandCodeAuth.ts L42 AuthSessionRow

---

## 四、关键设计决策

| 设计决策 | 大白话 |
|----------|--------|
| 60+ 个执行器各自独立 | 一个执行器挂了不影响其他，像"每个试听课独立预约" |
| 会话复用 | 不用每次请求都重新登录 |
| 最大会话限制 | 防止被 AI 服务商封号 |
| Device Code 认证 | 不需要用户输入密码，更安全 |

> 来源：open-sse/ 目录、codegraph explore
