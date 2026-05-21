# 🚧 Silicon Strategy Bureau · Blocking Escalation Protocol

> **Applicable To**: All members
>
> **Core Concept**: Blocking is not failure — blocking is the structured expression of "external input needed to continue." Turn black-box waiting into visible dependencies.
>
> Version: v1.0

---

## I. Blocking Levels

| Level | Meaning | Typical Scenario | Escalation Path |
|-------|---------|------------------|-----------------|
| **🔴 BLOCKED** | Fully blocked, requires user decision | User needs to explicitly decide (Option A vs B), user needs to provide missing information | Write to `task-board.json` → blocks field + auto-prompt next conversation |
| **🟡 STALLED** | Waiting for external data/event | Waiting for earnings release, waiting for policy to land, waiting for a date | Record in HANDOVER, set automatic check time |
| **🟢 DEPENDENCY** | Waiting for another Agent to complete | Agent B waiting for Agent C's work log, Agent A waiting for Agent B's internal info | Write to `task-board.json` → cross_role_dependencies field |
| **⚪ INFO_NEED** | Need information but non-blocking | Want to know but doesn't affect current work progress | Record in Tips Collection "supplementary info" section |

---

## II. Blocking Declaration Format (JSON)

```json
{
  "blocker_id": "blk-{YYYYMMDD}-{seq}",
  "role": "AgentA|AgentB|AgentC|AgentD",
  "level": "BLOCKED|STALLED|DEPENDENCY|INFO_NEED",
  "task_id": "Associated task ID (from task board)",
  "blocked_at": "Blocking occurrence time ISO 8601",
  "blocking_item": "One-line description of blocking cause",
  "action_required": "Specific action needed to resolve",
  "target_role": "Who needs to resolve (User/AgentA/AgentB/AgentC/AgentD)",
  "deadline": "If time constraint exists, ISO 8601 or null",
  "blocking_other_tasks": ["List of task IDs cascadingly affected"],
  "resolution": null,
  "resolved_at": null
}
```

### Example 1: BLOCKED (Awaiting User Decision)

```json
{
  "blocker_id": "blk-20260521-001",
  "role": "AgentB",
  "level": "BLOCKED",
  "task_id": "career-roadmap",
  "blocked_at": "2026-05-21T15:30:00",
  "blocking_item": "Lack of user's explicit preference on 'tech expert vs management track'",
  "action_required": "User answers: In the next 3 years, do you prefer the tech expert track or management track?",
  "target_role": "User",
  "deadline": null,
  "blocking_other_tasks": ["investment-analysis"],
  "resolution": null,
  "resolved_at": null
}
```

### Example 2: STALLED (Awaiting External Event)

```json
{
  "blocker_id": "blk-20260521-002",
  "role": "AgentA",
  "level": "STALLED",
  "task_id": "investment-analysis",
  "blocked_at": "2026-05-21T08:00:00",
  "blocking_item": "Company X restricted share lockup window until Month X Day X — not advisable to judge before then",
  "action_required": "Month X Day X+1: Check if lockup window closed + whether stock price stabilized",
  "target_role": "AgentA",
  "deadline": "2026-05-24T09:00:00",
  "blocking_other_tasks": [],
  "resolution": null,
  "resolved_at": null
}
```

### Example 3: DEPENDENCY (Awaiting Another Agent)

```json
{
  "blocker_id": "blk-20260521-003",
  "role": "AgentB",
  "level": "DEPENDENCY",
  "task_id": "skill-inventory",
  "blocked_at": "2026-05-21T16:00:00",
  "blocking_item": "Need AgentC to complete task X and update work log to extract capability signals",
  "action_required": "AgentC completes corresponding task and updates work log",
  "target_role": "AgentC",
  "deadline": null,
  "blocking_other_tasks": [],
  "resolution": null,
  "resolved_at": null
}
```

---

## III. Blocking Lifecycle

```
Discover Blockage →
  │
  ├── Assess level: BLOCKED / STALLED / DEPENDENCY / INFO_NEED
  │
  ├── Write to blockers list:
  │     ├── task-board.json → append to _blockers array
  │     └── own HANDOVER → STATE_BLOCK → blockers field
  │
  ├── If blocking_other_tasks not empty:
  │     └── Note in own HANDOVER "for other roles" section
  │
  └── Await resolution →
        │
        ├── Resolution conditions met →
        │     ├── Update blocker: resolution + resolved_at
        │     ├── Remove from _blockers array
        │     └── Continue blocked task
        │
        └── If past deadline and still blocked:
              └── Write to ⚠️ Rapid Alert Zone + notify user
```

---

## IV. Blocking Check on Wakeup (All-Member SOP)

```
On each role activation, prioritize:

Step 1 — Read task-board.json → _blockers array
Step 2 — Read own HANDOVER → STATE_BLOCK → blockers
Step 3 — If unresolved blockages found:
   ├── BLOCKED → Help user resolve (ask or remind)
   ├── STALLED → Check if resolution conditions met (time/event)
   ├── DEPENDENCY → Check if target role task completed
   └── Only continue original task after resolution
Step 4 — If no blockages → Normal continuation
```

---

## V. Known Anti-Patterns of Blocking Reporting

| Anti-Pattern | Consequence | Correct Approach |
|-------------|-------------|------------------|
| "Wait for user to come back" (no record) | User doesn't know there's a blockage when returning | Write to blockers |
| Blocking only recorded in conversation | Blockage lost on session switch | Write to HANDOVER + task board |
| Continue other work while blocked without marking | User thinks task is progressing, actually stuck | Explicitly mark BLOCKED, pause task |
| DEPENDENCY not notified to target role | Target role doesn't know someone is waiting | Clearly note in HANDOVER "for other roles" section |

---

> *"Blocking is not scary. What's scary is blocking that no one knows about."*
