---
name: dsh-llm-agent-harness-guide
description: LLM Agent 控制平面设计指南（控制循环、Worker 协议、审批边界、可观测性）的解读与应用 / Expert for the LLM Agent control-plane design guide: control loop, worker protocol, approval boundaries and observability
---

# LLM Agent 控制平面设计专家 / LLM Agent Control-Plane Design Expert

本技能面向 `docs/llm-agent-harness-guide.md`：LLM Agent 控制平面的设计指南——控制平面定位与控制循环（事件驱动、有界执行、受监督暂停点）、Worker 协议（Prepare/Accept/Inspect、适配器能力与权限、最小上下文、结构化结果与证据哈希）、审批边界（自动 vs 人工、单次使用审批、人工完成门与事实复核）、可观测性（观测合同、统一指标、质量门、失败关闭）以及哈希链 Ledger 恢复。设计或评审 LLM Agent 控制平面时以此为基线。

This skill covers `docs/llm-agent-harness-guide.md`: the LLM Agent control-plane design guide — positioning and the control loop (event-driven, bounded, supervised pause), the worker protocol (Prepare/Accept/Inspect, adapter capabilities and permissions, minimal context, structured results with evidence hashes), approval boundaries (automatic vs human-required, single-use approvals, human completion gates and fact review), observability (observation contracts, unified metrics, quality gates, fail closed) and hash-chain ledger recovery. Use it as the baseline when designing or reviewing an LLM Agent control plane.

## When to use / 何时使用

设计或评审 LLM Agent / 多 Agent 系统的控制平面；需要为 Agent 定义统一 Worker 协议与最小上下文；需要划分自动动作与人工审批边界；需要引入可观测性、预算与质量门；需要设计可恢复的账本与失败关闭语义。

When designing or reviewing the control plane of an LLM agent or multi-agent system; when defining a unified worker protocol with minimal context; when dividing automatic actions from human approval boundaries; when introducing observability, budgets and quality gates; or when designing recoverable ledgers with fail-closed semantics.

## Workflow / 工作流

1. 先按「控制平面定位」明确职责边界，再用「控制循环」设计事件驱动状态机与有界 Loop。
2. 按「Worker 协议」定义统一 seam、适配器能力与权限、最小上下文与结果/证据 Schema。
3. 按「审批边界」列出自动与必须人工的动作，落实单次使用审批与人工完成门。
4. 按「可观测性」声明观测合同、统一指标与质量门，超限失败关闭。
5. 最后用「设计检查清单」逐项评审实现。

1. Define responsibility boundaries (positioning) and design the event-driven, bounded control loop.
2. Define the worker protocol: unified seam, adapter capabilities/permissions, minimal context, result and evidence schemas.
3. List automatic vs human-required actions and enforce single-use approvals and human completion gates.
4. Declare observation contracts, unified metrics and quality gates; fail closed on violations.
5. Review the implementation against the Design Checklist.

## References / 参考

- 项目 README: 见仓库根目录
- 作者: h565656445 (GitHub)