# 🏗️ Silicon Strategy Bureau · Technical Architecture Document

> **Purpose**: Records the technical decisions of the workflow system — file format choices, JSON Schema versioning strategy, architectural pattern selection, etc.
>
> **Maintained by**: All members (whoever makes a technical decision documents it)
>
> Version: v2.0 · 2026-05-22 | Based on Multi-Agent Collaboration paper research + Architecture Alignment Analysis

---

## 1. Key Architecture Decision Records (ADR)

### ADR-001: Choose JSON as the Structured State Carrier

**Status**: Accepted

**Context**:
The Anthropic Harness paper uses `task.json` as the structured communication format between agents. Previously, the Silicon Strategy Bureau used HTML/Markdown prose-style files. A parseable state format is needed to support "precision wake-up".

**Decision**:
STATE_BLOCK, Task Board, Checkpoint, Blocking Protocol, and Delivery Acceptance Checklist all use JSON format. HANDOVER files and daily summary boards remain in HTML (human readability first).

**Consequences**:
- ✅ AI can directly parse STATE_BLOCK to locate state without reading the entire document
- ✅ Version changes can be validated via JSON Schema
- ⚠️ JSON is less human-readable, so STATE_BLOCK is embedded within HTML HANDOVER files
- ⚠️ Need to maintain backward compatibility of JSON Schema

---

### ADR-002: STATE_BLOCK Embedded in HTML HANDOVER Files (Not Standalone JSON)

**Status**: Accepted

**Context**:
Need to simultaneously satisfy "AI fast state parsing" and "human-readable context". If STATE_BLOCK were a standalone .json file, humans would need an extra step to view it.

**Decision**:
In existing HANDOVER HTML files, wrap JSON blocks with `---STATE_BLOCK---` and `---END_STATE---` markers. AI extracts the JSON block via regex matching on wake-up.

**Consequences**:
- ✅ Both humans and AI can get required information from the same file
- ✅ No increase in file count
- ⚠️ If JSON block is malformed, parse failure requires fallback to full-text reading
- ⚠️ STATE_BLOCK should not exceed 20 lines (to prevent bloat)

---

### ADR-003: Task Board Immutability (passes Mechanism)

**Status**: Accepted

**Context**:
Key constraint from the Harness paper: coding agents can only modify the `passes` field and cannot delete task entries. This prevents agents from "skipping hard steps and declaring completion".

**Decision**:
Agent operation permissions on Task Board JSON: ✅ Modify passes / ✅ Modify steps_done / ✅ Append blockers / ❌ Delete tasks / ❌ Add new tasks / ❌ Modify depends_on. New tasks require user approval.

**Consequences**:
- ✅ Prevents Overreaching (skipping steps) and Premature Completion (declaring done early)
- ⚠️ Task Board may bloat; completed tasks need periodic archiving

---

### ADR-004: Tiered Harness by Role (Not One-Size-Fits-All)

**Status**: Accepted

**Context**:
Different roles have significantly different task characteristics — analysis-intensive roles (e.g., investment analysis chain) are suitable for full Harness workflows, while short Q&A roles (e.g., technical engineering consultant) and privacy-sensitive roles (e.g., life consultant) are not suitable for full constraints.

**Decision**:
Adopt a three-tier strategy:
- **Full Harness**: Analysis-intensive roles (Task Board + Checkpoint + Blocking Protocol + Acceptance Checklist)
- **Medium Constraints**: Engineering roles (Task Board for long tasks, no constraints for short Q&A)
- **Lightest Constraints**: Life support roles (STATE_BLOCK only for session continuity)

**Consequences**:
- ✅ Does not stifle quick responses and privacy boundaries with framework constraints
- ⚠️ Role constraints are not uniform; differences must be clearly documented in the technical architecture

---

### ADR-005: Position as Orchestrator-Worker + Router Hybrid Pattern

**Status**: Accepted (2026-05-22)

**Context**:
Based on the comprehensive academic survey of Multi-Agent collaboration systems (Edison 2026 panoramic research + Bilin 4 architecture patterns + Prism Space enterprise deployment guide), aligning the Silicon Strategy Bureau's architecture pattern with the 7 industry-recognized paradigms.

**Decision**:
Silicon Strategy Bureau current architecture = **Orchestrator-Worker (user as central orchestrator) + Router (task distribution) + partial Handoff (state transfer between roles via HANDOVER files)**.

```
User (Orchestrator)
  ├── Agent A (Investment/Finance) ← Star-type Worker
  ├── Agent B (Career/Business) ← Star-type Worker
  ├── Agent C (Technology/Engineering) ← Star-type Worker
  └── Agent D (Life/Emotional) ← Star-type Worker (lightweight)
```

Comparison with mainstream frameworks:

| Dimension | Silicon Strategy Bureau | LangGraph | CrewAI | AutoGen/AG2 | OpenAI Swarm |
|------|-----------|-----------|--------|-------------|-------------|
| Orchestration Pattern | Orchestrator + Router | Directed Graph + Conditional Edges | Roles + Sequential/Hierarchical | GroupChat | Handoff |
| State Persistence | File System (HANDOVER + STATE_BLOCK) | PostgresSaver Checkpoint | Flow-based | Event sourcing | Context Variables |
| Model Lock-in | None (user assigns models) | None | None | None | OpenAI only |
| Agent Count | 4 (+ user as coordinator) | Unlimited | By role definition | GroupChat members | By handoff chain |
| Communication | User relay + HANDOVER files | StateGraph shared | Commander dispatch | Broadcast messages | Control transfer |

**Consequences**:
- ✅ 4 agents is within the optimal range (research suggests ≤5), coordination tax is manageable
- ✅ Orchestrator pattern covers 70% of production scenarios
- ⚠️ User as sole orchestrator is a single point of bottleneck — tasks cannot be routed when user is offline. Mitigation: automated tasks run independently
- ⚠️ File system state management lacks concurrent consistency guarantees (but current single-thread execution makes this a non-issue)

---

### ADR-006: Introduce Structured Cross-Role Message Format (Replacing Pure Free-Text Relay)

**Status**: Accepted (2026-05-22)

**Context**:
Current cross-role communication relies on user verbal relay + HANDOVER files, lacking a structured format. Research indicates: Magentic-One uses 4 message types (RequestReply/Broadcast/Reset/Deactivate), A2A protocol uses JSON-RPC 2.0 + 6 task states, OpenAI Agents SDK uses Handoff + Context Variables.

**Decision**:
Add `---CROSS_ROLE_MSG---` blocks in existing HANDOVER files, using a simplified message format:

```json
{
  "msg_id": "msg-YYYYMMDD-NNN",
  "from": "AgentA",
  "to": "AgentB",
  "type": "signal",
  "priority": "info|attention|urgent",
  "summary": "One-sentence summary",
  "body": "Detailed content...",
  "refs": ["path/to/related/file.json"],
  "expects_reply": false
}
```

Message type mapping:
| type | Purpose | A2A Equivalent |
|------|------|---------|
| `task` | Task delegation | Task state machine |
| `signal` | Cross-role signal notification | One-way push |
| `query` | Information query request | RequestReply |
| `handoff` | State transfer | Handoff control transfer |

**Consequences**:
- ✅ Eliminates the "verbal relay information loss" problem — messages have unique IDs and are traceable
- ✅ Lays groundwork for future automated cross-role communication
- ⚠️ Increases HANDOVER file complexity; this block can be omitted for simple scenarios
- ⚠️ Currently still requires user to manually relay messages (acting as message bus) — shared message queue can be considered in the long term

---

### ADR-007: Introduce Cost-Aware Task Routing

**Status**: Accepted (2026-05-22)

**Context**:
Research reveals the core constraint of Multi-Agent systems: token consumption is 3-15x that of single Agent. Prism Space guide proposes "task complexity scoring + degradation routing" strategy: complexity < 5 goes directly to single Agent. Anthropic data: Multi-Agent performance improvement of 90.2% but cost increase of 15x.

**Decision**:
Introduce **two-tier task routing**:

```
User Request
  │
  ├── Simple (information query / single-role task)
  │     → Route directly to one Agent, no multi-role collaboration
  │
  └── Complex (cross-domain analysis / multi-role collaboration)
        → Orchestrator decomposition → Worker parallel/sequential → Summary
```

Complexity judgment criteria:
| Complexity | Criteria | Routing Strategy |
|--------|---------|---------|
| 1-3 (Simple) | Single domain, no dependencies, known-answer pattern | Direct to target Agent |
| 4-6 (Medium) | Cross 2 domains, with dependencies, requires analysis | Orchestrator + 2 Workers |
| 7-10 (Complex) | Cross 3+ domains, multiple dependencies, multi-round verification | Full Orchestrator + All Workers |

**Consequences**:
- ✅ Simple queries no longer go through the full workflow, saving 60-80% tokens
- ✅ Retains full collaboration capability for complex tasks
- ⚠️ Complexity judgment itself has inference overhead (but far less than unnecessary collaboration overhead)

---

### ADR-008: Observability Strategy — Infrastructure More Important Than Agent Count

**Status**: Accepted (2026-05-22)

**Context**:
Research consistently emphasizes "observability before complexity" — before increasing Agent count, ensure you can see what each Agent is doing, how much it costs, and why it fails. LangGraph uses OpenTelemetry + trace IDs, Deep Agents uses per-task logging.

**Decision**:
Silicon Strategy Bureau observability is implemented in two layers:

1. **Per-session mandatory logging** (zero cost): Record role, task count, output file count, blockers encountered in the daily summary board
2. **On-demand enablement** (manual): Long-running tasks require mandatory Checkpoint writes; blocking events require mandatory blocking protocol logging

No external monitoring system (e.g., OpenTelemetry) is introduced — the team size and task frequency do not require it.

**Consequences**:
- ✅ Minimum cost to achieve basic observability
- ✅ Daily summary board becomes the source of team transparency
- ⚠️ No automated cost tracking (token consumption statistics) — not a current necessity
- ⚠️ If the team grows to 10+ Agents or automated tasks increase significantly, re-evaluation is needed

---

### ADR-009: File System as "Shared State", No External State Store for Now

**Status**: Accepted (2026-05-22)

**Context**:
Production-grade Multi-Agent systems commonly use Redis for shared state (Prism Space guide), LangGraph uses PostgresSaver. However, Silicon Strategy Bureau Agents execute **single-threaded serially** (user switches roles sequentially within the same session), so there is no concurrent write conflict.

**Decision**:
Maintain file system as shared state (HANDOVER + STATE_BLOCK + Checkpoint + Task Board JSON), without introducing Redis/database. File is state, file is communication.

**Consequences**:
- ✅ Zero deployment dependencies, minimal complexity
- ✅ File is documentation, naturally auditable and traceable
- ⚠️ If "parallel Agent execution" is introduced in the future, upgrade to external state storage is required
- ⚠️ Files may degrade (format errors causing parse failures), fallback mechanism already in place (read full text)

---

## 2. Industry Multi-Agent Framework Benchmark Analysis

> Based on comprehensive analysis of 4 core references (2026-05-22)

### 2.1 Architecture Paradigm Classification (Edison 2026 Panoramic Survey)

| Paradigm | Topology | Representative Frameworks | SSB Compatibility |
|------|------|---------|:---:|
| **Orchestrator-Worker** | Star | LangChain Deep Agents, Magentic-One, Anthropic Claude Agent SDK | ⭐⭐⭐⭐⭐ Current core |
| **Chain / Pipeline** | Chain | MetaGPT, ChatDev, CrewAI Sequential | ⭐⭐ Partial use in sequential analysis |
| **Flat / Debate** | Mesh | CAMEL, FREE-MAD | N/A (no multi-agent debate needs) |
| **Dynamic / Adaptive** | Dynamic | DyLAN, AMAS, REDEREF | ⭐ Academic frontier, future reference |
| **Handoff / Swarm** | Control transfer | OpenAI Swarm, Google ADK | ⭐⭐ HANDOVER files as lightweight Handoff |
| **Platform / Network** | P2P | OpenAgents Network | N/A (not a heterogeneous agent network) |
| **Consensus / Ensemble** | Parallel voting | Spotify Ads AI, AI NeuroSignal | N/A (not a voting scenario) |

### 2.2 Applicability Assessment of Seven Architecture Patterns

Based on Bilin (2026) decision matrix:

| Decision Dimension | SSB Characteristics | Best-Matching Pattern |
|---------|-------------|----------|
| Requires distributed development? | No (centralized orchestration) | Orchestrator |
| Requires parallelization? | Yes (parallel within analysis chain) | Router + Orchestrator |
| Requires multi-hop dialogue? | Yes (cross-session via STATE_BLOCK) | Handoff (lightweight) |
| Requires direct user interaction? | Yes (user directly dialogues with each Agent) | Skills (on-demand loading) |
| Token consumption priority? | High (API cost constraints) | Preference for Skills + simplicity first |

**Conclusion**: Silicon Strategy Bureau is naturally an Orchestrator-Worker + Router hybrid, with Handoff as a lightweight supplement. This is more flexible than a pure single mode, but requires the user to clearly know which mode the current task is in.

### 2.3 Key Paper References

The following papers provide theoretical support for Silicon Strategy Bureau design decisions:

| Paper | Core Contribution | Insight for SSB |
|------|---------|------------------|
| **AutoGen (COLM 2024)** | Multi-Agent dialogue coordination framework | Source of structured cross-role message type design |
| **MetaGPT (ICLR 2024)** | SOP-driven multi-agent collaboration | Source of Task Board steps[] structure |
| **CAMEL (NeurIPS 2023)** | Role-playing Agent communication | Theoretical basis for four-role persona design |
| **AgentVerse (ICLR 2024)** | Dynamic Agent recruitment and collaboration decisions | Future reference: dynamically select participating roles by task |
| **HALO (2025)** | Three-layer architecture + dynamic role generation | Theoretical support for ADR-004 tiered strategy |
| **MAP (Nature Comm. 2025)** | 5-role structured planning | +32% planning enhancement over CoT |
| **PlanGenLLMs (ACL 2025)** | 6-dimension planning quality evaluation | Evaluation framework for Task Board steps[] quality |
| **FREE-MAD (2025)** | Consensus-free debate (single round achieves multi-round effect) | If debate mechanism is introduced in the future, single round is sufficient |
| **Anthropic Research System** | Orchestrator-Worker 15x token measured data | Direct evidence for ADR-007 cost-aware routing |

---

## 3. File Format Architecture

```
workspace/
│
├── [Shared Layer] strategy-bureau/
│   ├── 组织架构.md              ← Team Charter
│   ├── AI通信协议.md            ← Collaboration Rules
│   ├── 任务看板.json            ← ⭐ Structured Task Registry (JSON)
│   ├── 阻塞协议.md              ← ⭐ Blocking Report Protocol (4 levels)
│   ├── Checkpoint模板.json      ← ⭐ Long-flow Phase Checkpoint Template (JSON)
│   ├── 交付验收清单模板.json    ← ⭐ Acceptance Checklist (JSON)
│   ├── 唤醒收口速查卡.md        ← ⭐ Wake-up/Handoff Quick Reference Card
│   ├── 技术架构.md              ← This file (v2.0)
│   ├── 全员日结板.md            ← Daily Team Status Board
│   └── 小贴士集.md              ← Knowledge Repository
│
├── [Role Domains]
│   ├── invest-strategy/         ← Agent A · Investment Analysis
│   │   └── checkpoints/         ← ⭐ Long-flow phase files
│   ├── career-strategy/         ← Agent B · Career Development
│   │   └── checkpoints/         ← ⭐ Long-flow phase files
│   ├── tech-consultant/         ← Agent C · Technical Engineering
│   │   └── checkpoints/         ← ⭐ Long-flow phase files
│   └── life-consultant/         ← Agent D · Life Consulting
│
├── [Deliverables Center] deliverables/
│   ├── *.html                   ← Various analysis reports
│   └── archive/                 ← Historical archives
│
└── .workbuddy/memory/           ← Global memory (long-term + daily logs)
```

---

## 4. JSON Schema Versioning Strategy

| File | Current Version | Versioning Policy |
|------|---------|---------|
| 任务看板.json | 1.0.0 | Semantic versioning: MAJOR.MINOR.PATCH |
| Checkpoint模板.json | 1.0.0 | Template files not versioned; checkpoint instances include template_version |
| 交付验收清单模板.json | 1.0.0 | Same as above |
| 阻塞协议.md | 1.0 | Date-versioned (suffix YYYY-MM-DD) |

**Backward Compatibility Rules**:
- New field added: only requires MINOR upgrade
- Required field modified: MAJOR upgrade
- Field deleted: MAJOR upgrade, with advance notice to all roles

---

## 5. Known Limitations and Pending Decisions

| Item | Status | Notes |
|------|------|------|
| Task Board archiving strategy | Pending | How to archive completed tasks? |
| STATE_BLOCK maximum size | Defined | Recommended not to exceed 20 lines of JSON |
| Cross-role Checkpoint referencing | Pending | Path convention or registry? |
| CROSS_ROLE_MSG automation | To be implemented | Currently still relies on user manual message relay |
| Token cost tracking | Low priority | ADR-008 defers automated cost tracking for now |
| Parallel Agent execution | Long-term | Current architecture is serial; parallelism requires external state storage |

---

## Changelog

| Date | Version | Changes |
|------|------|---------|
| 2026-05-21 | v1.0 | Initial version, included ADR-001 through ADR-004 |
| 2026-05-22 | v2.0 | **Multi-Agent paper research upgrade**: Added ADR-005~009 (architecture pattern alignment, structured messages, cost routing, observability, shared state); Added industry framework benchmark analysis (7 paradigms + 9 core papers); Updated file architecture diagram |

---

> *"Architecture documents are for your future self and future teammates. Today's decisions must be traceable tomorrow."*
