# LLM Agent 控制平面设计指南 / LLM Agent Control-Plane Design Guide

> 本文基于 Hermes Harness 源架构（`core/` 与 `specs/Agent-OS-*`）综合编写的双语设计指南；不包含源码，也不代表源仓库的官方文档。
> This bilingual design guide is synthesized from the Hermes Harness source architecture (`core/` and `specs/Agent-OS-*`); it contains no source code and is not the official documentation of the source repository.

## 1. 控制平面定位 / Control-Plane Positioning

LLM Agent 控制平面负责「把模型能力变成受控任务」：理解需求、生成合同、路由项目、控制状态、保留证据。控制平面本身不保存业务知识，也不代替目标项目执行专业工作。

- **发现**：从权威注册表找到项目；
- **调度**：需求编译为合同并路由；
- **状态控制**：有界 Loop 驱动状态迁移；
- **证据闭环**：状态变化写入可恢复账本。

The control plane turns model capability into governed tasks: understanding requests, drafting contracts, routing to projects, controlling state, and keeping evidence. It does not store business knowledge or do the professional work of target projects.

## 2. 控制循环 / The Control Loop

### 2.1 状态机 / State machine

```text
received → understanding → contract_drafted → routed → dispatched → evaluating
→ completed | retrying | waiting_for_user | waiting_for_approval | failed
```

### 2.2 事件驱动与有界 / Event-driven and bounded

- 控制循环只在事件发生时被唤醒：新需求、Worker 返回、用户补充、审批结果、超时；
- **有界执行**：Loop 最多执行两次，只允许一次修正，随后必须 completed 或 failed；
- 受监督模式在 `routed` 与 `dispatched` 之间插入 `awaiting_worker_start` 暂停点：先产出可检查的 Worker 包，批准后才派发。

The loop is awakened only by events (new request, worker result, user clarification, approval, timeout); execution is bounded (two runs, one fix); supervised mode pauses at awaiting_worker_start between routed and dispatched so an inspectable worker package is approved before dispatch.

## 3. Worker 协议 / Worker Protocol

### 3.1 统一 Seam / Unified seam

用三个动作统一接收 Agent、Skill 与本机工具：

- **Prepare**：校验运行任务与适配器，创建最小 Worker 快照与 prepared 作业；
- **Accept**：复算权限与证据哈希，接受一个终态结果（succeeded | failed | blocked）；
- **Inspect**：从 Worker Ledger 恢复状态，复验输入、回执与证据。

### 3.2 能力与权限适配器 / Capability and permission adapters

- 适配器声明 adapter_type（agent | skill | local_tool）、每个能力的上下文槽位、允许根目录、文件数与字节上限、内容模式与输出类型；
- 上下文默认 `context_read=snapshotted_only`；Worker 请求只能绑定声明中的槽位；
- v0.1 不接受发布、删除、付款、账号或核心规则修改等敏感副作用。

### 3.3 最小上下文 / Minimal context

- Worker 输入只含作业内快照路径、来源 SHA-256、任务与合同哈希，**不含源文件路径或运行合同路径**；
- 请求、适配器、运行合同由同一份已读字节计算字段与 SHA-256，避免「检查后替换」；
- Worker 只能读 context/，只能写 output/，结果至少附一份可复算证据。

### 3.4 结构化结果 / Structured results

- 结果必须通过 result Schema：status 只能是 succeeded / failed / blocked，failed/blocked 必须给出 errors；
- 每份证据绑定相对路径、类型、内容模式与 SHA-256；文本模式必须严格 UTF-8；
- Accept 从同一份稳定字节解析并计算哈希，**Worker 自报哈希不作为事实源**。

The protocol unifies agents, skills and local tools behind Prepare/Accept/Inspect. Adapters declare capabilities, context slots, allowed roots, byte limits and permission ceilings. Workers receive minimal context (snapshot paths and hashes only, never source paths), write structured results with at least one recomputable evidence, and the harness recomputes all hashes from stable bytes — self-reported hashes are never trusted.

## 4. 审批边界 / Approval Boundaries

### 4.1 自动 vs 人工 / Automatic vs human-required

可自动：读取已授权规则与状态、生成合同与验证报告、在运行目录创建任务与追加 Ledger、只读检查与可逆验证。

必须等待用户确认：发布/上线/对外沟通；删除、移动、归档与不可逆覆盖；付款、购买、账号或权限变更；正式产物晋级；修改核心权限、质量门或长期记忆规则。

Automatic: reading authorized rules and state, generating contracts and verification reports, creating task instances and appending ledgers, read-only checks and reversible validation. Human-required: publishing or external communication; deletion, moves, archival and irreversible overwrites; payments, purchases, account or permission changes; promotion of formal artifacts; changes to core permissions, quality gates or long-term memory rules.

### 4.2 审批的边界 / Scope of approval

- 审批只能解除**合同中明确列出的动作**，不能扩大整个任务的权限；
- 单次使用审批：消费记录只允许写入一次，重复使用即拒绝；
- 完成门：从 `waiting_for_approval` 进入 `completed` 必须收到显式 `ApproveCompletion`，该开关只对本次完成迁移有效；
- 事实复核：自然语言事实（如内容草稿）结构验证通过后仍停在 `content_fact_review`，人工复核前不得写 completed。

Approval only lifts actions explicitly listed in the contract — never the whole task. Approvals are single-use; completion requires an explicit ApproveCompletion for that transition only; fact-bearing content stays in content_fact_review until a human reviews it.

## 5. 可观测性 / Observability

- **观测合同**：先声明被观测对象（kind 决定必须通过的 Schema）、阶段依赖、统一预算与质量门；
- **统一指标**：每阶段五个非负整数维度——duration_ms、model_calls、input_tokens、output_tokens、cost_microunits（合同货币的百万分之一，避免浮点误差）；
- **预算预留**：StartStage 先判断当前用量 + 运行中阶段预留不会突破任一维度才放行；
- **质量门**：gate_id + metric + unit + operator + threshold；缺测量、单位漂移、证据越界、阈值不合格一律阻断；
- **失败关闭**：预算超限、质量不合格、证据漂移或 Ledger 损坏都必须 fail closed，不降级为警告。

Observability: declare the observed subject, stage dependencies, unified budget and quality gates up front; track five non-negative integer dimensions per stage including cost in microunits; reserve budget before starting a stage; enforce declared quality gates; and fail closed on any budget, quality, evidence or ledger violation.

## 6. 恢复与审计 / Recovery and Audit

- **哈希链 Ledger 是事实源**：每条事件含序号、前态/后态、合同哈希、前一事件哈希与事件哈希；
- **Snapshot 只是缓存**：链完整时由 Ledger 重建，链损坏时只暴露可信前缀且不写回；
- **不可逆动作不自动重试**：任何「是否已经发生」不确定的不可逆派发只能人工对账；
- **终态不重派**：completed/failed/cancelled 之后不接受任何迁移。

The hash-chain ledger is the source of truth; snapshots are caches rebuilt from it, and corruption exposes only the trusted prefix. Irreversible actions are never auto-retried when their occurrence is uncertain — human reconciliation only. Terminal states are never re-dispatched.

## 7. 设计检查清单 / Design Checklist

- [ ] 控制平面只做发现/调度/状态/证据，不存业务知识
- [ ] 状态机事件驱动、有界执行（两次执行、一次修正）
- [ ] Worker 通过统一协议接单，只读最小上下文，结果带证据哈希
- [ ] 能力与权限由适配器声明，敏感副作用默认拒绝
- [ ] 审批只解除合同列出的动作，单次使用，完成需显式人工批准
- [ ] 观测先声明合同与预算，超限/不合格失败关闭
- [ ] Ledger 哈希链为事实源，损坏只暴露可信前缀
- [ ] 不可逆动作不自动重试，只能人工对账

A checklist for designing an LLM Agent control plane: the plane only discovers/schedules/controls/evidences; the loop is event-driven and bounded; workers join through a unified protocol with minimal context and evidence hashes; capabilities and permissions come from adapters with sensitive side effects denied by default; approvals only lift contract-listed actions, are single-use, and completion needs explicit human approval; observability declares contracts and budgets up front and fails closed; the hash-chain ledger is the source of truth; irreversible actions are never auto-retried.