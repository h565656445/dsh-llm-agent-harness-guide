# dsh-llm-agent-harness-guide

<!-- DeepSeek Harness 衍生声明 -->
> **DeepSeek Harness 个人适配声明（Personal Adaptation Notice）**
>
> 本项目是 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的**个人适配产物（personal adaptation）**，**并非 DeepSeek Harness 官方文件（not an official DeepSeek Harness file）**，随附功能、使用说明与个人产物（bundled with features, documentation, and personal artifacts），可与 DeepSeek Harness 搭配使用，也可独立使用。
>
> This project is a **personal adaptation** for DeepSeek Harness, and is **NOT an official DeepSeek Harness file**, bundled with features, documentation, and personal artifacts. It can be used alongside DeepSeek Harness or standalone.

**作者 / Author**: [h565656445](https://github.com/h565656445)

**合作 / Collaboration**: 如有项目可以一起合作，欢迎联系。微信：`wohaishihenshuaide`。If you have projects, let's collaborate. WeChat: `wohaishihenshuaide`.


---

## 用途 / What this is for

LLM Agent 控制平面设计指南：控制循环、Worker 协议、审批边界与可观测性的设计方法论。

LLM agent control-plane design guide: control loops, worker protocol, approval boundaries and observability.

---
## LLM Agent Control-Plane Design Guide / LLM Agent 控制平面设计指南

本项目是基于 Hermes Harness 源架构（`core/` 与 `specs/Agent-OS-*`）综合编写的 LLM Agent 控制平面设计指南（纯文档仓库）。内容覆盖：控制平面的定位与控制循环（事件驱动、有界执行）、Worker 协议（Prepare/Accept/Inspect、能力与权限适配器、最小上下文、结构化结果与证据哈希）、审批边界（自动 vs 必须人工、单次使用审批、人工完成门）、以及可观测性（观测合同、统一指标、质量门、失败关闭）。不包含任何源码。

This repository is a bilingual LLM Agent control-plane design guide synthesized from the Hermes Harness source architecture (`core/` and `specs/Agent-OS-*`), pure documentation. It covers: control-plane positioning and the control loop (event-driven, bounded execution); the worker protocol (Prepare/Accept/Inspect, capability and permission adapters, minimal context, structured results with evidence hashes); approval boundaries (automatic vs human-required, single-use approvals, human completion gates); and observability (observation contracts, unified metrics, quality gates, fail closed). No source code is included.

## Features / 功能

- **控制循环**：事件驱动状态机、有界 Loop（两次执行、一次修正）、受监督暂停点
  Control loop: event-driven state machine, bounded loop (two runs, one fix), supervised pause point
- **Worker 协议**：Prepare/Accept/Inspect 统一 seam；能力与权限由适配器声明；Worker 只读最小上下文
  Worker protocol: Prepare/Accept/Inspect unified seam; capabilities and permissions declared by adapters; workers read minimal context only
- **审批边界**：发布/删除/付款/账号/规则修改必须人工；审批只解除合同中列出的动作
  Approval boundaries: publish/delete/payment/account/rule changes require humans; approval only lifts actions listed in the contract
- **可观测性**：观测合同 + 五维指标 + 质量门；预算超限或质量不合格立即失败关闭
  Observability: observation contracts, five metric dimensions, quality gates; budget or quality violations fail closed
- **恢复与审计**：哈希链 Ledger 为事实源，Snapshot 只是缓存；损坏只暴露可信前缀
  Recovery and audit: hash-chain ledger as source of truth, snapshots as caches; corruption exposes only the trusted prefix
- **双语**：中文与英文对照编写，可直接阅读，无运行依赖
  Bilingual: Chinese and English side by side; readable as-is, no runtime dependencies

## What's inside / 目录结构

```
dsh-llm-agent-harness-guide/
├── README.md                         # 双语说明 + DSH 衍生声明
├── LICENSE                           # MIT
├── docs/
│   └── llm-agent-harness-guide.md    # 双语 LLM Agent 控制平面设计指南（核心文档）
└── .dsh/                             # DeepSeek Harness 衍生包
    ├── preset.yml
    ├── agent.cordis.yml
    ├── README.md
    └── skills/dsh-llm-agent-harness-guide/SKILL.md
```

## Quick start / 快速开始

```powershell
# 1. 阅读设计指南
$repo = "E:\path\to\dsh-llm-agent-harness-guide"
Get-Content (Join-Path $repo "docs\llm-agent-harness-guide.md")

# 2. 用设计检查清单评审你的 Agent 控制平面
Get-Content (Join-Path $repo "docs\llm-agent-harness-guide.md") | Select-String "设计检查清单"

# 3. 安装 DSH 预设（可选）
$dst = Join-Path $env:DSH_HOME ".agent-presets\hermes-llm-agent-harness-guide"
Copy-Item -Recurse -Force (Join-Path $repo ".dsh") $dst
```

## DeepSeek Harness 衍生 / DSH Derivative

本项目附带 DeepSeek Harness 衍生包，位于 `.dsh/` 目录：

- `preset.yml` — Agent 预设元数据
- `agent.cordis.yml` — Cordis 组装（基于 standard 预设，persona 已定制）
- `skills/dsh-llm-agent-harness-guide/SKILL.md` — 项目专属技能（skill）

安装与接入方式见 [`.dsh/README.md`](.dsh/README.md)（双语）。

## License / 许可证

[MIT](LICENSE)

---

---

## 相关项目 / Related Projects

> 这是 DeepSeek Harness 个人适配系列（共 31 个仓库）的完整导航。 / This is the complete navigation for the DeepSeek Harness personal-adaptation series (31 repos).

### Agent OS 内核 / Kernel

[`dsh-agent-os-runtime`](https://github.com/h565656445/dsh-agent-os-runtime) · [`dsh-agent-os-planning`](https://github.com/h565656445/dsh-agent-os-planning) · [`dsh-agent-os-scheduler`](https://github.com/h565656445/dsh-agent-os-scheduler) · [`dsh-agent-os-worker-protocol`](https://github.com/h565656445/dsh-agent-os-worker-protocol) · [`dsh-agent-os-observability`](https://github.com/h565656445/dsh-agent-os-observability) · [`dsh-agent-os-specs`](https://github.com/h565656445/dsh-agent-os-specs)

### Harness 基础设施 / Infrastructure

[`dsh-harness-core`](https://github.com/h565656445/dsh-harness-core) · [`dsh-graph-entry`](https://github.com/h565656445/dsh-graph-entry) · [`dsh-async-job`](https://github.com/h565656445/dsh-async-job) · [`dsh-file-identity`](https://github.com/h565656445/dsh-file-identity) · [`dsh-json-projection`](https://github.com/h565656445/dsh-json-projection) · [`dsh-manual-approval`](https://github.com/h565656445/dsh-manual-approval) · [`dsh-observation-writer`](https://github.com/h565656445/dsh-observation-writer) · [`dsh-provider-control`](https://github.com/h565656445/dsh-provider-control) · [`dsh-schema-negotiator`](https://github.com/h565656445/dsh-schema-negotiator) · [`dsh-upgrade-governance`](https://github.com/h565656445/dsh-upgrade-governance)

### 规格与文档 / Specs & Docs

[`dsh-harness-specs`](https://github.com/h565656445/dsh-harness-specs) · [`dsh-novel-specs`](https://github.com/h565656445/dsh-novel-specs) · [`dsh-architecture-guide`](https://github.com/h565656445/dsh-architecture-guide) · [`dsh-powershell-patterns`](https://github.com/h565656445/dsh-powershell-patterns) · [`dsh-json-schema-driven-dev`](https://github.com/h565656445/dsh-json-schema-driven-dev) · **`dsh-llm-agent-harness-guide`（本仓库 / this repo）**

### 适配器 / Adapters

[`dsh-short-story-engine`](https://github.com/h565656445/dsh-short-story-engine) · [`dsh-tutorial-video-state-machine`](https://github.com/h565656445/dsh-tutorial-video-state-machine) · [`dsh-governance-kernel`](https://github.com/h565656445/dsh-governance-kernel) · [`dsh-sports-pipeline`](https://github.com/h565656445/dsh-sports-pipeline) · [`dsh-motion-grammar`](https://github.com/h565656445/dsh-motion-grammar)

### DSH 总集成 / Integration

[`dsh-integration`](https://github.com/h565656445/dsh-integration) · [`dsh-presets-pack`](https://github.com/h565656445/dsh-presets-pack) · [`dsh-skills-pack`](https://github.com/h565656445/dsh-skills-pack) · [`dsh-starter-kit`](https://github.com/h565656445/dsh-starter-kit)

