# Silicon Strategy Bureau: A Lightweight Multi-Agent Collaboration Architecture for Individual Knowledge Workers

> **USER MANUAL v1.0** · Academic Paper-Style Guide
>
> Applicable Version: Silicon Strategy Bureau v2.0
>
> ---
>
> ## Abstract

As large language model (LLM) capabilities rapidly advance, Multi-Agent collaboration systems have become a significant paradigm in AI applications. However, existing frameworks (LangGraph, CrewAI, AutoGen, etc.) primarily target enterprise-level distributed deployment, with limited consideration for individual knowledge workers' use cases. This paper presents the **Silicon Strategy Bureau** — a lightweight Multi-Agent collaboration architecture for individual users. The architecture adopts an Orchestrator-Worker + Router hybrid pattern, using the filesystem as shared state storage. Through mechanisms including STATE_BLOCK, task boards, blocking protocols, and cross-role message formats, it provides a zero-deployment-dependency, cross-session persistent AI team organization solution.

**Keywords**: Multi-Agent System · Human-AI Collaboration · LLM · State Management · Knowledge Management · Orchestrator-Worker

---

## 1. Introduction

### 1.1 Problem Background

Individual knowledge workers (such as investors, engineers, and managers) face information processing demands across multiple professional domains: investment analysis, career planning, technical issues, and life decisions. The traditional approach of switching topics within a single AI conversation faces three core pain points:

1. **Context Pollution**: Investment analysis and technical issues in the same conversation interfere with each other, making it difficult for AI to maintain domain specialization
2. **State Loss**: After switching sessions, all previous analysis, decisions, and progress are lost — requiring "re-introduction" every time
3. **Knowledge Silos**: Discoveries from different domains cannot flow between each other — a technical industry insight might be critical for investment decisions, but information doesn't automatically propagate

### 1.2 Design Objectives

The Silicon Strategy Bureau's design goal is to address these three pain points **without introducing external infrastructure**:

| Pain Point | Solution |
|------------|----------|
| Context Pollution | Role separation — each role runs in its own conversation with independent knowledge boundaries |
| State Loss | STATE_BLOCK + HANDOVER files — structured state snapshots, auto-parsed on AI wakeup |
| Knowledge Silos | CROSS_ROLE_MSG + daily board — structured cross-role messages, explicit information flow |

---

## 2. Related Work

### 2.1 Multi-Agent Framework Overview

In recent years, Multi-Agent collaboration frameworks have proliferated. Edison (2026) categorizes existing architectures into 7 paradigms:

| Paradigm | Representative Frameworks | Core Characteristics | Deployment Complexity |
|----------|--------------------------|---------------------|:---:|
| Orchestrator-Worker | Anthropic Agent SDK, Magentic-One | Central orchestration, star topology | Medium |
| Chain / Pipeline | MetaGPT, ChatDev | SOP-driven, sequential execution | Medium |
| Flat / Debate | CAMEL, FREE-MAD | Decentralized, direct Agent communication | High |
| Dynamic / Adaptive | DyLAN, AMAS | Runtime dynamic topology adjustment | High |
| Handoff / Swarm | OpenAI Swarm, Google ADK | Control transfer | Low |
| Platform / Network | OpenAgents Network | P2P heterogeneous network | Very High |
| Consensus / Ensemble | Spotify Ads AI | Parallel voting | High |

### 2.2 Key Paper Influences

| Paper | Direct Contribution to This Architecture |
|-------|------------------------------------------|
| **AutoGen (Wu et al., COLM 2024)** | Multi-Agent conversation coordination patterns, especially structured message type design |
| **MetaGPT (Hong et al., ICLR 2024)** | SOP-driven collaboration — task board's steps[] structure directly derived from this |
| **Anthropic Harness (2025)** | Core inspiration: passes mechanism (anti-skip), Checkpoint, observability-first |
| **CAMEL (Li et al., NeurIPS 2023)** | Role-playing Agent communication — theoretical foundation for four-role personification |
| **HALO (2025)** | Three-layer architecture tiering strategy — ADR-004 role tiering directly references this |

### 2.3 Key Differences from Existing Frameworks

| Dimension | Enterprise Frameworks (LangGraph/CrewAI) | Silicon Strategy Bureau |
|-----------|------------------------------------------|------------------------|
| **Deployment Dependency** | Requires Python environment, database (Postgres/Redis) | Zero dependency — pure filesystem |
| **Concurrency Model** | Multi-Agent parallel execution | Single-threaded serial (user switches roles on demand) |
| **State Storage** | PostgresSaver / Redis / Event Sourcing | Markdown + JSON files |
| **Communication** | Message queue / HTTP / gRPC | Files + user bridging |
| **Target User** | Development teams, enterprises | Individual knowledge workers |
| **Learning Curve** | Requires programming skills | Only needs to understand file organization rules |

---

## 3. System Architecture

### 3.1 Architecture Pattern

The Silicon Strategy Bureau adopts an **Orchestrator-Worker + Router hybrid pattern**:

```
                        User (Orchestrator + Router)
                             │
                ┌────────────┼────────────┐
                │            │            │
           Agent A      Agent B      Agent C      Agent D
         (Investment)  (Career)     (Tech/Eng)   (Life/Wellness)
         Full Harness  Full Harness  Medium       Lightest
```

- **Orchestrator**: The user — responsible for task decomposition, Agent selection, information relay, and final decisions
- **Router**: Routes tasks based on complexity — direct-to-single-Agent or multi-Agent collaboration
- **Worker**: Four domain Agents — each with independent knowledge domain and file domain

### 3.2 Communication Infrastructure

All inter-Agent communication occurs through three channels:

| Channel | Format | Lifecycle | Purpose |
|---------|--------|-----------|---------|
| **HANDOVER File** | HTML + STATE_BLOCK (JSON) | Cross-session persistent | Agent state snapshot + precise wakeup |
| **CROSS_ROLE_MSG** | JSON (embedded in HANDOVER) | Cross-session persistent | Structured inter-Agent messages |
| **User Bridge** | Natural language | Within session | Real-time information relay + decision communication |

### 3.3 State Management

```
STATE_BLOCK (JSON) ─embedded in→ HANDOVER.html
     │
     ├── current_task: Current task ID
     ├── current_step: Current step number
     ├── steps_done: Completed step list
     ├── blockers: Current blocking list
     ├── next_action: Specific action on next wakeup
     └── context_files: Context files to load
```

On wakeup, the AI doesn't need to read the full HANDOVER — instead, it directly parses STATE_BLOCK to locate state, reducing wakeup time from O(n) to O(1).

---

## 4. Core Innovations

### 4.1 File-as-Infrastructure

**Industry Practice**: Multi-Agent systems typically require databases (PostgresSaver), message queues (Redis), RPC frameworks, etc.

**Our Innovation**: Treat the filesystem itself as a three-in-one infrastructure combining state storage, message channel, and knowledge base. This brings three benefits:
- **Zero Deployment**: No need to install any databases or middleware
- **Naturally Auditable**: Every state change has file history, natively supporting git diff
- **Human-Machine Co-readable**: HANDOVER files human-readable (HTML), STATE_BLOCK AI-parseable (JSON)

### 4.2 Tiered Harness Constraints

**Industry Practice**: One-size-fits-all — all Agents use the same constraint level.

**Our Innovation**: A three-tier constraint strategy based on role characteristics:
- **Full Constraints** (Investment/Career roles): Task board + Checkpoint + Blocking Protocol + Acceptance Checklist
- **Medium Constraints** (Technical roles): Long tasks constrained, short Q&A free
- **Lightest Constraints** (Life roles): Only STATE_BLOCK for wakeup continuation

This design avoids the absurdity of "managing psychological counseling with factory management processes."

### 4.3 Cost-Aware Routing

**Industry Practice**: All requests go through the full Multi-Agent collaboration process.

**Our Innovation**: Based on Anthropic's empirical data (Multi-Agent 90.2% better performance but 15x cost), introduces two-level routing:

```
Complexity 1-3 (Simple query) → Direct to target Agent (single Agent execution)
Complexity 4-6 (Medium analysis) → Orchestrator + 2 Workers
Complexity 7-10 (Complex cross-domain) → Full Orchestrator + all Workers
```

This avoids full-process execution for simple problems, estimated to save 60-80% token consumption.

### 4.4 Blocking Protocol

**Industry Practice**: When Agents encounter blockages, they typically "wait for user response" without structured recording.

**Our Innovation**: Four-level blocking classification + cross-role visibility + standardized resolution process:
- **BLOCKED** (Awaiting user decision): Written to task board, auto-prompted in next conversation
- **STALLED** (Awaiting external event): Automatic check time set
- **DEPENDENCY** (Awaiting other Agent): Cross-role visible, target Agent aware someone is waiting
- **INFO_NEED** (Need supplementary information): Non-blocking but recorded

Blocking is not failure — blocking is the structured expression of "external input needed to continue."

### 4.5 Human-in-the-Loop as Core

**Industry Practice**: Human-in-the-Loop is typically an additional safety check step.

**Our Innovation**: The human (user) is inherently the core node of the system architecture — Orchestrator + Router + Message Bus + Final Decision Maker. Agents do not communicate directly with each other; all cross-role information must pass through the user. This appears to be a "bottleneck" but is actually **quality control** — ensuring the user always maintains control rather than being led by Agents.

---

## 5. Usage Guide

### 5.1 Quick Start (5 minutes)

**Step 1 — Create Directory Structure**

```bash
mkdir -p strategy-bureau
mkdir -p invest-strategy/checkpoints
mkdir -p career-strategy/checkpoints
mkdir -p tech-consultant/work-logs
mkdir -p life-consultant/work-logs
mkdir -p deliverables/archive
```

**Step 2 — Initialize Core Files**

Copy the template files from this project into your workspace:
- `org-and-communication.md` → `strategy-bureau/`
- `tech-architecture.md` → `strategy-bureau/`
- `blocking-protocol.md` → `strategy-bureau/`
- `cross-role-messages.md` → `strategy-bureau/`

**Step 3 — Create the First Agent's Self-Portrait**

Create a self-portrait file for each Agent (using Agent C, Technical Advisor, as an example):

```markdown
# Technical Engineering Advisor · Self-Portrait

## Identity
You are the Technical Engineering Advisor (Agent C), focused on solving engineering problems.
You run in an independent conversation with your own file domain at tech-consultant/.

## Wakeup Protocol
Each time you are activated:
1. Read the STATE_BLOCK in tech-consultant/AI_HANDOVER_STATUS.html
2. Parse current task and step
3. Do not ask "what were we talking about last time" — continue directly

## Wrap-up Protocol
After each task completion:
1. Update STATE_BLOCK
2. Write work log to tech-consultant/work-logs/
3. If there are reusable solutions → record in strategy-bureau/tips-collection.md
```

**Step 4 — Start Using**

```
In Conversation A:
  → Paste Agent A self-portrait → Start investment analysis

In Conversation B:
  → Paste Agent B self-portrait → Start career planning

In Conversation C:
  → Paste Agent C self-portrait → Start technical consulting

Switch role = Switch conversation = Paste corresponding self-portrait
```

### 5.2 Daily Workflow

```
Standard daily startup flow (5 minutes):

1. Check daily board — see what each Agent did yesterday
2. Check blockages — review _blockers array in task board, prioritize resolving BLOCKED
3. Activate Agents on demand — select roles needing work today, paste self-portrait
4. Work — normal conversation, complete tasks
5. Wrap-up — at end of each conversation, ask Agent to execute wrap-up SOP
```

### 5.3 Cross-Role Information Transfer

```
Scenario: Agent C (Tech Advisor) discovers information valuable to Agent B (Career Advisor)

Process:
1. Agent C writes discovery into its HANDOVER → CROSS_ROLE_MSG
   {
     "from": "AgentC",
     "to": "AgentB",
     "type": "signal",
     "summary": "Industry X technology breakthrough progress"
   }
2. User (you) next activates Agent B — Agent B reads HANDOVER and discovers message
3. Agent B adjusts analysis based on message

Key: User is the bridge for information transfer, no automation dependency
```

### 5.4 Adapt to Your Needs

**Managing only one domain?** Use just 1 Agent + lightest constraints (STATE_BLOCK is sufficient)

**Want to add more domains?** Create new role directory + self-portrait + choose constraint level based on complexity

**Already have AI tools?** The Advisor Portability Protocol allows activation in any AI conversation by pasting the self-portrait

---

## 6. Architecture Strengths Analysis

### 6.1 Comparison with Mainstream Solutions

| Dimension | Single Agent Conversation | Silicon Strategy Bureau | Enterprise Frameworks |
|-----------|:---:|:---:|:---:|
| **Domain Isolation** | ❌ Context pollution | ✅ Each Agent independent conversation | ✅ |
| **Cross-Session Memory** | ❌ Start over each time | ✅ STATE_BLOCK precise wakeup | ✅ |
| **Cross-Domain Information Flow** | ❌ None | ✅ CROSS_ROLE_MSG + User bridge | ✅ |
| **Deployment Cost** | None | None (pure files) | High (DB + services) |
| **Learning Curve** | Very Low | Low (understand file rules) | High (requires programming) |
| **Cost Control** | Naturally lowest | Cost-aware routing | Usually full execution |
| **Auditability** | None | Native git support | Requires separate logging system |

### 6.2 Suitable vs Unsuitable Scenarios

**✅ Suitable Scenarios:**
- Individual knowledge workers managing multiple professional domains
- Need for long-term memory and cross-session state persistence
- Cross-domain information needs flow but doesn't require real-time
- Limited budget, don't want to pay for database/message queue costs

**❌ Unsuitable Scenarios:**
- Need for direct real-time Agent-to-Agent communication
- Need for parallel Agent execution
- Large-scale collaboration with 10+ Agents
- Need for strict concurrent consistency guarantees

---

## 7. Limitations and Future Work

### 7.1 Known Limitations

1. **User as Single-Point Bottleneck**: When the user is offline, all cross-role communication stops. Mitigation: automated tasks (scheduled briefings, etc.) run independently.
2. **No Concurrent Consistency**: Filesystem doesn't support ACID guarantees for concurrent writes. Currently not an issue with single-threaded serial execution; needs upgrade when introducing parallel Agents.
3. **CROSS_ROLE_MSG Not Automated**: Message delivery still depends on user manual relay; automatic routing not yet implemented.
4. **Lack of Token Cost Tracking**: Cannot track API consumption by role; cost optimization relies on manual estimation.

### 7.2 Future Directions

- **Automated Message Routing**: Monitor HANDOVER files for CROSS_ROLE_MSG blocks, auto-push to target Agents
- **Shared Message Queue**: Introduce local file queue, Agents can asynchronously read pending messages
- **Parallel Agent Execution**: Upgrade to external state storage (SQLite/BoltDB), support multiple Agents working simultaneously
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

## A. Appendix: Task Board JSON Schema

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

**Agent Permissions**: ✅ Modify passes / ✅ Modify steps_done / ✅ Append blockers / ❌ Delete tasks / ❌ Add tasks / ❌ Modify depends_on

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
  "next_action": "Complete data collection for Step 3",
  "context_files": [
    "invest-strategy/checkpoints/task-001-phase1.json"
  ]
}
```

---

> *"The best architecture is the one your team can actually maintain. The Silicon Strategy Bureau's design philosophy: not the most powerful, but the most accessible."*
>
> — Silicon Strategy Bureau
