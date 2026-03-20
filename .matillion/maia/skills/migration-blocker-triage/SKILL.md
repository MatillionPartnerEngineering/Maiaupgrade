---
name: migration-blocker-triage
description: Escalation decision tree and blocker categorization for migration failures. Classifies blockers by priority, assigns ownership, and provides resolution playbooks for each category.
---

# Migration Blocker Triage

## When to Use
- When a pipeline execution fails and you need to classify the failure
- When prioritizing multiple blockers across workloads
- When creating customer action items from test results

## Why This Matters
In prior migrations, **unstructured triage wasted hours** diagnosing secondary symptoms. This decision tree cuts diagnosis to minutes by focusing on the **primary blocker** and routing to the correct owner immediately.

## Decision Tree

```
❌ Pipeline Failed
│
├─ Is it an authentication error?
│  (ORA-01017, ORA-28000, HTTP 401, "invalid credentials", "access denied")
│  → CATEGORY: AUTH
│  → OWNER: Customer DBA / Infra
│  → PLAYBOOK: Verify secret exists in DPC, check account lock status, confirm password
│  → RESOLUTION TIME: 1-24 hrs
│
├─ Is it a missing object?
│  ("table does not exist", "schema not found", "object not found")
│  → CATEGORY: INFRASTRUCTURE
│  → OWNER: Customer Infra / Migration Team
│  → PLAYBOOK: Create schema/table, verify CREATE component runs first
│  → RESOLUTION TIME: 30 min - 2 hrs
│
├─ Is it a network/connectivity error?
│  (Connection timeout, "host unreachable", DNS failure)
│  → CATEGORY: NETWORK
│  → OWNER: Customer Infra
│  → PLAYBOOK: Verify host is active, check firewall rules, confirm VPN
│  → RESOLUTION TIME: 4-48 hrs (longest category)
│
├─ Is it a permissions error?
│  ("Access Denied" on S3, "not authorized", IAM policy error)
│  → CATEGORY: IAM/PERMISSIONS
│  → OWNER: Customer Cloud Team
│  → PLAYBOOK: Identify exact permission needed, provide IAM policy snippet
│  → RESOLUTION TIME: 1-24 hrs
│
├─ Is it a component/syntax error?
│  (SQL syntax error, type mismatch, .contains() → .includes())
│  → CATEGORY: SYNTAX/REFACTOR
│  → OWNER: Migration Team
│  → PLAYBOOK: Fix component config, check migration-documentation skill
│  → RESOLUTION TIME: 1-4 hrs
│
├─ Is it a data error?
│  ("Duplicate row", constraint violation, NULL where NOT NULL)
│  → CATEGORY: DATA
│  → OWNER: Migration Team (DEV) / Customer DBA (source data)
│  → PLAYBOOK: Check DEV seeding artifacts, truncate staging, verify source
│  → RESOLUTION TIME: 30 min - 4 hrs
│
└─ Is it a timeout?
   (Execution time exceeded)
   → CATEGORY: TIMEOUT
   → OWNER: N/A (not a bug)
   → PLAYBOOK: Mark as long-running, increase timeout if needed
   → RESOLUTION TIME: N/A
```

## Blocker Priority Framework

| Priority | Criteria | Response |
|----------|----------|----------|
| **P1 — HIGH** | Blocks >5 pipelines OR blocks entire workload | Escalate immediately; include in daily standup |
| **P2 — MEDIUM** | Blocks 1-5 pipelines | Include in customer action request; track weekly |
| **P3 — LOW** | Non-blocking (warning, DEV-only artifact) | Document; address after all P1/P2 resolved |

### Blocker Hierarchy (Within a Single Failure)

1. **PRIMARY BLOCKER** — First error in execution flow. This is the ONLY thing to fix.
2. **SECONDARY SYMPTOMS** — Downstream failures caused by #1 (e.g., SNS fails because load failed because auth failed)
3. **FRAMEWORK ISSUES** — Affect multiple workloads (e.g., missing IAM role affects all S3 pipelines)

**Rule:** Always fix PRIMARY first. Never fix SECONDARY in isolation — it resolves automatically when PRIMARY is fixed.

## Customer Action Item Template

```markdown
| Priority | Blocker | Owner | Unblocks | Status |
|----------|---------|-------|----------|--------|
| P1 | [Description] | [Team] | [X pipelines in Y workload] | 🔴 Open |
```

## Cross-Workload Pattern Recognition

When the same blocker appears across workloads, elevate to **Framework Issue**:

| Pattern | Example | Impact |
|---------|---------|--------|
| Same secret missing in multiple workloads | `shared_db_secret` used in Oracle + Finance | Fix once, unblocks both |
| Same host unreachable | `legacy-db-host` in 3 workloads | One network fix unblocks all |
| Same IAM permission | S3 tagging across all S3 workloads | One policy update fixes all |

## Key Lessons

1. **The first error is the only error that matters** — In a 21-pipeline workload, a single Oracle auth failure cascaded to 7 "secondary" failures. Fixing the one auth issue resolved all 7.
2. **Network issues take the longest** — Budget 48 hrs for firewall/DNS changes; plan other work while waiting
3. **Batch customer actions by owner** — Don't send 5 separate requests to the same team; consolidate into one prioritized list
4. **Oracle account lockout is a cascade risk** — One bad attempt locks the account for ALL pipelines. Always test with one pipeline first.
5. **"Access Denied" on S3 is almost always missing tagging permissions** — Standard read/write policies omit `s3:GetObjectTagging` and `s3:PutObjectTagging`
