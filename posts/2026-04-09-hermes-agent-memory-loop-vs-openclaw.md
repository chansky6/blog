---
date: 2026-04-09
title: Hermes Agent vs OpenClaw：两条长期运行 Agent 路线的分野
category: AI
tags:
- Hermes Agent
- OpenClaw
- AI Agent
- Memory
- Agent Architecture
description: 结合 Hermes Agent DeepWiki、官方文档、实现细节，以及 OpenClaw 官方 docs、GitHub 与公开架构分析，比较两者在系统中心、状态管理、扩展机制与记忆架构上的根本分歧。
---

# Hermes Agent vs OpenClaw：两条长期运行 Agent 路线的分野

如果把 2026 年的个人 AI Agent 看成一个正在形成的新范式，那么 Hermes Agent 和 OpenClaw 大概是最值得放在一起看的两条路线。

它们都不满足于“会聊天的 LLM + 几个工具”。它们都在试图回答一个更大的问题：怎样把模型、工具、状态、会话和外部世界真正拼成一个可长期运行的 agent 系统。

但两者的答案并不一样。

我这篇文章主要依据这些材料来判断：

- Hermes Agent 的 DeepWiki、官方文档与实现细节
- OpenClaw 的官方 docs、官网、GitHub 页面
- 几篇公开架构分析文章，但只把其中能被官方资料交叉印证的部分纳入结论

先给结论：

- OpenClaw 更像一套“开放的个人 AI 操作层”：以 channels、agents、tools/plugins、memory engines 为中心，强调能力表面和检索能力
- Hermes Agent 更像一套“收束后的长期运行 agent 平台”：以会话生命周期、记忆闭环、skills、profiles、gateway、cron 为中心，强调连续性和系统一致性

两者都强，但强在不同地方。

## 一、它们其实在回答两个不同的问题

OpenClaw 的公开叙事非常鲜明：

- 它是 personal AI assistant
- 可以存在于 WhatsApp、Telegram、Discord 等聊天入口
- 通过 gateway、agent loop、plugins、memory、multi-agent routing 来组织系统

Hermes 的叙事则是另一种口味：

- self-improving AI agent
- built-in learning loop
- persistent memory
- skills
- session search
- profiles
- cron
- messaging gateway

如果把它们的“系统重心”抽象出来：

- OpenClaw 更像在问：如何把 agent 放进尽可能多的现实接口中，并给它足够大的能力表面？
- Hermes 更像在问：如何让一个 agent 在不同会话、不同平台、不同时间点之间保持连续性，并逐步积累能力？

这是理解两者差异的起点。

## 二、系统中心不一样：OpenClaw 从“外部入口”组织，Hermes 从“持续运行的 agent 内核”组织

这是我看两者资料后最强烈的感受。

### OpenClaw：从外向内的架构

OpenClaw 官方文档里，`Agent Loop`、`Channels`、`Plugins`、`Multi-Agent Routing`、`Gateway` 都是一级概念。公开教程里经常把它描述成三层：

- Channel Layer
- Brain Layer
- Body Layer

而且它明确强调：

- 不同聊天协议先做 channel normalization
- 同一 session 的 agent run 要串行化
- gateway 负责 routing、session、authentication、state consistency
- tools/plugins 是能力表面的一部分

这说明 OpenClaw 的架构组织方式，首先考虑的是“消息从哪里来、经哪条路进系统、由哪个 agent 接住、落到哪种工具和记忆后端”。

它更像一种“消息驱动的 agent 操作层”。

### Hermes：从内向外的架构

Hermes 的 DeepWiki 和实现看下来，更像是另一种展开方式：

- 先有一个稳定的 `AIAgent` conversation loop
- 再有工具系统、memory、skills、session DB
- 然后把 CLI、Telegram、Discord、Slack、ACP、cron 等都接到这个内核外面

也就是说，Hermes 的基本视角不是“某个聊天平台如何接进来”，而是“同一个 agent 内核如何跨入口持续工作，并在不同 turn、不同 session 之间维护连续性”。

如果用一句话概括：

- OpenClaw 更像先设计“agent 如何接触世界”
- Hermes 更像先设计“agent 如何持续存在”

这是两种很不一样的产品哲学。

## 三、长期运行能力：Hermes 更像平台收束，OpenClaw 更像开放编排

### Hermes 的做法：把长期运行做成内建 primitive

Hermes 的长期运行能力不是零散功能，而是一组互相咬合的原语：

- `profiles`：一个 profile 就是一套完全隔离的 agent 环境，带独立 config、memory、sessions、skills、cron 和 gateway 状态
- `gateway`：统一连接 Telegram、Discord、Slack、WhatsApp、Signal 等入口
- `session_search`：把所有 past conversation 放进 SQLite + FTS5，按需召回
- `cron`：把调度直接做成内建能力，而不是外部拼接
- `delegate_task`：把复杂任务拆成隔离 subagent 并行处理

这组 primitive 有一个明显特点：它们都不是“加一个功能”，而是在定义 agent 的长期生存方式。

Hermes 想要的不是一个一次性助手，而是一个：

- 可以常驻在多个入口
- 能保有长期状态
- 能定时自己工作
- 能把经验沉淀成 skill
- 能在不同 profile 间完成职责隔离

的 agent 平台。

### OpenClaw 的做法：把长期运行做成开放组合

OpenClaw 的公开资料则更多体现为：

- channel routing
- session serialization
- multi-agent routing
- plugins
- per-agent workspace
- gateway as daemon

它也显然是长期运行系统，但它更像把这件事拆成可编排的积木：

- 哪个 channel 进来
- 哪个 agent 接住
- 哪种 memory engine 负责 recall
- 哪个 plugin 负责能力扩展

这套思路的好处是自由度高，适合能力快速扩张；代价则是系统心智更重，操作面也更宽。

所以我的判断是：

- Hermes 更像把长期运行“产品化”了
- OpenClaw 更像把长期运行“平台化”了

这两条路都成立，但气质完全不同。

## 四、扩展机制的分歧：Hermes 把“怎么做”外部化成 skill，OpenClaw 把“能做什么”外部化成 plugin

这部分也非常关键。

### Hermes：技能优先

Hermes 的 skills 是一等公民，而且不是普通意义上的 prompt snippet。

它们更像：

- 可按需加载的程序性文档
- 能被 agent 自己创建、更新、修补
- 与 project context、tool availability、cron、slash commands 深度整合

Hermes 的一个核心观点是：

- 记忆负责记“人”和“环境”
- skill 负责记“做法”和“流程”

所以当 Hermes 在一次复杂任务中发现可复用流程时，它倾向于把它沉淀成 skill，而不是继续堆进 memory。

这很像把 agent 的长期能力分成两层：

- fact layer
- procedure layer

这比单纯的“知识库存取”更接近软件工程里的知识分层。

### OpenClaw：插件优先

OpenClaw 的资料里，`Plugins`、`Tools & Plugins`、`Plugin Bundles`、`Building Plugins` 都是高频概念。它当然也有 skills，但从公开结构看，plugin surface 的权重更大。

这意味着 OpenClaw 更强调：

- 如何把外部服务、渠道、工具能力接进来
- 如何扩展 agent 的能力边界
- 如何让一个 agent 触达更多现实世界接口

换句话说：

- Hermes 擅长把“方法论”沉淀为可复用程序性知识
- OpenClaw 擅长把“外部能力”沉淀为可调用系统接口

一个更像“程序性记忆平台”，一个更像“能力接入平台”。

## 五、记忆：最能体现两者分歧的一章

记忆不是这篇文章的全部，但它确实是最能暴露设计哲学的部分。

### Hermes 的记忆不是大仓库，而是记忆循环

Hermes 的内置记忆极其克制：

- `MEMORY.md`
- `USER.md`

默认预算也很小：

- `memory_char_limit = 2200`
- `user_char_limit = 1375`

所以它不是想做一个“大型长期知识库”，而是想做一个始终常驻 system prompt 的高价值短记忆层。

更关键的是它的实现方式。Hermes 的记忆不是静态文件，而是一整套会话生命周期机制：

1. session 开始时从磁盘读取 memory
2. 生成 frozen snapshot 注入当前 system prompt
3. 中途通过 memory tool 写入时，立即落盘，但不改本次 session 的 prompt
4. 每隔若干轮触发 periodic nudge，后台 review 是否值得写 memory / skill
5. 在 compression 前做 memory flush
6. 在 gateway reset 前做 pre-reset flush
7. 新 session 再重新加载新的 snapshot

这套机制里，我认为最值得看的实现细节有三个：

#### 1. Frozen snapshot

Hermes 不会因为 mid-session 新写入 memory，就马上重建 system prompt。

它刻意让本轮 prompt 保持稳定，把 memory 更新延迟到下一 session 再生效。这样做的核心收益是：

- 保持 prefix cache 稳定
- 避免 system prompt 在一个 session 内不断漂移
- 让行为边界更可预测

这是一种非常工程化的选择。

#### 2. Periodic nudge + background review

Hermes 不只依赖模型自发记忆，而是维护 turn-based 计数器。到达阈值后，会在 final response 之后 fork 出一个 review agent，后台审视：

- 有没有值得写入 memory 的用户偏好或长期事实
- 有没有值得沉淀成 skill 的可复用方法

这意味着 Hermes 把“复盘”做成了系统行为，而不是完全交给模型即兴发挥。

#### 3. Flush before loss

Hermes 在上下文即将丢失时会主动触发 memory flush：

- compression 前 flush
- session reset 前 flush
- gateway 中还会带 live memory stale guard，避免旧 transcript 覆盖掉新的 memory 状态

这说明 Hermes 真正关心的问题是：

不是“能不能记”，而是“在上下文丢失之前，能不能把最重要的东西安全地沉淀下来”。

这一点我非常看重。

### OpenClaw 的记忆更像可检索的长期知识系统

对照 OpenClaw，重点就完全不同了。

OpenClaw 官方文档中，memory 体系明显更大、更偏检索层：

- Memory Overview
- Builtin Memory Engine
- QMD Memory Engine
- Memory Search
- Session memory search
- Dreaming

从文档可确认的结构看：

1. Workspace-level memory
- 官方文档明确写到：模型真正“记住”的，是写入 workspace 的内容；there is no hidden state

2. Builtin Memory Engine
- 默认是 per-agent SQLite backend
- 支持 keyword、vector、hybrid search
- 可选 sqlite-vec acceleration

3. QMD Memory Engine
- local-first sidecar
- BM25 + vector + reranking
- 不只是读 workspace memory，还能索引更广内容
- 不可用时可回退到 builtin engine

4. Memory Search
- 对 memory 做 chunking，再做 embeddings / keyword / hybrid retrieval
- 文档还明确谈到了 MMR、temporal decay、session memory search

5. Dreaming
- 文档把它定义成 optional background consolidation pass
- 会从短期信号里筛选、打分、晋升长期记忆
- 还会自动管理 recurring cron job 做 full dreaming sweep

所以 OpenClaw 的 memory 主角色，不是 Hermes 那种“system prompt 常驻短记忆 + flush lifecycle”，而是一个更完整的 retrieval substrate。

### 这两条记忆路线的本质区别

一句话总结：

- Hermes 把 memory 放在“会话生命周期管理”的轴线上
- OpenClaw 把 memory 放在“长期检索与知识整理”的轴线上

更具体一点：

- Hermes 的强项是记忆时机、系统稳定性、生命周期闭环
- OpenClaw 的强项是记忆深度、检索层次、memory engine 扩展性

如果你关心的是：
“这个 agent 能不能把真正重要的偏好和长期事实稳稳带到下一次会话里？”

Hermes 很强。

如果你关心的是：
“这个 agent 能不能拥有一个更像搜索引擎/知识系统的长期记忆后端？”

OpenClaw 更有野心。

## 六、多 agent 与并发模型：Hermes 更强调隔离，OpenClaw 更强调路由

### Hermes：profile 隔离 + subagent delegation

Hermes 的多 agent 模型是两层：

1. 长期层
- `profiles`，每个 profile 是完整隔离的 agent 身份

2. 短期层
- `delegate_task`，为单个任务派生隔离的 subagent

这个设计非常清晰：

- 长期隔离靠 profile
- 临时并行靠 delegation

再加上 gateway 和 cron，Hermes 整体看起来非常像“一个主 agent 平台 + 多个运行实例 + 多种入口”。

### OpenClaw：multi-agent routing + serialized session loop

OpenClaw 的公开资料更强调：

- multi-agent routing
- per-session serialized run
- command queue
- gateway 统一接住不同 channels

这意味着 OpenClaw 在多 agent 问题上的侧重点是：

- 哪个 channel / session 该路由到哪个 agent
- 如何保证同一 session 内 agent loop 串行执行，避免状态打架

所以两边的重点又不一样：

- Hermes 更强调“agent 个体的长期隔离”
- OpenClaw 更强调“消息到 agent 的路由与执行秩序”

## 七、各自的优缺点

### Hermes Agent 的优点

1. 系统收束度高
- memory、skills、profiles、cron、gateway、session_search 之间咬合得非常紧

2. 长期运行体验更完整
- 不只是能接平台，而是把连续性、复盘、调度、隔离都做成了内建 primitive

3. 记忆闭环很成熟
- frozen snapshot、periodic nudge、background review、compression flush、pre-reset flush，这一套很像经过认真系统设计

4. skills 路线非常聪明
- 它把“可复用做法”从记忆里分离出来，降低了 memory 污染

### Hermes Agent 的缺点

1. 内置 memory 容量小
- 适合高价值短记忆，不适合承载庞大的长期知识库

2. 系统更有主张
- 这是优点也是缺点。收束得越好，可塑性边界也越明显

3. 扩展风格偏“收编进平台”
- 对想要非常自由地搭自己的 agent 运行层的人来说，可能没 OpenClaw 那么“野”

### OpenClaw 的优点

1. 架构表面更开放
- channels、plugins、memory engines、multi-agent routing，一看就是高自由度系统

2. memory 栈更大
- builtin SQLite、QMD、hybrid retrieval、dreaming，这一套明显更像完整的长期知识系统

3. channel / gateway 思路更强烈
- 对“agent 如何接触真实世界”这件事，OpenClaw 的公开架构表达非常清楚

4. 更适合 power user
- 如果你想要的是一套能继续扩、继续拼、继续折腾的个人 AI 操作层，它很有吸引力

### OpenClaw 的缺点

1. 系统复杂度更高
- channel、plugin、memory engine、retrieval tuning、sidecar、session routing，本身就是运维和心智负担

2. 强检索不等于强闭环
- memory engine 更丰富，并不自动等于“上下文丢失前最重要的信息一定能稳稳收住”

3. 更容易变成高手系统
- 对普通用户来说，表面能力越多，实际维护成本也越高

## 八、我的判断：OpenClaw 更像开放的个人 AI 操作层，Hermes 更像正在成形的 Agent 平台

如果让我用一句话下判断：

- OpenClaw 更像“开放而激进的个人 AI 操作层”
- Hermes 更像“正在收束成型的长期运行 Agent 平台”

这并不是说谁更先进，而是两边优化目标不同。

OpenClaw 的强，在于它把 channels、plugins、memory engines、multi-agent routing 这些东西都摆上桌面，给高级用户很大的编排空间。

Hermes 的强，则在于它把 agent continuity 这件事做得更像一个平台问题：

- 如何跨 session 保持一致性
- 如何把 memory 做成闭环
- 如何把 skills 变成程序性长期记忆
- 如何让 profile、gateway、cron、session_search 形成一套统一运行模型

从长期 personal agent 的角度看，我个人更偏向 Hermes 这条路。

原因也很简单：

真正能长期工作的 agent，最后比拼的未必是谁的能力表面更大，而是谁能把“持续存在”这件事真正做扎实。

而 Hermes 目前最打动我的，正是这一点。

## 九、资料依据

本文判断主要基于以下材料：

### Hermes Agent
- DeepWiki：Overview、Memory and Sessions、Gateway Architecture、Skills System
- 官方文档：Persistent Memory、Scheduled Tasks、Skills System、Profiles、Sessions、Subagent Delegation
- 实现细节：`run_agent.py`、`tools/memory_tool.py`、`agent/memory_manager.py`、`tools/session_search_tool.py`

### OpenClaw
- 官方 docs：Agent Loop、Memory Overview、Builtin Memory Engine、QMD Memory Engine、Memory Search、Channels、Plugins
- 官网与 GitHub 页面
- 外部分析文章：主要作为辅助阅读，只在与官方 docs 可交叉印证时才纳入结论
