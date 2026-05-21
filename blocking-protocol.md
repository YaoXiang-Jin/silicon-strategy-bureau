# 🚧 Silicon Strategy Bureau · Blocking Escalation Protocol

> **Applies to**: All members
>
> **Core Philosophy**: A block is not a failure — it is a structured expression of "external input needed to proceed." Turn black-box waiting into visible dependencies.
>
> Version: v1.0

---

## 1. Blocking Levels

| Level | Meaning | Typical Scenario | Escalation Path |
|-------|---------|------------------|-----------------|
| **🔴 BLOCKED** | Fully blocked, requires user decision | User needs to make a clear choice (Plan A vs Plan B), or user needs to provide missing information | Write to `Task Board.json` → blocks field + auto-prompt on next conversation |
| **🟡 STALLED** | Waiting for external data/event | Waiting for earnings release, waiting for policy implementation, waiting for a specific date | Record in HANDOVER, set auto-check time |
| **🟢 DEPENDENCY** | Waiting for another Agent to complete | Agent B waiting for Agent C's work log, Agent A waiting for Agent B's internal information | Write to `Task Board.json` → cross_role_dependencies field |
| **⚪ INFO_NEED** | Information needed but non-blocking | Would like to know but does not affect current work progress | Record in Tips Collection "Pending Info" area |

---

## 2. Blocker Declaration Format (JSON)

```json
{
  "blocker_id": "blk-{YYYYMMDD}-{seq}",
  "role": "AgentA|AgentB|AgentC|AgentD",
  "level": "BLOCKED|STALLED|DEPENDENCY|INFO_NEED",
  "task_id": "Associated task ID (from Task Board)",
  "blocked_at": "ISO 8601 timestamp when blocking occurred",
  "blocking_item": "One-line description of the blocking cause",
  "action_required": "Specific action needed to resolve the block",
  "target_role": "Who needs to resolve it (User/AgentA/AgentB/AgentC/AgentD)",
  "deadline": "If time-constrained, ISO 8601 or null",
  "blocking_other_tasks": ["List of task IDs cascadingly affected by this block"],
  "resolution": null,
  "resolved_at": null
}
```

### Example 1: BLOCKED (Waiting for User Decision)

```json
{
  "blocker_id": "blk-20260521-001",
  "role": "AgentB",
  "level": "BLOCKED",
  "task_id": "career-roadmap",
  "blocked_at": "2026-05-21T15:30:00",
  "blocking_item": "Missing user's clear preference between 'Technical Expert vs Management Track'",
  "action_required": "User to answer: In the next 3 years, do you prefer the technical expert track or the management track?",
  "target_role": "User",
  "deadline": null,
  "blocking_other_tasks": ["investment-analysis"],
  "resolution": null,
  "resolved_at": null
}
```

### Example 2: STALLED (Waiting for External Event)

```json
{
  "blocker_id": "blk-20260521-002",
  "role": "AgentA",
  "level": "STALLED",
  "task_id": "investment-analysis",
  "blocked_at": "2026-05-21T08:00:00",
  "blocking_item": "A listed company's lock-up period expiration window lasts until Month X Day Y — no judgment should be made before then",
  "action_required": "On Month X Day Y+1: Check if lock-up window has closed + whether stock price has stabilized",
  "target_role": "AgentA",
  "deadline": "2026-05-24T09:00:00",
  "blocking_other_tasks": [],
  "resolution": null,
  "resolved_at": null
}
```

### Example 3: DEPENDENCY (Waiting for Another Agent)

```json
{
  "blocker_id": "blk-20260521-003",
  "role": "AgentB",
  "level": "DEPENDENCY",
  "task_id": "skill-inventory",
  "blocked_at": "2026-05-21T16:00:00",
  "blocking_item": "Need AgentC to complete a certain task and update work log before capability signals can be extracted",
  "action_required": "AgentC to complete the corresponding task and update work log",
  "target_role": "AgentC",
  "deadline": null,
  "blocking_other_tasks": [],
  "resolution": null,
  "resolved_at": null
}
```

---

## 3. Blocker Lifecycle

```
Discover blocking →
  │
  ├── Assess level: BLOCKED / STALLED / DEPENDENCY / INFO_NEED
  │
  ├── Write to blockers list:
  │     ├── Task Board.json → append to _blockers array
  │     └── Own HANDOVER → STATE_BLOCK → blockers field
  │
  ├── If blocking_other_tasks is non-empty:
  │     └── Note in own HANDOVER "For Other Roles" section
  │
  └── Wait for resolution →
        │
        ├── Resolution conditions met →
        │     ├── Update blocker: resolution + resolved_at
        │     ├── Remove from _blockers array
        │     └── Continue blocked task
        │
        └── If still blocked past deadline:
              └── Write to ⚠️ Quick Alert area + notify user
```

---

## 4. Wake-Up Blocking Check (All-Member SOP)

```
Every time a role is awakened, execute first:

Step 1 — Read Task Board.json → _blockers array
Step 2 — Read own HANDOVER → STATE_BLOCK → blockers
Step 3 — If unresolved blockers found:
   ├── BLOCKED → Help user resolve the block first (ask or remind)
   ├── STALLED → Check if resolution conditions are met (time/event)
   ├── DEPENDENCY → Check if target role's task is complete
   └── Only continue original task after resolution
Step 4 — If no blockers → proceed normally
```

---

## 5. Known Anti-Patterns of Block Reporting

| Anti-Pattern | Consequence | Correct Approach |
|-------------|-------------|------------------|
| "Wait for user to come back" (no record) | User returns not knowing there is a block | Write to blockers |
| Blocking only written in conversation | Lost after session switch | Write to HANDOVER + Task Board |
| Continue working on other things without marking block | User thinks task is progressing, but it's actually stuck | Explicitly mark BLOCKED, pause task |
| DEPENDENCY not notified to target role | Target role unaware someone is waiting | Write in HANDOVER "For Other Roles" section |

---

> *"Blocking is not the problem. The problem is blocking that no one knows about."*
