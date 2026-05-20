---
title: "可落地的多智能体架构：Missions 如何让 Agent 团队长期执行软件任务"
category: "AI工具"
layout: default
---

# 可落地的多智能体架构：Missions 如何让 Agent 团队长期执行软件任务

## 简介

本文整理自 AI Engineer 视频 [The Multi-Agent Architecture That Actually Ships — Luke Alvoeiro, Factory](https://www.youtube.com/watch?v=ow1we5PzK-o)。演讲者 Luke Alvoeiro 来自 Factory，曾参与开源 coding agent Goose，本文主要讨论一种名为 **Missions** 的多智能体软件工程架构：如何把单个 coding agent 扩展成可运行数小时到数天的 agent 团队，并通过规划、验证、交接和共享状态降低长期任务漂移。

这篇内容的价值不在于“多 agent 一定要并行”，而是提出了一个更工程化的判断：当前软件工程瓶颈逐渐从模型智能转向**人类注意力与监督带宽**，因此真正可落地的多智能体系统需要解决上下文丢失、验证偏差、协作冲突、模型分工和长周期可观测性问题。

---

## 核心要点

- 软件工程中的新瓶颈不是模型能不能写代码，而是人类能否同时监督足够多任务；多智能体系统的目标是让人决定“做什么”，系统长期执行“怎么做”。
- 多智能体协作可以拆成五类基本模式：委托、创建者-验证者、直接通信、协商、广播；Missions 组合了其中的委托、创建者-验证者、广播和协商。
- Missions 的核心角色是 orchestrator、workers、validators：规划者先澄清需求并产出计划与 validation contract，worker 用干净上下文实现，validator 用独立上下文做验证。
- 长周期 agent 系统不能只依赖实现后生成的测试；测试若由代码倒推，往往只会确认实现者的决定。验证契约必须在编码前定义，作为与实现无关的“完成标准”。
- 软件开发任务不适合盲目高并行，因为 agent 会冲突、重复工作、做出不一致架构决策；更稳妥的模式是“特性级串行 + 只读任务内部并行”。
- 可持续运行数天的关键不是更长上下文，而是结构化交接、里程碑检查、共享状态、行为级 QA 和可观测的 mission control。
- 多模型架构会成为优势：规划、实现、验证需要不同模型能力；把合适模型放到合适角色，比锁死单一 provider 更抗弱点、更能随模型进步而增益。

## 正文

### 1. 问题背景：软件工程瓶颈从智能转向注意力

Luke 的起点判断是：今天的模型已经足够理解大量软件任务，但工程师每天只能推进少数任务，因为每个任务都需要人类注意力、每个 commit 都需要 review。团队可能有 50 个 feature backlog，但真正被推进的只有少数。

因此多智能体系统要解决的问题不是“让一个 agent 更聪明”，而是：

- 人类只决定目标、范围和验收标准；
- agent 系统能在数小时或数天内执行；
- 人类回来时能看到可验证、可追踪、可修复的结果；
- 代码库在任务结束后比开始前更适合人和 agent 继续工作。

### 2. 多智能体协作的五种基本模式

演讲将前沿多智能体框架简化为五类协作原语：

| 模式 | 含义 | 价值 | 风险 / 难点 |
|---|---|---|---|
| Delegation（委托） | 一个 agent 生成子 agent 去完成子任务，再接收结果 | 最简单、最常见，适合调研、读代码、局部实现 | 父 agent 需要能整合和判断结果 |
| Creator-Verifier（创建者-验证者） | 一个 agent 实现，另一个独立 agent 检查 | 分离关注点，降低实现者自证偏差 | 验证标准必须独立于实现 |
| Direct Communication（直接通信） | agent 之间无中心协调地互相通信 | 理论上灵活 | 状态分散，缺少单一事实源，难维护一致性 |
| Negotiation（协商） | agent 围绕共享资源、接口、代码区域进行协调 | 可处理冲突与 win-win 交易 | 需要明确共享资源和决策边界 |
| Broadcast（广播） | 一个 agent 向多个 agent 发布状态、约束、上下文 | 维持长任务一致性，非常实用 | 不够“炫”，但缺少它容易漂移 |

Missions 没有采用完全自由通信，而是把委托、创建者-验证者、广播、协商组合进一个有结构的工作流。

### 3. Missions：不是单次 agent 会话，而是 agent 生态系统

Missions 的基本流程是：

1. 用户描述目标；
2. orchestrator 通过对话澄清范围；
3. 用户批准计划；
4. 系统按 feature / milestone 长时间执行；
5. 每个阶段通过验证、交接和共享状态自我纠偏。

它的关键定义是：**mission 不是一个 agent session，而是由多个 agent 通过结构化 handoff 与共享状态协作的生态系统。**

三类角色分别是：

- **Orchestrator（编排者）**：负责规划、提问、澄清不明确需求，输出 features、milestones 和 validation contract。
- **Workers（执行者）**：负责具体实现。每个 worker 拿到干净上下文，读 spec、实现 feature，并通过 Git commit 把工作交给下一个环节。
- **Validators（验证者）**：负责独立验证。不只跑 lint、type check、test 和 code review，还要验证端到端行为是否成立。

### 4. Validation Contract：在编码前定义“完成”的含义

演讲中特别强调：很多 coding agent 会出现一种假象——实现后写测试，测试通过、覆盖率很高，但这些测试往往是由实现形状倒推出来的，不能真正发现需求层面的 bug。

因此 Missions 在规划阶段就生成 **validation contract**：

- 在任何编码前定义正确性；
- 与具体实现方案独立；
- 复杂项目中可能包含数百条 assertion；
- 每个 feature 被分配一个或多个 assertion；
- 所有 feature 合起来必须覆盖全部 assertion。

这等于把“验收标准”从实现者手里拿出来，变成规划阶段的独立契约。它是长期任务不漂移的核心机制之一。

### 5. 双层验证：代码审查 + 行为级用户测试

每个 milestone 后，Missions 会运行两类 validator：

1. **Scrutiny Validator**  
   偏传统验证：运行测试、类型检查、lint，并为每个已完成 feature 启动专门 code review agent。

2. **User Testing Validator**  
   更像 QA 工程师：启动应用，通过 computer use 或类似能力实际操作页面、填写表单、点击按钮、检查页面渲染与完整功能流。

第二类验证更慢，因为系统需要等待真实应用运行和交互。Luke 提到 Missions 的大部分 wall-clock time 并不是花在生成 token，而是在等待现实世界执行。这个观察很重要：真正的 agent 软件工程瓶颈会转向**环境执行、反馈采集和端到端验证**。

两个 validator 都没有见过实现过程，也不对实现有心理投资，因此验证是有意设计成“对抗性”的。

### 6. 结构化 Handoff：长期任务的记忆不是靠模型记住，而是强制写下来

当 worker 完成 feature 时，不能只说“完成了”，而要填写结构化交接信息：

- 完成了什么；
- 还有什么没完成；
- 运行过哪些命令；
- 每个命令的 exit code；
- 发现了什么问题；
- 是否遵守 orchestrator 规定的流程。

这些交接会在 milestone 边界被检查。如果发现问题，orchestrator 会重新定义修复工作，让 mission 回到正轨。关键思想是：

> 长周期 agent 系统不要寄希望于 agent “记得发生了什么”，而要强制它把事实写下来，并让后续环节处理这些事实。

Factory 提到其最长 mission 已运行 16 天，并认为可以运行 30 天。这个能力来自结构，而不只是上下文长度。

### 7. 执行策略：特性级串行，而不是盲目并行

一个直觉做法是同时跑 10 个 agent，以为就能得到 10 倍吞吐。但 Luke 认为在软件开发领域这通常无效，因为 agent 会：

- 踩到彼此修改；
- 重复工作；
- 做出不一致的架构决策；
- 消耗大量 token，却被协调成本抵消。

Missions 的策略是：

- feature 层面串行执行；
- 同一时刻只运行一个 worker 或 validator；
- feature 内部允许只读并行，例如搜索代码库、研究 API；
- validator 内部也允许只读并行，例如多个 code review agent。

这是一种“串行主干 + 定向只读并行”的设计。看起来慢，但长期任务中错误率下降，正确性会复利。

### 8. Mission Control：长任务需要专门可观测界面

普通 chat 界面不适合持续数天的任务。用户需要快速知道：

- 项目完成了多少；
- 原始预算消耗了多少；
- 当前 active worker 正在做什么；
- 最近 handoff summary 说了什么；
- validator 发现了什么问题；
- 系统将如何调整后续计划。

因此 Factory 构建了 Mission Control 视图。用户可以像项目经理一样监督，也可以批准计划后离开，等系统异步推进。

### 9. Droid Whispering：为不同角色选择不同模型

Missions 假设不同角色需要不同模型能力：

| 角色 | 更需要的能力 |
|---|---|
| 规划 | 慢速、谨慎、深度推理 |
| 实现 | 快速代码流畅性、创造力、工程执行 |
| 验证 | 精确遵循指令、严格检查、低偏差 |

没有单一模型或 provider 在所有角色上都最好。因此团队内部把这种能力称为 **droid whispering**：理解不同 LLM 如何交互、在哪里失败、失败如何在多天任务中叠加，然后有意识地把模型放到合适位置。

一个重要实践是：验证模型甚至可以使用不同 provider，避免与实现模型共享同类偏差。模型无关架构的结构优势在于：系统不被单一模型家族的最弱能力限制。反过来，validation contract 和 milestone checkpoint 也能让非最前沿模型、甚至 open-weight model 在结构保护下有效工作。

### 10. 生产经验：验证几乎不会第一次成功

Factory 展示了一个构建 Slack clone 的例子，几个值得保留的观察是：

- 约 60% 时间和 token 花在实现上；
- validation 几乎不会第一次成功；
- 系统经常需要创建 follow-up features；
- 最终代码中约 50% 是测试，测试覆盖率约 90%；
- prompt caching 对长任务成本控制很重要。

这说明 QA loop 不是附加功能，而是多智能体长期执行的核心。Missions 的价值不在于“第一次就做对”，而在于能系统性发现问题、生成后续修复任务，并持续收敛。

### 11. 避免被下一代模型淘汰：把编排逻辑写成 prompts 和 skills

多 agent 系统建设者常担心“下一个模型发布就让架构过时”。Missions 的应对方式是让系统随模型进步而变强：

- 大部分编排逻辑写在 prompts 和 skills 中，而不是硬编码状态机；
- feature 分解、失败处理等策略大约由数百行文本控制；
- 少量文本变化就可以显著改变执行策略；
- worker 行为由 orchestrator 为每个 mission 定义的 skills 驱动；
- 确定性逻辑保持很薄，只负责记账、运行验证、阻塞未解决 handoff 问题等。

换句话说，Missions 提供纪律与流程，模型提供智能；系统尽量使用模型已熟悉的 primitives，如 `AGENTS.md`、skills 等。

## 可复用方法 / 行动清单

### 设计长期软件工程 Agent 系统的检查清单

- [ ] 先定义任务目标和范围，而不是直接让 agent 开始写代码。
- [ ] 在编码前生成 validation contract，明确“完成”的外部标准。
- [ ] 将实现者和验证者分离，验证者使用新上下文，必要时使用不同模型或 provider。
- [ ] 每个 worker 完成后必须输出结构化 handoff，包括完成项、未完成项、命令、exit code、问题与流程合规性。
- [ ] 在 milestone 边界集中处理错误、创建 follow-up features，而不是让错误在长上下文中隐性累积。
- [ ] 避免对写操作盲目并行；只对读代码、查文档、代码审查等只读环节并行。
- [ ] 建立 mission-level 可观测界面或文档，记录进度、预算、当前任务、验证结果和计划变更。
- [ ] 为规划、实现、验证选择不同能力画像的模型；不要默认一个模型适合所有角色。
- [ ] 把可变策略写进 prompts / skills，保持 deterministic orchestration 薄而稳定。

### 判断一个多智能体架构是否可落地

可以用以下问题快速评估：

1. 是否有单一事实源，还是 agent 之间到处私聊导致状态碎片化？
2. 是否在实现前定义验收标准，还是实现后让 agent 自己写测试证明自己正确？
3. 是否有独立验证者和端到端行为测试？
4. 是否有结构化交接，能让新 agent 接手而不依赖隐式上下文？
5. 是否能在发现问题后自动创建修复任务并回到主线？
6. 是否明确区分读并行和写并行？
7. 是否能随着模型变强而自然受益，而不是被硬编码流程限制？

## 关键概念

- **Mission**：围绕一个较大软件目标运行的长周期 agent 工作流，不是单一聊天会话，而是由规划、实现、验证和共享状态组成的 agent 生态。
- **Validation Contract**：编码前生成的验收契约，用 assertion 定义正确性，避免实现者用事后测试确认自己的决定。
- **Structured Handoff**：worker 交接时必须记录的结构化事实，包括完成情况、命令、退出码、问题和流程遵守情况。
- **Creator-Verifier Separation**：实现和验证由不同 agent、不同上下文完成，降低自证偏差。
- **Targeted Internal Parallelization**：主流程保持串行，只在只读调研、搜索、代码审查等低冲突环节并行。
- **Droid Whispering**：理解不同模型的能力与失败模式，并为规划、实现、验证等角色选择合适模型的能力。

## 参考资源

- 原视频：[The Multi-Agent Architecture That Actually Ships — Luke Alvoeiro, Factory](https://www.youtube.com/watch?v=ow1we5PzK-o)
- 相关项目 / 组织：Factory、Goose、Open Droid、AI Agentic AI Foundation
- 演讲中提到的入口：Open Droid 中尝试 `/missions`

## 我的理解

这篇演讲的启发是：多智能体软件工程的重点不是“多开几个 agent 并行写代码”，而是把软件工程中本来就有效的机制——规划、验收标准、代码审查、QA、交接、里程碑、可观测性——重新设计成 agent-native 的形式。真正可落地的 agent 系统更像一个有纪律的工程组织，而不是一个更长上下文的聊天机器人。
