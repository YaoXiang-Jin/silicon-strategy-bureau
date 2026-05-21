# Silicon Strategy Bureau · Cross-Role Message Format

> Companion template file for ADR-006
>
> Version: v1.0 · 2026-05-22

## I. Usage Scenarios

Used when an Agent needs to pass structured information to another Agent within a HANDOVER file. Replaces the error-prone "user verbal relay" pattern.

## II. Message Format (JSON)

```json
{
  "msg_id": "msg-YYYYMMDD-NNN",
  "from": "AgentA|AgentB|AgentC|AgentD",
  "to": "AgentA|AgentB|AgentC|AgentD",
  "type": "task|signal|query|handoff",
  "priority": "info|attention|urgent",
  "summary": "One-line summary, <50 chars",
  "body": "Detailed content...",
  "refs": [
    "path/to/related/file.json",
    "path/to/another/file.html"
  ],
  "expects_reply": false
}
```

## III. Message Type Details

| type | Semantics | Direction | expects_reply | Example |
|------|-----------|-----------|:---:|---------|
| `task` | Delegate task | User→Agent or AgentB→AgentC | true | AgentB delegates AgentC to output tech capability summary |
| `signal` | Cross-role signal notification | Any→Any | false | AgentC discovers industry X tech breakthrough→notifies AgentB |
| `query` | Information query request | Any→Any | true | AgentA queries AgentB for internal info updates |
| `handoff` | State transfer | Agent→Agent (via User) | false | AgentA completes analysis→hands off to AgentB for correlation analysis |

## IV. Priority Usage Guide

| priority | Meaning | User Processing Strategy |
|----------|---------|--------------------------|
| `info` | Normal notification, not urgent | Deliver at next session |
| `attention` | Needs attention, but not urgent | Deliver within the day |
| `urgent` | Requires immediate action | Deliver immediately and trigger recipient action |

## V. How to Embed in HANDOVER

Insert in HANDOVER HTML file:

```html
<!-- CROSS_ROLE_MSG -->
<script type="application/json" class="cross-role-msg">
{
  "msg_id": "msg-20260522-001",
  "from": "AgentB",
  "to": "AgentA",
  "type": "signal",
  "priority": "attention",
  "summary": "Company X management has increased recent public appearances, restructuring expectations rising",
  "body": "According to public reports, the company's chairman attended 3 industry events in the past two weeks, all mentioning 'industry localization entering an acceleration phase.' Combined with the restricted share lockup window about to close, suggest AgentA re-evaluate the timing for adding to this position.",
  "refs": ["career-strategy/checkpoints/career-roadmap.json"],
  "expects_reply": false
}
</script>
<!-- /CROSS_ROLE_MSG -->
```

## VI. Retrieval Method

User (or automation script) scans HANDOVER files for `class="cross-role-msg"` script blocks on wakeup, extracts JSON, routes by `to` field.

## VII. Security Constraints

- Message content must not contain personal privacy information (salary, ID numbers, etc.)
- `body` field must not exceed 500 characters (to prevent message bloat)
- `refs` file paths must be within workspace (to prevent path traversal)
