# Silicon Strategy Bureau: A Lightweight Multi-Agent Collaboration Architecture for Individual Knowledge Workers

> **USER MANUAL v1.0** · Academic Paper Style
>
> Applicable Version: Silicon Strategy Bureau v2.0
>
> ---
>
> ## Abstract

With the rapid advancement of Large Language Model (LLM) capabilities, Multi-Agent collaboration systems have become an important paradigm for AI applications. However, existing frameworks (LangGraph, CrewAI, AutoGen, etc.) are primarily designed for enterprise-level distributed deployment, with insufficient consideration for individual knowledge worker use cases. This paper proposes the **Silicon Strategy Bureau** — a lightweight Multi-Agent collaboration architecture designed for individual users. The architecture adopts a hybrid Orchestrator-Worker + Router pattern, uses the file system as shared state storage, and implements mechanisms including STATE_BLOCK, task kanban, blocking protocol, and cross-role message formats to deliver a zero-deployment-dependency, cross-session persistent AI team organization solution.

**Keywords**: Multi-Agent Systems · Human-AI Collaboration · LLM · State Management · Knowledge Management · Orchestrator-Worker

---

## 1. Introduction

### 1.1 Problem Background

Individual knowledge workers (such as investors, engineers, and managers) face information processing demands across multiple professional domains: investment analysis, career planning, technical problems, and life decisions. The traditional approach is to switch topics within a single AI conversation, which encounters three core pain points:

1. **Context Pollution**: Investment analysis and technical problems interfere with each other in the same conversation, making it difficult for the AI to maintain domain expertise
2. **State Loss**: After switching conversations, all prior analysis, decisions, and progress are lost — requiring a full re-introduction each time
3. **Knowledge Silos**: Discoveries from different domains cannot be shared with each other — industry insights from the technical domain may be critical to investment decisions, yet information does not flow automatically

### 1.2 Design Goals

The Silicon Strategy Bureau's design goal is to solve the three pain points above **without introducing external infrastructure**:

| Pain Point | Solution |
|------|---------|
| Context Pollution | Role separation — each role runs in an independent conversation with its own knowledge boundary |
| State Loss | STATE_BLOCK + HANDOVER files — structured state snapshots that AI auto-parses upon wake-up |
| Knowledge Silos | CROSS_ROLE_MSG + daily summary board — structured cross-role messages for explicit information flow |

---

## 2. Related Work

### 2.1 Multi-Agent Framework Overview

In recent years, Multi-Agent collaboration frameworks have proliferated. Edison (2026) categorizes existing architectures into 7 paradigms:

| Paradigm | Representative Frameworks | Core Characteristics | Deployment Complexity |
|------|---------|---------|:---:|
| Orchestrator-Worker | Anthropic Agent SDK, Magentic-One | Central orchestration, star topology | Medium |
| Chain / Pipeline | MetaGPT, ChatDev | SOP-driven, sequential execution | Medium |
| Flat / Debate | CAMEL, FREE-MAD | Decentralized, direct Agent communication | High |
| Dynamic / Adaptive | DyLAN, AMAS | Runtime dynamic topology adjustment | High |
| Handoff / Swarm | OpenAI Swarm, Google ADK | Control transfer | Low |
| Platform / Network | OpenAgents Network | P2P heterogeneous network | Very High |
| Consensus / Ensemble | Spotify Ads AI | Parallel voting | High |

### 2.2 Key Paper Influences

| Paper | Direct Contribution to This Architecture |
|------|------------------|
| **AutoGen (Wu et al., COLM 2024)** | Reference pattern for multi-Agent conversation coordination, especially structured message type design |
| **MetaGPT (Hong et al., ICLR 2024)** | SOP-driven collaboration — the steps[] structure of the task kanban is directly derived from this |
| **Anthropic Harness (2025)** | Core inspiration: passes mechanism (skip prevention), Checkpoint, observability-first approach |
| **CAMEL (Li et al., NeurIPS 2023)** | Role-playing Agent communication — theoretical foundation for the four-role persona design |
| **HALO (2025)** | Three-layer architecture stratification — direct reference for ADR-004's role tiering |

### 2.3 Key Differences from Existing Frameworks

| Dimension | Enterprise Frameworks (LangGraph/CrewAI) | Silicon Strategy Bureau |
|------|-------------------------------|-----------|
| **Deployment Dependencies** | Requires Python environment, database (Postgres/Redis) | Zero dependencies — pure file system |
| **Concurrency Model** | Multi-Agent parallel execution | Single-threaded serial (user switches roles on demand) |
| **State Storage** | PostgresSaver / Redis / Event Sourcing | Markdown + JSON files |
| **Communication** | Message queues / HTTP / gRPC | Files + user bridging |
| **Target Users** | Dev teams, enterprises | Individual knowledge workers |
| **Learning Curve** | Requires programming skills | Only requires understanding file organization rules |

---

## 3. System Architecture

### 3.1 Architecture Pattern

The Silicon Strategy Bureau adopts a **hybrid Orchestrator-Worker + Router pattern**:

```
                        User (Orchestrator + Router)
                             │
                ┌────────────┼────────────┐
                │            │            │
           Agent A       Agent B      Agent C       Agent D
        (Investment/    (Career/     (Tech/        (Life/
          Finance)      Business)    Engineering)   Emotion)
        Full Harness   Full Harness  Medium        Lightest
```

- **Orchestrator**: The user — responsible for task decomposition, Agent selection, information relay, and final decision-making
- **Router**: Determines, based on task complexity, whether to route directly to a single Agent or initiate multi-Agent collaboration
- **Worker**: Four domain Agents — each with independent knowledge and file domains

### 3.2 Communication Infrastructure

All inter-Agent communication occurs through three channels:

| Channel | Format | Lifecycle | Purpose |
|------|------|---------|------|
| **HANDOVER File** | HTML + STATE_BLOCK (JSON) | Cross-session persistent | Agent state snapshot + precise wake-up |
| **CROSS_ROLE_MSG** | JSON (embedded in HANDOVER) | Cross-session persistent | Structured inter-Agent messages |
| **User Bridge** | Natural language | Within session | Real-time information relay + decision communication |

### 3.3 State Management

```
STATE_BLOCK (JSON) ─embedded in→ HANDOVER.html
     │
     ├── current_task: Current task ID
     ├── current_step: Current step number
     ├── steps_done: List of completed steps
     ├── blockers: Current blocking items
     ├── next_action: Specific action to take on next activation
     └── context_files: Context files to load
```

Upon wake-up, the AI does not need to read the entire HANDOVER — it directly parses the STATE_BLOCK to locate state, reducing wake-up time from O(n) to O(1).

---

## 4. Core Innovations

### 4.1 File-as-Infrastructure

**Industry Practice**: Multi-Agent systems typically require databases (PostgresSaver), message queues (Redis), RPC frameworks, etc.

**Our Innovation**: The file system itself serves as a unified infrastructure for state storage, message channels, and knowledge base. This brings three benefits:
- **Zero Deployment**: No need to install any database or middleware
- **Naturally Auditable**: Every state change has file history, natively supporting git diff
- **Human-AI Co-readable**: HANDOVER files are human-readable (HTML), STATE_BLOCK is AI-parseable (JSON)

### 4.2 Tiered Harness Constraints

**Industry Practice**: One-size-fits-all — all Agents use the same constraint level.

**Our Innovation**: A three-tier constraint strategy based on role characteristics:
- **Full Constraints** (investment/career roles): Task kanban + Checkpoint + Blocking Protocol + Acceptance Checklist
- **Medium Constraints** (technical roles): Long tasks are constrained, short Q&A is free
- **Lightest Constraints** (life roles): STATE_BLOCK only for wake-up continuity

This design avoids the absurdity of applying factory management workflows to psychological counseling.

### 4.3 Cost-Aware Routing

**Industry Practice**: All requests go through the full Multi-Agent collaboration process.

**Our Innovation**: Based on Anthropic's empirical data (Multi-Agent outperforms single Agent by 90.2% but costs 15x more), a two-level routing system is introduced:

```
Complexity 1-3 (simple query) → Route directly to target Agent (single Agent execution)
Complexity 4-6 (medium analysis) → Orchestrator + 2 Workers
Complexity 7-10 (complex cross-domain) → Full Orchestrator + all Workers
```

This avoids running the full pipeline for simple problems, saving an estimated 60-80% token consumption.

### 4.4 Blocking Protocol

**Industry Practice**: When Agents encounter blocks, they typically "wait for user reply" without structured recording.

**Our Innovation**: Four-level blocking classification + cross-role visibility + standardized resolution workflow:
- **BLOCKED** (awaiting user decision): Written to task kanban, auto-prompted in next conversation
- **STALLED** (awaiting external event): Auto-check time set
- **DEPENDENCY** (awaiting another Agent): Cross-role visible; target Agent knows someone is waiting
- **INFO_NEED** (awaiting supplementary information): Non-blocking but recorded

Blocking is not failure — it is the structured expression of "needing external input to proceed."

### 4.5 Human-in-the-Loop as Core

**Industry Practice**: Human-in-the-Loop is typically an add-on safety check step.

**Our Innovation**: The human (user) is inherently a core node of the system architecture — Orchestrator + Router + message bus + final decision maker. Agents do not directly communicate with each other; all cross-role information must pass through the user. This appears to be a "bottleneck" but is actually **quality control** — ensuring the user maintains full control rather than being led by Agents.

---

## 5. Usage Guide

### 5.1 Quick Start (5 minutes)

**Step 1 — Create directory structure**

```bash
mkdir -p strategy-bureau
mkdir -p invest-strategy/checkpoints
mkdir -p career-strategy/checkpoints
mkdir -p tech-consultant/工作记录
mkdir -p life-consultant/工作记录
mkdir -p deliverables/archive
```

**Step 2 — Initialize core files**

Copy the template files from this project into your workspace:
- `组织架构与通信协议.md` → `strategy-bureau/`
- `技术架构.md` → `strategy-bureau/`
- `阻塞协议.md` → `strategy-bureau/`
- `跨角色消息格式.md` → `strategy-bureau/`

**Step 3 — Create the first Agent's self-portrait**

For each Agent, create a self-portrait file (using Agent C — Technical Consultant as an example):

```markdown
# Technical Engineering Consultant · Self-Portrait

## Identity
You are the Technical Engineering Consultant (Agent C), focused on solving engineering problems.
You run in an independent conversation with your own file domain tech-consultant/.

## Wake-up Protocol
On each activation:
1. Read the STATE_BLOCK in tech-consultant/AI_HANDOVER_STATUS.html
2. Parse current task and step
3. Do not ask "what did we talk about last time" — continue directly

## Closure Protocol
After each task completion:
1. Update STATE_BLOCK
2. Write work log to tech-consultant/工作记录/
3. If there is a reusable solution → record it to strategy-bureau/小贴士集.md
```

**Step 4 — Start using**

```
In Conversation A:
  → Paste Agent A self-portrait → Start investment analysis

In Conversation B:
  → Paste Agent B self-portrait → Start career planning

In Conversation C:
  → Paste Agent C self-portrait → Start technical consultation

Switching roles = Switching conversations = Pasting the corresponding self-portrait
```

### 5.2 Daily Workflow

```
Standard daily start-up process (5 minutes):

1. Check daily summary board — see what each Agent did yesterday
2. Check blockers — review the _blockers array in the task kanban; prioritize resolving BLOCKED items
3. Activate Agents on demand — choose which roles to work with today, paste self-portraits
4. Work — have normal conversations, complete tasks
5. Closure — at the end of each conversation, require the Agent to execute the closure SOP
```

### 5.3 Cross-Role Information Transfer

```
Scenario: Agent C (Technical Consultant) discovers information valuable to Agent B (Career Consultant)

Workflow:
1. Agent C writes findings into their HANDOVER → CROSS_ROLE_MSG
   {
     "from": "AgentC",
     "to": "AgentB",
     "type": "signal",
     "summary": "Technology breakthrough progress in a certain industry"
   }
2. When the user next activates Agent B, Agent B reads the HANDOVER and discovers the message
3. Agent B adjusts analysis based on the message

Key: The user is the bridge for information transfer; no automation dependency
```

### 5.4 Adapt to Your Needs

**Only managing one domain?** Use just 1 Agent + lightest constraints (STATE_BLOCK is sufficient)

**Want to add more domains?** Create a new role directory + self-portrait + choose constraint level based on complexity

**Already have existing AI tools?** The consultant portable protocol allows activating via pasting self-portrait in any AI conversation

---

## 6. Architecture Advantage Analysis

### 6.1 Comparison with Mainstream Approaches

| Dimension | Single Agent Conversation | Silicon Strategy Bureau | Enterprise Framework |
|------|:---:|:---:|:---:|
| **Domain Isolation** | ❌ Context pollution | ✅ Each Agent in independent conversation | ✅ |
| **Cross-Session Memory** | ❌ Reset each time | ✅ STATE_BLOCK precise wake-up | ✅ |
| **Cross-Domain Information Flow** | ❌ None | ✅ CROSS_ROLE_MSG + user bridge | ✅ |
| **Deployment Cost** | None | None (pure files) | High (databases + services) |
| **Learning Curve** | Very low | Low (understand file rules) | High (requires programming) |
| **Cost Control** | Naturally lowest | Cost-aware routing | Typically full execution |
| **Auditability** | None | git natively supported | Requires separate logging system |

### 6.2 Suitable vs. Unsuitable Scenarios

**✅ Suitable Scenarios:**
- Individual knowledge workers managing multiple professional domains
- Need for long-term memory and cross-session state persistence
- Cross-domain information needs to flow but does not require real-time delivery
- Limited budget, unwilling to pay for database/message queue costs

**❌ Unsuitable Scenarios:**
- Need for direct real-time communication between Agents
- Need for parallel Agent execution
- Large-scale collaboration with 10+ Agents
- Need for strict concurrency consistency guarantees

---

## 7. Limitations and Future Work

### 7.1 Known Limitations

1. **User as Single-Point Bottleneck**: When the user is offline, all cross-role communication stops. Mitigation: automated tasks (scheduled briefings, etc.) run independently.
2. **No Concurrency Consistency**: The file system does not support ACID guarantees for concurrent writes. Currently a non-issue with single-threaded serial execution; needs upgrade if parallel Agents are introduced.
3. **CROSS_ROLE_MSG is Non-Automated**: Message delivery still depends on manual user relay; automatic routing not yet implemented.
4. **Lack of Token Cost Tracking**: Cannot track API consumption by role; cost optimization relies on manual estimation.

### 7.2 Future Directions

- **Automated Message Routing**: Monitor CROSS_ROLE_MSG blocks in HANDOVER files and auto-push to target Agents
- **Shared Message Queue**: Introduce a local file queue for asynchronous Agent message reading
- **Parallel Agent Execution**: Upgrade to external state storage (SQLite/BoltDB) to support concurrent multi-Agent work
- **Cost Dashboard**: Track token consumption by role/task

---

## 8. References

1. Wu, Q. et al. (2024). **AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation**. COLM 2024.
2. Hong, S. et al. (2024). **MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework**. ICLR 2024.
3. Li, G. et al. (2023). **CAMEL: Communicative Agents for "Mind" Exploration of Large Language Model Society**. NeurIPS 2023.
4. Chen, W. et al. (2024). **AgentVerse: Facilitating Multi-Agent Collaboration and Exploring Emergent Behaviors**. ICLR 2024.
5. Anthropic. (2025). **Effective Harnesses for Long-Running Agents**. Technical Report.
6. Wang, L. et al. (2025). **HALO: A Hierarchical Architecture for LLM-based Autonomous Agents**.
7. Zhang, Y. et al. (2025). **MAP: Multi-Agent Planning with Structured Roles**. Nature Communications.
8. Liu, J. et al. (2025). **PlanGenLLMs: Evaluating Planning Quality in LLM Agents**. ACL 2025.
9. Edison Research. (2026). **A Comprehensive Survey of Multi-Agent Collaboration Systems**. Technical Report.
10. BiliBili Tech. (2026). **Multi-Agent Architecture Decision Matrix: 7 Patterns for Production**.
11. Prism Space. (2026). **Enterprise Multi-Agent Deployment Guide: Cost, Observability, and State**.

---

## A. Appendix: Task Kanban JSON Schema

```json
{
  "version": "1.0.0",
  "_blockers": [],
  "AgentA": {
    "tasks": [
      {
        "task_id": "task-001",
        "title": "Task Title",
        "description": "Task Description",
        "steps": ["Step 1", "Step 2", "Step 3"],
        "steps_done": [0],
        "passes": false,
        "depends_on": [],
        "checkpoint": "path/to/checkpoint.json"
      }
    ]
  },
  "AgentB": { "tasks": [] },
  "AgentC": { "tasks": [] },
  "AgentD": { "tasks": [] }
}
```

**Agent Permissions**: ✅ modify passes / ✅ modify steps_done / ✅ append blockers / ❌ delete tasks / ❌ add tasks / ❌ modify depends_on

---

## B. Appendix: STATE_BLOCK Template

```json
{
  "role": "AgentA",
  "last_session": "2026-05-21T15:30:00",
  "current_task": "task-001",
  "current_step": 2,
  "steps_done": [0, 1, 2],
  "blockers": [],
  "next_action": "Complete Step 3: Data Collection",
  "context_files": [
    "invest-strategy/checkpoints/task-001-phase1.json"
  ]
}
```

---

> *"The best architecture is the one the team can actually maintain. The Silicon Strategy Bureau's design philosophy: not the most powerful, but the most accessible."*
>
> —— Silicon Strategy Bureau
