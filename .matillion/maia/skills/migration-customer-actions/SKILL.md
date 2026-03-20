---
name: migration-customer-actions
description: Template and methodology for creating, tracking, and resolving customer action items during migration. Includes priority framework, ownership assignment, and status tracking.
---

# Migration Customer Action Management

## When to Use
- After test execution reveals blockers requiring customer action
- When creating weekly status updates
- When consolidating action items across multiple workloads

## Why This Matters
Migration velocity is gated by **customer response time**, not migration team effort. In prior migrations, 80% of blockers required customer action (credentials, IAM, network, DBA). Structured, prioritized action requests cut average resolution time from days to hours.

## Action Request Document Template

### File Location
`migration_project/validation_reports/Customer_Action_Request_[DATE].md`

### Structure

```markdown
# Customer Action Request — [Customer Name] Migration
**Date:** [DATE]
**Prepared by:** Migration Team
**Response requested by:** [DATE + 2 business days]

## Summary
[X] action items across [Y] workloads. [Z] are P1-HIGH blocking [N] pipelines.

## Action Items

| # | Priority | Action | Owner | Workload | Pipelines Blocked | Status |
|---|----------|--------|-------|----------|-------------------|--------|
| 1 | P1-HIGH | [Specific action needed] | [Team] | [Workload] | [Count] | 🔴 Open |
| 2 | P2 | [Specific action needed] | [Team] | [Workload] | [Count] | 🔴 Open |

## Detail Per Action

### Action 1: [Title]
- **What:** [Exact action needed — be specific enough that customer can act without asking questions]
- **Why:** [Error message or symptom observed]
- **Where:** [Exact resource — bucket name, hostname, schema, account]
- **Blocks:** [List of pipeline names]
- **How to verify:** [How customer confirms it's done]

## Recently Resolved
| # | Action | Resolved Date | Impact |
|---|--------|---------------|--------|
| 1 | [What was fixed] | [Date] | [What it unblocked] |
```

## Best Practices

### 1. Be Specific and Actionable
❌ "Fix S3 permissions"  
✅ "Add `s3:GetObjectTagging` and `s3:PutObjectTagging` to IAM role `matillion-dpc-role` for bucket `customer-data-bucket`"

### 2. Group by Owner
Don't send 5 items to 3 different teams in one email. Group by owner so each team sees only their items.

### 3. Include Verification Steps
Always tell the customer how to confirm the action is complete. This prevents "I think I did it" back-and-forth.

### 4. Track Resolution in the Document
When an item is resolved:
- Move to "Recently Resolved" section
- Note the date and what it unblocked
- This builds a visible history of progress

### 5. Limit to 5-7 Items
More than 7 items causes decision paralysis. If you have 15 blockers, send the top 5-7 with a note that more will follow.

## Escalation Cadence

| Time Since Request | Action |
|-------------------|--------|
| 0 days | Send action request document |
| 2 business days | Follow up on P1 items only |
| 5 business days | Escalate P1 to project sponsor |
| 7 business days | Flag in weekly status as "at risk" |

## Integration with Other Skills

- **migration-blocker-triage** → Classifies failures into action items
- **migration-test-execution** → Produces the test results that generate action items
- **migration-weekly-update** → Includes action item status in weekly report
- **migration-strategy-and-plan-template** → Updates workload status based on resolution

## Key Lessons

1. **One consolidated document beats multiple emails** — Customer teams lose track of scattered requests
2. **Resolved items section builds trust** — Shows progress and acknowledges customer effort
3. **P1 vs P2 distinction drives urgency** — Without priority, everything gets treated as P3
4. **"Response requested by" date works** — Polite deadline increases response rate
5. **Verification steps save round-trips** — "How to verify" eliminates 50% of follow-up questions
