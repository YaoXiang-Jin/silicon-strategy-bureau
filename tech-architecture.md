# 🏗️ Silicon Strategy Bureau · Technical Architecture

> **Purpose**: Record technical decisions of the workflow system itself — file format choices, JSON Schema versioning strategy, architecture pattern selection, etc.
>
> **Maintained By**: All members (whoever makes technical decisions records them)
>
> Version: v2.0 · 2026-05-22 | Based on Multi-Agent Collaboration Research + Architecture Alignment Analysis

---

## I. Key Architecture Decision Records (ADR)

### ADR-001: Choose JSON as Structured State Carrier

**Status**: Accepted

**Context**:
The Anthropic Harness paper uses `task.json` as the structured communication format between agents. The Silicon Strategy Bureau previously used HTML/Markdown prose-style files. A parseable state format is needed to support "precise wakeup."

**Decision**:
STATE_BLOCK, task board, Checkpoint, blocking protocol, and acceptance checklist all use JSON format. HANDOVER files and daily boards remain HTML (human readability priority).

**Consequences**:
- ✅ AI can directly parse STATE_BLOCK to locate state without full-text reading
- ✅ Version changes can be validated via JSON Schema
- ⚠️ JSON is poor for human readability, so STATE_BLOCK is embedded in HTML HANDOVER files
- ⚠️ Need to maintain JSON Schema backward compatibility

---

### ADR-002: Embed STATE_BLOCK in HTML HANDOVER Files (Not as Standalone JSON)

**Status**: Accepted

**Context**:
Need to simultaneously satisfy "AI fast state parsing" and "human-readable context." If STATE_BLOCK were an independent .json file, humans would need extra steps to view it.

**Decision**:
In existing HANDOVER HTML files, wrap JSON with `---STATE_BLOCK---` and `---END_STATE---` markers. AI uses regex matching to extract JSON block on wakeup.

**Consequences**:
- ✅ Both humans and AI can get needed information from the same file
- ✅ No increase in file count
- ⚠️ If JSON block format errors, parsing fails and falls back to full-text reading
- ⚠️ STATE_BLOCK should not exceed 20 lines (to avoid bloat)

---

### ADR-003: Task Board Non-Deletable (Passes Mechanism)

**Status**: Accepted

**Context**:
Key constraint from Harness paper: coding agents can only modify the `passes` field, cannot delete task entries. This prevents agents from "skipping hard steps and declaring completion."

**Decision**:
Task board .json Agent operation permissions: ✅ Modify passes / ✅ Modify steps_done / ✅ Append blockers / ❌ Delete tasks / ❌ Add tasks / ❌ Modify depends_on. Adding new tasks requires user approval.

**Consequences**:
- ✅ Prevents Overreaching (skipping steps) and Premature Completion (declaring done early)
- ⚠️ Task board may bloat; completed tasks need periodic archiving

---

### ADR-004: Tiered Harness Application Across Four Roles (Not One-Size-Fits-All)

**Status**: Accepted

**Context**:
Different roles have significantly different task characteristics — analysis-intensive roles (like investment analysis chain) suit full Harness workflow, while short Q&A (like technical advisor) and privacy-sensitive roles (like lifestyle advisor) don't suit full constraints.

**Decision**:
Adopt a three-tier strategy:
- **Full Harness**: Analysis-intensive roles (task board + Checkpoint + blocking protocol + acceptance checklist)
- **Medium Constraints**: Engineering roles (long tasks use task board, short Q&A unconstrained)
- **Lightest Constraints**: Lifestyle support roles (only STATE_BLOCK for wakeup continuation)

**Consequences**:
- ✅ Won't stifle quick responses and privacy boundaries with framework constraints
- ⚠️ Role constraints are non-uniform; differences must be clearly documented in technical architecture

---

### ADR-005: Position as Orchestrator-Worker + Router Hybrid Pattern

**Status**: Accepted (2026-05-22)

**Context**:
Based on the Multi-Agent Collaboration Systems Academic Survey (Edison 2026 Panoramic Survey + BiliBili 4 Architecture Patterns + Prism Space Enterprise Guide), aligning the Silicon Strategy Bureau's architecture pattern with industry-recognized 7 paradigms.

**Decision**:
Silicon Strategy Bureau current architecture = **Orchestrator-Worker (user as central orchestrator) + Router (task distribution) + partial Handoff (state transfer between roles via HANDOVER files)**.

```
User (Orchestrator)
  ├── Agent A (Investment/Finance) ← Star Worker
  ├── Agent B (Career/Business)    ← Star Worker
  ├── Agent C (Technology/Engineering) ← Star Worker
  └── Agent D (Life/Emotional)     ← Star Worker (lightweight)
```

Compared with mainstream frameworks:

| Dimension | Silicon Strategy Bureau | LangGraph | CrewAI | AutoGen/AG2 | OpenAI Swarm |
|-----------|------------------------|-----------|--------|-------------|--------------|
| Orchestration Pattern | Orchestrator + Router | Directed Graph + Conditional Edges | Roles + Sequential/Hierarchical | GroupChat | Handoff |
| State Persistence | Filesystem (HANDOVER + STATE_BLOCK) | PostgresSaver Checkpoint | Flow-based | Event sourcing | Context Variables |
| Model Lock-in | None (user assigns models) | None | None | None | OpenAI only |
| Agent Count | 4 (+user as coordinator) | Unlimited | By role definition | GroupChat members | By handoff chain |
| Communication | User relay + HANDOVER files | StateGraph shared | Commander dispatch | Broadcast messages | Control transfer |

**Consequences**:
- ✅ 4 Agent count is in optimal range (research suggests ≤5), coordination tax controllable
- ✅ Orchestrator pattern covers 70% of production scenarios
- ⚠️ User as sole orchestrator is a single-point bottleneck — when user is offline, tasks cannot be routed. Mitigation: automated tasks run independently
- ⚠️ Filesystem state management lacks concurrent consistency guarantees (but current single-threaded execution doesn't pose a problem)

---

### ADR-006: Introduce Structured Cross-Role Message Format (Replacing Pure Free-Text Relay)

**Status**: Accepted (2026-05-22)

**Context**:
Current cross-role communication relies on user verbal relay + HANDOVER files, lacking structured format. Research shows: Magentic-One uses 4 message types (RequestReply/Broadcast/Reset/Deactivate), A2A protocol uses JSON-RPC 2.0 + 6 task states, OpenAI Agents SDK uses Handoff + Context Variables.

**Decision**:
Add `---CROSS_ROLE_MSG---` block in existing HANDOVER files, using simplified message format:

```json
{
  "msg_id": "msg-YYYYMMDD-NNN",
  "from": "AgentA",
  "to": "AgentB",
  "type": "signal",
  "priority": "info|attention|urgent",
  "summary": "One-line summary",
  "body": "Detailed content...",
  "refs": ["path/to/related/file.json"],
  "expects_reply": false
}
```

Message type mapping:
| type | Purpose | A2A Equivalent |
|------|---------|----------------|
| `task` | Task delegation | Task state machine |
| `signal` | Cross-role signal notification | One-way push |
| `query` | Information query request | RequestReply |
| `handoff` | State transfer | Handoff control transfer |

**Consequences**:
- ✅ Eliminates "verbal relay information loss" — messages have unique IDs for traceability
- ✅ Lays foundation for future automated cross-role communication
- ⚠️ Increases HANDOVER file complexity; can omit this block for simple scenarios
- ⚠️ Currently still requires user to manually relay messages (acting as message bus) — long-term consider shared message queue

---

### ADR-007: Introduce Cost-Aware Task Routing

**Status**: Accepted (2026-05-22)

**Context**:
Research reveals the core constraint of Multi-Agent systems: token consumption is 3-15x that of single Agent. Prism Space guide proposes "task complexity scoring + degradation routing" strategy: complexity <5 directly handled by single Agent. Anthropic data: Multi-Agent 90.2% better performance than single Agent, but 15x cost.

**Decision**:
Introduce **two-level task routing**:

```
User Request
  │
  ├── Simple (info query / single role task)
  │     → Route directly to one Agent, no multi-role collaboration
  │
  └── Complex (cross-domain analysis / multi-role coordination)
        → Orchestrator decomposition → Worker parallel/sequential → Aggregation
```

Complexity criteria:
| Complexity | Criteria | Routing Strategy |
|------------|----------|------------------|
| 1-3 (Simple) | Single domain, no dependencies, known answer pattern | Direct to target Agent |
| 4-6 (Medium) | Cross 2 domains, with dependencies, requires analysis | Orchestrator + 2 Workers |
| 7-10 (Complex) | Cross 3+ domains, multiple dependencies, multi-round validation | Full Orchestrator + all Workers |

**Consequences**:
- ✅ Simple queries no longer go through full process, saving 60-80% tokens
- ✅ Retains full collaboration capability for complex tasks
- ⚠️ Complexity assessment itself requires reasoning overhead (but far less than unnecessary collaboration overhead)

---

### ADR-008: Observability Strategy — Infrastructure That Precedes Agent Count

**Status**: Accepted (2026-05-22)

**Context**:
Research consistently emphasizes "observability before complexity" — before increasing Agent count, ensure visibility into what each Agent is doing, how much it costs, and why it failed. LangGraph uses OpenTelemetry + trace ID, Deep Agents uses per-task logs.

**Decision**:
Silicon Strategy Bureau observability is implemented in two layers:

1. **Per-Session Mandatory** (zero cost): Record role, task count, output file count, encountered blockages in daily board
2. **On-Demand** (manual): Long-flow tasks must write Checkpoints; when blockages occur, must write blocking protocol records

No external monitoring system (like OpenTelemetry) is introduced — team scale and task frequency don't require it.

**Consequences**:
- ✅ Minimal cost to achieve basic observability
- ✅ Daily board becomes transparent information source for the team
- ⚠️ No automated cost tracking (token consumption statistics) — not currently a priority
- ⚠️ If future development reaches 10+ Agents or automated tasks increase, re-evaluation needed

---

### ADR-009: Filesystem as "Shared State", No External State Storage for Now

**Status**: Accepted (2026-05-22)

**Context**:
Production Multi-Agent systems commonly use Redis for shared state (Prism Space guide), LangGraph uses PostgresSaver. However, Silicon Strategy Bureau Agents execute **single-threaded serially** (user switches roles one at a time in the same session), with no concurrent write conflicts.

**Decision**:
Maintain filesystem as shared state (HANDOVER + STATE_BLOCK + Checkpoint + task board .json), not introduce Redis/databases. Files are state, files are communication.

**Consequences**:
- ✅ Zero deployment dependencies, minimum complexity
- ✅ Files are documents, naturally auditable and traceable
- ⚠️ If "parallel Agent execution" is introduced in the future, must upgrade to external state storage
- ⚠️ Files may corrupt (format errors causing parse failure); fallback mechanism already exists (read full text)

---

## II. Industry Multi-Agent Framework Benchmarking

> Based on comprehensive analysis of 4 core papers (2026-05-22)

### 2.1 Architecture Paradigm Classification (Edison 2026 Panoramic Survey)

| Paradigm | Topology | Representative Frameworks | SSB Match |
|----------|----------|--------------------------|:---:|
| **Orchestrator-Worker** | Star | LangChain Deep Agents, Magentic-One, Anthropic Claude Agent SDK | ⭐⭐⭐⭐⭐ Primary |
| **Chain / Pipeline** | Linear | MetaGPT, ChatDev, CrewAI Sequential | ⭐⭐ Partial use in sequential analysis |
| **Flat / Debate** | Mesh | CAMEL, FREE-MAD | N/A (no multi-Agent debate need) |
| **Dynamic / Adaptive** | Dynamic | DyLAN, AMAS, REDEREF | ⭐ Academic frontier, future reference |
| **Handoff / Swarm** | Control Transfer | OpenAI Swarm, Google ADK | ⭐⭐ HANDOVER files as lightweight Handoff |
| **Platform / Network** | P2P | OpenAgents Network | N/A (not heterogeneous Agent network) |
| **Consensus / Ensemble** | Parallel Voting | Spotify Ads AI, AI NeuroSignal | N/A (not voting scenario) |

### 2.2 Applicability Assessment of Seven Architecture Patterns

Based on BiliBili (2026) decision matrix:

| Decision Dimension | SSB Characteristics | Best Match Pattern |
|-------------------|---------------------|-------------------|
| Need distributed development? | No (centralized orchestration) | Orchestrator |
| Need parallelization? | Yes (within analysis chain parallelism) | Router + Orchestrator |
| Need multi-hop conversation? | Yes (cross-session via STATE_BLOCK) | Handoff (lightweight) |
| Need direct user interaction? | Yes (user directly converses with each Agent) | Skills (on-demand loading) |
| Token consumption priority? | High (API cost constraints) | Prefer Skills + Simplicity first |

**Conclusion**: Silicon Strategy Bureau is naturally an Orchestrator-Worker + Router hybrid, with Handoff as lightweight supplement. This is more flexible than pure single-mode, but requires user to clearly know which mode each task is currently in.

### 2.3 Key Paper Citations

The following papers provided theoretical support for SSB design decisions:

| Paper | Core Contribution | Insight for SSB |
|-------|-------------------|-----------------|
| **AutoGen (COLM 2024)** | Multi-Agent conversation coordination framework | Source for structured cross-role message type design |
| **MetaGPT (ICLR 2024)** | SOP-driven multi-Agent collaboration | Source for task board steps[] structure |
| **CAMEL (NeurIPS 2023)** | Role-playing Agent communication | Theoretical foundation for four-role personification |
| **AgentVerse (ICLR 2024)** | Dynamic Agent recruitment and collaboration decisions | Future reference: dynamically select participating roles by task |
| **HALO (2025)** | Three-layer architecture + dynamic role generation | Theoretical support for ADR-004 tiered strategy |
| **MAP (Nature Comm. 2025)** | 5-role structured planning | +32% planning enhancement over CoT |
| **PlanGenLLMs (ACL 2025)** | 6-dimension planning quality evaluation | Framework for evaluating task board steps[] quality |
| **FREE-MAD (2025)** | Consensus-free debate (single round achieves multi-round effect) | If debate mechanism introduced in future, single round sufficient |
| **Anthropic Research System** | Orchestrator-Worker 15x token empirical measurement | Direct evidence for ADR-007 cost-aware routing |

---

## III. File Format Architecture

```
workspace/
│
├── 【Shared Layer】strategy-bureau/
│   ├── org-and-communication.md    ← Team charter
│   ├── ai-communication-protocol.md ← Collaboration rules
│   ├── task-board.json             ← ⭐ Structured task registry (JSON)
│   ├── blocking-protocol.md         ← ⭐ Blocking escalation protocol (4 levels)
│   ├── checkpoint-template.json     ← ⭐ Long-flow phase snapshot template (JSON)
│   ├── acceptance-checklist-template.json ← ⭐ Acceptance criteria (JSON)
│   ├── wakeup-wrapup-quickcard.md   ← ⭐ Wakeup/wrap-up SOP
│   ├── tech-architecture.md         ← This file (v2.0)
│   ├── team-daily-board.md          ← Daily team status
│   └── tips-collection.md           ← Experience accumulation
│
├── 【Role Domains】
│   ├── invest-strategy/            ← Agent A · Investment Analysis
│   │   └── checkpoints/            ← ⭐ Long-flow phase files
│   ├── career-strategy/            ← Agent B · Career Development
│   │   └── checkpoints/            ← ⭐ Long-flow phase files
│   ├── tech-consultant/            ← Agent C · Technology Engineering
│   │   └── checkpoints/            ← ⭐ Long-flow phase files
│   └── life-consultant/            ← Agent D · Life Advisor
│
├── 【Deliverables Center】deliverables/
│   ├── *.html                      ← Various analysis reports
│   └── archive/                    ← Historical archives
│
└── .workbuddy/memory/              ← Global memory (long-term + daily logs)
```

---

## IV. JSON Schema Versioning Strategy

| File | Current Version | Versioning Strategy |
|------|----------------|---------------------|
| task-board.json | 1.0.0 | Semantic versioning: MAJOR.MINOR.PATCH |
| checkpoint-template.json | 1.0.0 | Template file not versioned; checkpoint instances include template_version |
| acceptance-checklist-template.json | 1.0.0 | Same as above |
| blocking-protocol.md | 1.0 | Date version (suffix YYYY-MM-DD) |

**Backward Compatibility Rules**:
- Adding fields: Only requires MINOR upgrade
- Modifying required fields: MAJOR upgrade
- Removing fields: MAJOR upgrade, with advance notice to all roles

---

## V. Known Limitations and Pending Decisions

| Item | Status | Notes |
|------|--------|-------|
| Task board archiving strategy | TBD | How to archive completed tasks? |
| STATE_BLOCK maximum size | Defined | Recommended not exceeding 20 lines of JSON |
| Cross-role Checkpoint referencing | TBD | Path convention or registry? |
| CROSS_ROLE_MSG automation | Not implemented | Currently still manual relay by user |
| Token cost tracking | Low priority | ADR-008 defers automated cost statistics |
| Parallel Agent execution | Long-term | Current architecture is serial; parallel requires external state storage |

---

## Changelog

| Date | Version | Changes |
|------|---------|---------|
| 2026-05-21 | v1.0 | Initial version, including ADR-001 through ADR-004 |
| 2026-05-22 | v2.0 | **Multi-Agent paper research upgrade**: Added ADR-005~009 (architecture pattern alignment, structured messaging, cost routing, observability, shared state); Added industry framework benchmarking (7 paradigms + 9 core papers); Updated file architecture diagram |

---

> *"Architecture documents are for your future self and future teammates. Today's decisions must be traceable tomorrow."*
