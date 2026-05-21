# Silicon Strategy Bureau · Cross-Role Message Format

> Companion template file for ADR-006
>
> Version: v1.0 · 2026-05-22

## 1. Usage Scenarios

Use when an Agent needs to pass structured information to another Agent within a HANDOVER file. Replaces the easily-lost "user verbal relay" pattern.

## 2. Message Format (JSON)

```json
{
  "msg_id": "msg-YYYYMMDD-NNN",
  "from": "AgentA|AgentB|AgentC|AgentD",
  "to": "AgentA|AgentB|AgentC|AgentD",
  "type": "task|signal|query|handoff",
  "priority": "info|attention|urgent",
  "summary": "One-sentence summary, <50 characters",
  "body": "Detailed content...",
  "refs": [
    "path/to/related/file.json",
    "path/to/another/file.html"
  ],
  "expects_reply": false
}
```

## 3. Message Type Breakdown

| type | Semantics | Direction | expects_reply | Example |
|------|------|------|:---:|------|
| `task` | Task delegation | User→Agent or AgentB→AgentC | true | AgentB delegates AgentC to output a technical capability summary |
| `signal` | Cross-role signal notification | Any→Any | false | AgentC discovers a tech breakthrough in an industry→notifies AgentB |
| `query` | Information query request | Any→Any | true | AgentA queries AgentB for internal information updates |
| `handoff` | State handoff | Agent→Agent (via User) | false | AgentA completes an analysis→hands off to AgentB for related analysis |

## 4. Priority Usage Guidelines

| priority | Meaning | User Handling Strategy |
|----------|------|-------------|
| `info` | General notification, not urgent | Pass along next session |
| `attention` | Needs attention, but not urgent | Pass along within the day |
| `urgent` | Requires immediate action | Pass along immediately and trigger recipient action |

## 5. Embedding in HANDOVER

Insert into HANDOVER HTML file:

```html
<!-- CROSS_ROLE_MSG -->
<script type="application/json" class="cross-role-msg">
{
  "msg_id": "msg-20260522-001",
  "from": "AgentB",
  "to": "AgentA",
  "type": "signal",
  "priority": "attention",
  "summary": "A company's management has increased public appearances recently, restructuring expectations heating up",
  "body": "According to public reports, the company's chairman has attended 3 industry events in the past two weeks, all mentioning 'industry localization entering an acceleration phase.' Combined with the lock-up expiration window about to close, recommend AgentA re-evaluate the timing for increasing position in this stock.",
  "refs": ["career-strategy/checkpoints/career-roadmap.json"],
  "expects_reply": false
}
</script>
<!-- /CROSS_ROLE_MSG -->
```

## 6. Retrieval Method

The user (or automation script) scans HANDOVER files for `class="cross-role-msg"` script blocks upon wake-up, extracts JSON, and routes by the `to` field.

## 7. Security Constraints

- Message content must not contain personal privacy information (salary, ID numbers, etc.)
- `body` field must not exceed 500 characters (to prevent message bloat)
- `refs` file paths must be within the workspace (to prevent path traversal)
