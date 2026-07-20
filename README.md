# 🤖 learn_hot_AIproject

> **持续分享热门 AI 开源项目源码深度剖析学习笔记**
> **Continuously sharing source-code deep-dive notes on trending AI open-source projects.**

<p align="center">
  <a href="https://github.com/jiangdaxia-AI/learn_hot_AIproject/stargazers"><img src="https://img.shields.io/github/stars/jiangdaxia-AI/learn_hot_AIproject?style=social&label=Stars" alt="Stars"></a>
  <a href="https://github.com/jiangdaxia-AI/learn_hot_AIproject"><img src="https://img.shields.io/github/last-commit/jiangdaxia-AI/learn_hot_AIproject?label=Last%20Commit" alt="Last Commit"></a>
  <a href="https://github.com/jiangdaxia-AI/learn_hot_AIproject"><img src="https://img.shields.io/github/repo-size/jiangdaxia-AI/learn_hot_AIproject?label=Repo%20Size" alt="Repo Size"></a>
  <a href="https://github.com/jiangdaxia-AI/learn_hot_AIproject"><img src="https://img.shields.io/github/languages/count/jiangdaxia-AI/learn_hot_AIproject?label=Languages" alt="Languages"></a>
  <img src="https://img.shields.io/badge/updated-continuously-brightgreen?label=Update" alt="Continuously Updated">
  <img src="https://img.shields.io/badge/projects-3%20%26%20growing-blue?label=Projects" alt="Projects">
</p>

---

## 📖 简介 | Introduction

**中文**

本仓库以**源码级颗粒度**逐方法、逐类地深度剖析当下热门的 AI 开源项目，面向产品经理、运营及非技术背景人员，用大白话讲清"**做什么、为什么、怎么做**"。

🔥 **本仓库持续更新**，后续会不断收录更多热门 AI 项目的源码剖析笔记，欢迎 ⭐ Star 关注跟进。

**English**

This repository deep-dives into trending AI open-source projects at the **source-code level** — method by method, class by class. Written for PMs, operators and non-technical readers in plain language: *what it does, why it matters, how it works*.

🔥 **Continuously updated** with more trending AI project analyses. ⭐ Star to stay tuned.

---

## 🗂️ 收录项目 | Included Projects

| 项目 | Project | 简介 | 技术栈 | 源码仓库 |
|------|---------|------|--------|----------|
| **DeepTutor** | DeepTutor | HKUDS 实验室开源的 AI 驱动终身个性化辅导系统，多 Agent 架构 | Python / FastAPI / RAG | [HKUDS/DeepTutor](https://github.com/HKUDS/DeepTutor) |
| **Clawith** | Clawith | dataelement 开源的项目，6 层架构，16,323 代码节点 | — | [dataelement/Clawith](https://github.com/dataelement/Clawith) |
| **jcode** | jcode | 用 Rust 编写的下一代终端 AI 编码代理工具，67 个 crates | Rust | [1jehuang/jcode](https://github.com/1jehuang/jcode) |

---

## 📂 目录结构 | Directory Structure

```
├── Clawith/       # Clawith 源码剖析（系统全景/架构/核心模块/流程/数据实体）
├── DeepTutor/     # DeepTutor 源码剖析（12 部分 38 章，按系统分层组织）
│   └── 涵盖 API层/Agent层/基础设施层/服务层/学习层/能力层/Web层/工具层/核心流程/数据实体
└── jcode/         # jcode 源码剖析（系统全景/架构/核心模块/流程/数据实体）
```

每个项目目录下均包含：

- `00-目录与阅读指南.md` — 全书结构总览与阅读路线建议
- `XX-系统全景.md` — 项目定位、目标用户、痛点、技术栈、竞品对比
- 分层架构与核心模块逐章剖析
- `项目名-全书合并版.md` — 所有章节合并的单文件版

---

## 🧭 阅读建议 | Reading Guide

| 读者类型 | Reader | 建议阅读路径 |
|----------|--------|-------------|
| 🧑‍💼 产品经理/运营 | PM / Ops | 从「系统全景」章开始，快速了解项目全貌 |
| 🏗️ 架构师 | Architect | 重点阅读「架构总览」与「核心流程」 |
| 💻 开发者 | Developer | 按「核心模块 → 数据实体」顺序深入源码细节 |
| ⚡ 快速了解 | Quick Look | 只看每个项目的「系统全景」一章即可 |

---

## 🔬 分析方法 | Methodology

笔记基于 **CodeGraph** 代码索引（节点 + 边）结合源码阅读生成，覆盖函数 / 结构体 / 方法等代码实体及其调用关系，保证剖析的完整性与准确性。

Notes are generated from **CodeGraph** code indexes (nodes + edges) combined with source reading, covering functions / structs / methods and their call relations to ensure completeness and accuracy.

---

## ⭐ Star & Follow

如果这个仓库对你有帮助，欢迎点个 ⭐ **Star**，这是持续更新的最大动力！

If you find this repository helpful, please give it a ⭐ **Star** — it's the biggest motivation to keep updating!

---

## 👤 维护信息 | Maintainer

- 维护人：**jiangdaxia**（[@jiangdaxia-AI](https://github.com/jiangdaxia-AI)）
- 持续收录热门 AI 项目源码剖析，定期更新
- Continuously curating source-code analyses of trending AI projects.
