---
date: 2026-04-10
title: Agent 不只是会调工具：我理解的五种开发范式
category: AI
tags:
- AI Agent
- Agent Architecture
- LLM
- LangGraph
- OpenAI
- Anthropic
description: 基于 Anthropic、OpenAI、Google、LangGraph、AutoGen、CrewAI 等一手资料，梳理今天 Agent 开发最常见的五种范式，以及它们各自适合解决什么问题。
---

# Agent 不只是会调工具：我理解的五种开发范式

过去一年里，“Agent”几乎成了 AI 圈最不缺的词。做搜索的在讲 Agent，做代码助手的在讲 Agent，做办公自动化的也在讲 Agent。可一旦真的开始做系统，就会发现：大家说的并不是同一种东西。

有些 Agent，本质上只是“一个会调用工具的对话模型”；有些 Agent，则更像一套可持续运行的工作流系统；再往前走，还有多 Agent 协作、长期记忆、会话恢复、调度执行这些能力。它们都叫 Agent，但开发范式已经完全不同了。

最近我系统看了一遍几家官方文档和主流框架资料，最大的感受反而很朴素：今天 Agent 的工程实践，正在从“让模型自由发挥”转向“围绕模型、工具、状态和编排构建一个可控运行时”。

而且，这件事已经有了相当清晰的行业共识。

比如 [Anthropic 在 Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) 里，明确区分了两类系统：

- workflow：LLM 和工具按预定义代码路径运行
- agent：由 LLM 动态决定过程与工具使用

OpenAI 在 [Building agents](https://developers.openai.com/tracks/building-agents) 里，则把 Agent 平台直接抽象成四个基础原语：

- models
- tools
- state/memory
- orchestration

这两个表述放在一起，其实已经很接近今天 Agent 开发的主线：问题不再是“要不要做 Agent”，而是“你打算怎样组织模型、工具、状态和流程”。

## 一、Agent 开发为什么会出现“范式分化”

早期很多 Agent 系统，本质上还是一次性对话：

- 给一个 system prompt
- 暴露几个工具
- 让模型自己判断何时调用
- 调完后把结果拼回上下文

这套方式能跑，但一遇到复杂任务，问题很快就出现了：

- 多步骤任务容易失控
- 上下文会快速膨胀
- 很难回放和调试
- 一旦跨会话，就要重新处理状态
- 多角色协作会变得混乱

所以 Agent 开发慢慢分化出了几条路线。它们不是互相替代，而是在解决不同层次的问题：

- 如何让模型用工具
- 如何让任务稳定执行
- 如何让复杂任务可拆分
- 如何让多个 Agent 协作
- 如何让 Agent 跨时间持续存在

如果硬要浓缩成一句话，我会这么说：

> Agent 的开发范式，本质上是在回答“该把多少控制权交给模型，又该把多少结构写回系统”。

## 二、今天最主流的五种 Agent 开发范式

### 1. 单 Agent + Tool Calling

这是最基础，也依然最常用的一种。

系统结构很简单：

- 一个主模型
- 一组工具，比如搜索、代码执行、数据库、RAG、浏览器
- 模型根据上下文决定要不要调用工具
- 应用层负责把状态串起来

这类范式的优点非常明显：

- 架构最简单
- 开发速度快
- 成本相对低
- 对大多数“助手型产品”已经够用

但它的问题也很直接：

- 复杂任务下鲁棒性有限
- 长链路执行容易漂移
- 很依赖 prompt 和工具定义质量
- 可观测性和回放能力通常不强

适用场景：

- 客服助手
- 简单知识问答
- 单轮或短流程任务
- 轻量 coding copilot
- 内部工具助手

如果只看工程落地，我反而觉得这类系统最容易被低估。因为很多团队最后发现，自己真正需要的不是“自治 Agent”，而只是“带工具的高质量助手”。

OpenAI 的 [Agents SDK](https://openai.github.io/openai-agents-python/) 就很能代表这个方向：它保留了单 Agent 的简洁性，但在外层补上了 guardrails、sessions、handoffs 和 tracing。

### 2. 工作流 Agent：把不确定性关进节点里

这是我认为当前最成熟的生产范式之一。

它的核心思想不是让模型全程自由决策，而是先把系统拆成一系列节点：

- 分类
- 检索
- 路由
- 生成
- 校验
- 执行
- 人工确认
- 汇总输出

每个节点都可以是：

- 一个 LLM 调用
- 一个工具调用
- 一个规则判断
- 一个人工审批动作

Anthropic 在那篇文章里，把这类 workflow 进一步拆成了几个非常实用的模式：

- Prompt chaining
- Routing
- Parallelization
- Orchestrator-workers
- Evaluator-optimizer

它的建议也很明确：先从简单、可组合的模式开始，不要一上来就追求复杂自治。这个判断我很认同。

[LangGraph 的文档](https://docs.langchain.com/oss/python/langgraph/workflows-agents) 其实也在强化同样的思想：workflow 有预定义代码路径，agent 则更动态；但真正可生产化的系统，往往需要 persistence、durable execution、interrupts、memory 这些工程能力来托底。

这类范式的优点是：

- 可控性强
- 好测试、好回放
- 易于观测
- 很适合接企业流程

缺点是：

- 设计成本更高
- 灵活性不如纯 Agent
- 流程复杂后，维护成本会转移到编排层

适用场景：

- 审批与工单流转
- 内容生产流水线
- 检索、分析、总结类链路
- 需要 SLA、审计、可回放的企业系统

如果说单 Agent 是默认起点，那么 workflow/graph 基本就是今天生产系统的主流答案。

### 3. 规划-执行：让复杂任务先拆，再做

当任务开始变复杂，只靠单轮工具调用就不够了。这时候最常见的升级路线，就是 Planner-Executor。

它的基本结构通常是：

- Planner：先把目标拆成计划
- Executor：逐步执行计划
- 必要时重新规划
- 有时再加一个 Critic 做质量校验

这类范式介于“固定工作流”和“完全自治”之间。它比 workflow 更灵活，又比自由 Agent 更容易控制。

这条路线背后的经典研究和工程模式包括：

- [ReAct](https://arxiv.org/abs/2210.03629)：把 reasoning 和 acting 交替起来
- [Plan-and-Solve Prompting](https://aclanthology.org/2023.acl-long.147/)：先显式规划，再逐步求解
- [Reflexion](https://arxiv.org/abs/2303.11366)：让 Agent 基于 verbal feedback 自我修正

Anthropic 提到的 orchestrator-workers 和 evaluator-optimizer，本质上也都和这类思路高度相关。

它的优点是：

- 更适合复杂任务
- 有明确的任务拆解层
- 对 coding、research、分析型任务很有效

缺点是：

- 计划本身可能不可靠
- 计划和执行容易脱节
- token、延迟和状态复杂度都会上升

适用场景：

- 多文件代码修改
- 深度研究任务
- 多数据源分析
- 复杂自动化任务拆解

我自己的感觉是：这是“高级单 Agent”最值得学的一条路线。很多真正好用的 coding agent，本质上都在这里。

### 4. 多 Agent 协作：把复杂度拆给角色，而不是塞给一个大脑

多 Agent 是最容易被神化的一类，但它不是没价值，只是要用得克制。

它的核心做法是把不同职责拆给不同角色：

- coordinator / manager
- researcher
- coder
- reviewer
- critic
- tool specialist

这些 Agent 之间常见的协作方式包括：

- handoff：把任务转交给另一个 agent
- team chat：多个 agent 在共享上下文中协作
- hierarchical decomposition：主 agent 派发给子 agent

[OpenAI Agents SDK 的 handoffs](https://openai.github.io/openai-agents-python/handoffs/) 就是很典型的代表；[AutoGen](https://microsoft.github.io/autogen/stable/) 则更明确地把自己定义为 event-driven 的多 Agent 框架；[Google ADK 的 multi-agent systems 文档](https://google.github.io/adk-docs/agents/multi-agents/) 甚至把常见模式都写成了清单，比如 coordinator/dispatcher、parallel fan-out/gather、review/critique、human-in-the-loop；[CrewAI](https://docs.crewai.com/) 也把 agents、crews、flows 做成了非常直白的产品抽象。

多 Agent 的优点是：

- 角色边界清晰
- 复杂任务更容易分治
- 某些任务质量上限更高

缺点则同样明显：

- 成本更高
- 调试更难
- 上下文同步复杂
- 很容易出现“看起来很忙，其实只是 token 膨胀”的假协作

适用场景：

- 复杂 research
- 代码生成 + 审查 + 安全校验
- 角色边界清晰的业务流程
- 需要明确职责分工的系统

如果单 Agent + workflow 已经能解决问题，我的建议始终是：先不要上多 Agent。

### 5. 长时运行 Agent：真正的难点，往往不是回答，而是持续存在

这是很多人一开始不会重视，但做久了迟早要补的一层。

当 Agent 不再只是“一次请求”，而要跨会话、跨时间、可恢复、可调度时，开发范式就发生了根本变化。你要考虑的不再只是 prompt 和工具，而是：

- session persistence
- memory store
- resume / checkpoint
- long-running workflow
- human-in-the-loop
- context compaction
- scheduling

OpenAI 的 [Sessions 文档](https://openai.github.io/openai-agents-python/sessions/) 已经把 memory operations、SQLite/Redis/SQLAlchemy session 等能力纳入了标准能力；LangGraph 明确强调 persistence、durable execution、interrupts、time travel；Google 的 [Vertex AI Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview) 也把 sessions、memory bank、deployment、observability 做成了托管能力。

这类范式的价值在于：

- 能支撑长期任务
- 能做跨会话个性化
- 能处理失败恢复
- 更接近真实业务中的 Agent 运行方式

但它的难点也更深：

- memory 很容易污染
- 检索和写回策略非常关键
- 需要更严格的数据治理和权限控制

适用场景：

- 长周期任务助手
- 持续运行的 coding / ops agent
- 销售、客服、运营类长期会话系统
- 带调度能力的自动化平台

说到底，真正进入 production 之后，Agent 比拼的往往不是“这一轮多聪明”，而是“过了三天、三周、三个月以后，它还能不能稳定地继续工作”。

## 三、把五种范式放在一张表里看

| 范式 | 核心思路 | 优点 | 风险 | 更适合什么场景 |
|---|---|---|---|---|
| 单 Agent + Tool Calling | 一个主模型配多种工具 | 简单、快、成本低 | 长流程不稳 | 助手、问答、轻量自动化 |
| Workflow / Graph | 把任务拆成节点与边 | 可控、可观测、可测试 | 编排层复杂 | 企业流程、生产链路 |
| Planner-Executor | 先规划，再逐步执行 | 适合复杂任务 | 计划可能失真 | coding、research、分析 |
| Multi-Agent | 把职责拆给多个角色 | 分工清晰、可分治 | 调试难、成本高 | 复杂协作任务 |
| Long-running / Stateful | 让 Agent 跨时间持续运行 | 可恢复、可记忆、可调度 | 状态治理难 | 生产级 Agent 平台 |

如果只让我给一个非常务实的建议，那就是：

> 不要直接从左跳到最右。最好的路线，通常是从单 Agent 起步，再逐步补 workflow、planning、memory 和持久化能力。

## 四、主流框架，其实分别代表了不同取向

很多人会把框架之争理解成“谁更强”，但我越来越觉得，它们真正代表的是不同的开发重心。

### OpenAI Agents SDK：轻量 runtime + 平台原语

OpenAI 的路线比较像“先给你基础运行时原语，再逐步往外长”：

- agents
- tools
- guardrails
- handoffs
- sessions
- tracing

它适合快速构建单 Agent 或轻量多 Agent 系统，尤其适合已经在 OpenAI 工具栈里的团队。

参考：
- [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
- [Guardrails](https://openai.github.io/openai-agents-python/guardrails/)
- [Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
- [Sessions](https://openai.github.io/openai-agents-python/sessions/)

### LangGraph：workflow/graph-first 的生产导向

LangGraph 给我的感觉一直很明确：它首先是一个 agent orchestration runtime，而不是“帮你写 prompt 的糖衣”。

它强调的是：

- graph execution
- persistence
- durable execution
- interrupts
- memory
- subgraphs

如果你做的是复杂流程、长时任务、需要恢复与回放的系统，LangGraph 的设计思路会很自然。

参考：
- [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents)

### AutoGen：多 Agent 协作的实验场与工程框架

AutoGen 很鲜明地站在 multi-agent 这边。它的问题意识更偏向：

- 如何让多个 agent 协作
- 如何让对话驱动复杂任务
- 如何把 agent system 变成 event-driven 程序

它很适合探索多角色协作，但也因此更需要良好的状态与边界管理。

参考：
- [AutoGen Docs](https://microsoft.github.io/autogen/stable/)

### CrewAI：把“组织流程”映射成 agent team

CrewAI 的抽象非常直白：

- agent
- crew
- flow
- process

它适合那些希望用角色和流程来表达业务自动化的人。优点是上手直观，缺点是如果系统规模继续长大，仍然会遇到 workflow 和 state 管理的老问题。

参考：
- [CrewAI Docs](https://docs.crewai.com/)

### Semantic Kernel：把 Agent 放回企业应用框架里

Microsoft 的 [Semantic Kernel Agent Framework](https://learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/) 让我看到的是另一种思路：它并不执着于把 Agent 做成一个“独立新物种”，而是更强调如何把 agentic patterns 融回现有企业软件栈。

如果你的场景本来就深嵌在企业应用、插件、流程和 .NET 生态里，这条路线会更顺手。

### Google ADK / Vertex Agent Engine：开发工具包 + 托管运行时

Google 的 Agent 路线是两层的：

- [ADK](https://google.github.io/adk-docs/) 负责开发范式，里面有 workflow agents、sequential/parallel/loop agents、multi-agent systems
- [Vertex AI Agent Engine](https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview) 负责托管运行时，提供 sessions、memory bank、deployment、observability

这条路线的重点很清楚：不仅告诉你怎么写 Agent，也告诉你怎么把它真的跑起来。

## 五、哪些模式已经成熟，哪些还偏“好看但难用”

如果按照今天的工程成熟度来分，我会把它们大致分成三层。

### 已经比较成熟的

- 单 Agent + tool calling
- workflow / graph orchestration
- planner-executor
- evaluator-optimizer
- session + memory + human-in-the-loop

这些东西已经不是概念展示，而是很多真实系统都在用的结构。

### 半成熟、需要克制使用的

- orchestrator-worker
- handoff-based multi-agent
- review / critic agent

它们是有价值的，但要求更高的可观测性、测试和边界设计。

### 还偏实验或容易被营销放大的

- 大规模 agent society
- 完全无约束的 autonomous agent
- 自我进化闭环 agent
- 靠多 agent 自发协商解决大多数问题

像 [Reflexion](https://arxiv.org/abs/2303.11366) 和 [Voyager](https://voyager.minedojo.org/) 这样的工作很有启发性，但如果直接照搬进生产，通常会遇到可控性和成本问题。

## 六、如果今天让我给一个 Agent 落地路线

如果是我要从零做一个 Agent 系统，我大概会按下面这条路线推进。

### 第一步：先做一个真正可靠的单 Agent

先把这些问题解决好：

- 模型选型
- 工具接口定义
- structured output
- 失败处理
- tracing
- eval

不要急着上多 Agent，先把一个 Agent 做扎实。

### 第二步：把高频任务固化成 workflow

当你发现某类任务反复出现，而且失败模式也差不多，就应该把它从“自由发挥”收束成节点化流程。

这样做的收益，通常比继续堆 prompt 大得多。

### 第三步：只在复杂任务里引入 planning / critique

不是所有任务都值得显式规划，但复杂 coding、research、审查任务很适合。

这时再加：

- planner
- executor
- critic
- retry / replan

系统会比纯单 Agent 稳很多。

### 第四步：最后再补持久化、记忆和调度

当系统开始真的“长期在线”之后，再认真做：

- sessions
- memory policy
- compaction
- checkpoint / resume
- scheduled runs
- human approval

走到这一步，你做的已经不是“一个会回答的 AI”，而是“一个有生命周期的运行系统”。

## 七、我自己的判断

如果一定要给今天的 Agent 开发范式下一个总判断，我会写成下面这句话：

> Agent 的工程实践，已经从“让模型在对话里调用工具”，走向“围绕模型、工具、状态与编排构建一个可持续运行的系统”；而真正成熟的路线，不是直接追求全自主多 Agent，而是从单 Agent 起步，逐步升级到 workflow、planning、memory 和长期运行能力。

这也是为什么我越来越觉得，Agent 的难点从来不只是“智能”，而是“结构”。

不是让模型更像人，而是让系统更像系统。

## 参考来源

### 官方文档 / 一手资料

- Anthropic, Building effective agents  
  https://www.anthropic.com/engineering/building-effective-agents
- OpenAI, Building agents  
  https://developers.openai.com/tracks/building-agents
- OpenAI Agents SDK  
  https://openai.github.io/openai-agents-python/
- OpenAI Agents SDK — Sessions  
  https://openai.github.io/openai-agents-python/sessions/
- OpenAI Agents SDK — Handoffs  
  https://openai.github.io/openai-agents-python/handoffs/
- OpenAI Agents SDK — Guardrails  
  https://openai.github.io/openai-agents-python/guardrails/
- LangGraph Overview  
  https://docs.langchain.com/oss/python/langgraph/overview
- LangGraph Workflows and agents  
  https://docs.langchain.com/oss/python/langgraph/workflows-agents
- Google Agent Development Kit  
  https://google.github.io/adk-docs/
- Google ADK — Multi-agent systems  
  https://google.github.io/adk-docs/agents/multi-agents/
- Vertex AI Agent Engine overview  
  https://cloud.google.com/vertex-ai/generative-ai/docs/agent-engine/overview
- Semantic Kernel Agent Framework  
  https://learn.microsoft.com/en-us/semantic-kernel/frameworks/agent/
- AutoGen Docs  
  https://microsoft.github.io/autogen/stable/
- CrewAI Docs  
  https://docs.crewai.com/

### 论文 / 研究资料

- ReAct: Synergizing Reasoning and Acting in Language Models  
  https://arxiv.org/abs/2210.03629
- Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models  
  https://aclanthology.org/2023.acl-long.147/
- Reflexion: Language Agents with Verbal Reinforcement Learning  
  https://arxiv.org/abs/2303.11366
- Voyager  
  https://voyager.minedojo.org/
