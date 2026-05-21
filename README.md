# 🏛️ Silicon Strategy Bureau

> 💡 **WorkBuddy + DeepSeek V4 Pro can be remarkably intelligent.** Choose the right approach, and AI can assist you deeply for extended periods — not to replace your thinking, but to give you a practical multi-agent collaboration framework. Zero dependencies, pure file-driven, plug-and-play.

> A lightweight Multi-Agent collaboration architecture for individual knowledge workers — not a code framework, but a set of design patterns, communication protocols, and workflow specifications.

[![License](https://img.shields.io/badge/license-Custom-green.svg)](./LICENSE)
[![Version](https://img.shields.io/badge/version-v2.0-brightgreen.svg)]()
[![Status](https://img.shields.io/badge/status-Active-success.svg)]()
[![Platform](https://img.shields.io/badge/platform-GitHub-black.svg)](https://github.com/YaoXiang-Jin/silicon-strategy-bureau)
[![Mirror](https://img.shields.io/badge/mirror-Gitee-red.svg)](https://gitee.com/yx--jin/silicon-strategy-bureau)

---

## 📖 What Is This?

**Silicon Strategy Bureau** is a **human-AI collaborative Multi-Agent organizational architecture**. It organizes multiple AI agents into a small team with clear division of labor and asynchronous collaboration, managed by a human Orchestrator for task distribution, state management, and knowledge accumulation.

```
User (Orchestrator + Physical Executor)
  ├── Agent A (Investment / Finance)
  ├── Agent B (Career / Business)
  ├── Agent C (Technology / Engineering)
  └── Agent D (Life / Wellness)
```

### Core Philosophy

- **Humans are augmented, not replaced**: AI handles analysis, planning, frameworks; humans handle decisions, execution, feedback
- **File as Infrastructure**: Zero databases, zero message queues, zero external dependencies — pure filesystem-driven
- **Visible, Traceable, Verifiable**: Task boards + Checkpoints + Blocking Protocol based on Anthropic Harness paper

---

## ✨ Why Silicon Strategy Bureau?

| Pain Point | Traditional Approach | Silicon Strategy Bureau |
|------------|---------------------|------------------------|
| Context pollution from multi-domain switching | One conversation for everything | 4 Agents, each in its own domain |
| State loss after session switch | Re-introduce yourself every time | STATE_BLOCK precise wakeup, O(1) location |
| Information silos across domains | Manual copy-paste | CROSS_ROLE_MSG structured cross-role messaging |
| Enterprise frameworks too heavy | Requires Python + DB + Message Queue | Zero dependencies, pure filesystem |

---

## 🏗️ Core Innovations

| Innovation | Description |
|------------|-------------|
| **File-as-Infrastructure** | Filesystem = State Store + Message Channel + Knowledge Base, three-in-one |
| **Tiered Harness Constraints** | Full constraints for analytical roles, medium for engineering, lightest for lifestyle |
| **Cost-Aware Routing** | Simple tasks → single agent (60-80% token savings), complex tasks → full collaboration |
| **Four-Level Blocking Protocol** | BLOCKED / STALLED / DEPENDENCY / INFO_NEED, blocking visualization |
| **Human-in-the-Loop as Core** | User is the Orchestrator, agents do not communicate directly |

---

## 📚 File Navigation

| File | Content |
|------|---------|
| 📄 `README.md` | This file — Project overview |
| 📘 `USER_MANUAL.md` | **Academic paper-style user manual** — Must-read! 8 chapters + 11 references |
| 🏗️ `tech-architecture.md` | 9 ADRs + 7 architecture paradigm comparisons + benchmarking |
| 🏛️ `org-and-communication.md` | Role definitions, information flow design, Harness workflow SOP |
| 🚧 `blocking-protocol.md` | 4-level blocking + lifecycle + resolution SOP |
| 💬 `cross-role-messages.md` | CROSS_ROLE_MSG JSON Schema |

---

## 🚀 Quick Start (5 minutes)

```bash
# 1. Create directory structure
mkdir -p strategy-bureau
mkdir -p invest-strategy/checkpoints career-strategy/checkpoints
mkdir -p tech-consultant/work-logs life-consultant/work-logs

# 2. Copy template files
cp org-and-communication.md strategy-bureau/
cp blocking-protocol.md strategy-bureau/
cp cross-role-messages.md strategy-bureau/

# 3. Create self-portrait for each Agent (Markdown)
# 4. Paste self-portrait in AI conversation → Activate Agent
# 5. Start collaborating!
```

Detailed steps in [`USER_MANUAL.md`](./USER_MANUAL.md) Chapter 5.

---

## 🆚 Comparison with Major Frameworks

| Dimension | Silicon Strategy Bureau | LangGraph | CrewAI | AutoGen | OpenAI Swarm |
|-----------|:---:|:---:|:---:|:---:|:---:|
| Orchestration | **Orch+Router** | Directed Graph | Sequential | GroupChat | Handoff |
| State Storage | **Filesystem** | Postgres | Flow-based | Event | Context Var |
| Deployment | **Zero** | Python+DB | Python | Python | Python |
| Learning Curve | **Low** | High | Medium | Medium | Low |
| Target Scale | Individual | Enterprise | Team | Team | Individual |

---

## 📖 Design Philosophy

> *"The best architecture is the one your team can actually maintain. Not the most powerful, but the most accessible."*
>
> *"Files are state. Files are communication. When your AI team can be audited with git diff, you've already won."*

**Nine Architectural Decision Records (ADRs)** fully document the context and consequences of each technical choice. See [`tech-architecture.md`](./tech-architecture.md).

---

## 🔬 Academic Foundation

This architecture draws from 9 core papers:

- **AutoGen** (COLM 2024) — Multi-Agent conversation coordination
- **MetaGPT** (ICLR 2024) — SOP-driven collaboration
- **CAMEL** (NeurIPS 2023) — Role-playing communication
- **HALO** (2025) — Three-layer architecture + dynamic roles
- **MAP** (Nature Comm. 2025) — Structured planning
- **Anthropic Harness** (2025) — Orchestrator-Worker empirical validation
- More in [`USER_MANUAL.md`](./USER_MANUAL.md) Chapter 8

---

## 📄 License

**Silicon Strategy Bureau Public License v1.0** — Custom License

| Use Case | Permission |
|----------|-----------|
| Personal study/research | ✅ Free |
| Non-profit education | ✅ Free |
| Open source contribution | ✅ Free |
| Enterprise internal use | ⚠️ Commercial license required |
| Commercial product integration | ⚠️ Commercial license required |
| Commercial distribution/service | ⚠️ Commercial license required |

> Copyright &copy; 2026 [YaoXiang-Jin](https://github.com/YaoXiang-Jin) — All rights reserved. Commercial use: yaoxjin@126.com.

---

## 🌟 Star History

If you find this project helpful, please give it a ⭐ Star!

---

> Built with ❤️ by [YaoXiang-Jin](https://github.com/YaoXiang-Jin) | Feedback and suggestions welcome via Issues

> Also available on [Gitee](https://gitee.com/yx--jin/silicon-strategy-bureau) for Chinese-speaking developers
