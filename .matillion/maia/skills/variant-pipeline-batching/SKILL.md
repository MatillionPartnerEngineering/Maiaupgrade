---
name: variant-pipeline-batching
description: Identify and fix weekend/legacy/v2 pipeline variants together to prevent re-discovering the same issues. Use when working on workloads that have scheduling variants or legacy copies.
---

# Variant Pipeline Batching

## Core Principle

**When you fix a workload, always check for and fix all its variants simultaneously.** Variants share the same codebase and will have identical blockers.

## What Are Variants?

Variants are copies of a workload with different scheduling, scope, or parameters:

| Suffix Pattern | Purpose | Example |
|---|---|---|
| `*_WEEKEND` | Weekend-only schedule | CRM_WEEKEND, ERP_WEEKEND |
| `*_LEGACY` | Old version kept for compatibility | ERP_LEGACY |
| `*_V2` / `*_V3` | Versioned iteration | ERP_V2 |
| `*_FULL` / `*_INCREMENTAL` | Load type variant | CRM_FULL, CRM_INCREMENTAL |
| `*_DAILY` / `*_HOURLY` | Frequency variant | MARKETING_DAILY, MARKETING_HOURLY |

## Identification Process

### Step 1: Find Variants

For each workload being fixed:
1. Search for pipelines with the same base name + suffix
2. Check for pipelines in the same folder with similar structure
3. Look at shared pipeline callers — variants often call the same children

### Step 2: Confirm Shared Code

Variants share blockers when they:
- Call the same child/shared pipelines
- Use the same Python scripts
- Reference the same credentials
- Use the same component patterns (e.g., both have META column Python)

### Step 3: Batch the Fix

Apply the same fix to ALL variants in one session:
1. Fix the primary workload first
2. Apply identical changes to all variants
3. Test all variants in same batch run
4. Document all variants in single validation report

## Rules

1. **Never close a workload without checking for variants** — they WILL fail with the same issue later
2. **Fix all variants in the same PR/commit** — keeps changes atomic and auditable
3. **Test variants together** — if primary passes but variant fails, investigate the delta
4. **Track variants as a group in migration strategy** — one status for the family
5. **If variants diverge significantly, treat as separate workloads** — rare but possible

## Common Variant Patterns

| Primary | Variants | Shared Blockers |
|---|---|---|
| TICKETING_MASTER | TICKETING_WEEKEND | Python cursor(), control table |
| ERP_MASTER | ERP_WEEKEND | OAuth cert, deprecated JDBC v1 |
| CRM_MASTER | CRM_WEEKEND | Performance config, Bulk API |
| BACKUP_MASTER | (scheduling variants) | API Query profile |

## Anti-Patterns

- ❌ Fixing TICKETING_MASTER, marking complete, then discovering TICKETING_WEEKEND has same issue 2 weeks later
- ❌ Testing variants individually when they share 95% of code
- ❌ Creating separate tickets/tasks for each variant's identical blocker

## Efficiency Gains

- **Without batching:** Fix workload (2 hrs) + rediscover same issue in variant (1 hr) + fix variant (1 hr) = 4 hrs
- **With batching:** Fix workload + apply to variant (2.5 hrs total) = **37% time savings**
- **At scale (15 workloads with variants):** Saves ~22 hours across migration
