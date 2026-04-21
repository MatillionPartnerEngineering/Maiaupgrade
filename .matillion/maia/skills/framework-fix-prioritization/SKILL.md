---
name: framework-fix-prioritization
description: How to identify and prioritize framework-level blockers that unblock 5-30 workloads with a single fix. Use when multiple workloads share the same error pattern or when planning fix order for maximum ROI.
---

# Framework Fix Prioritization

## Core Principle

**One framework-level fix unblocks 5-30 workloads.** Always identify and fix shared/framework issues before individual workload issues.

## What is a Framework-Level Blocker?

A blocker that:
- Appears in **3+ workloads** with the same error
- Originates in a **shared pipeline** (called by multiple workloads)
- Is caused by **environment/infrastructure configuration** (not workload-specific data)
- Affects a **common component pattern** (e.g., all Python scripts, all API calls)

## Identification Process

### Step 1: Aggregate Errors Across Workloads

After running batch validation:
1. Group errors by error message similarity
2. Group errors by component type
3. Group errors by pipeline path (shared pipelines appear in multiple workload traces)

### Step 2: Calculate ROI

```
ROI = (Workloads Unblocked × Avg Time Per Individual Fix) ÷ Time to Fix Framework Issue
```

**Example:**
- SNS notification shared pipeline has SQL syntax error
- Affects 30 workloads (all use error notification)
- Individual workaround: 30 min each = 15 hrs total
- Framework fix: 30 min once
- **ROI: 30x**

### Step 3: Priority Order

| Priority | Category | Typical Impact | Fix Time | Examples |
|----------|----------|---------------|----------|----------|
| **P0** | Shared pipelines (called by all) | 20-30 workloads | 30 min | SNS notification, Audit logging, Schema creation |
| **P1** | Environment configuration | 10-20 workloads | 30-60 min | External Access Integration, variable resolution, warehouse permissions |
| **P2** | Template patterns | 5-15 workloads | 2-3 hrs (template build) | Python cursor() template, META column manager |
| **P3** | Component-type issues | 3-7 workloads | 1-2 hrs | Deprecated JDBC connector, API profile batch |

## Common Framework Blockers (ETL-to-DPC)

### SNS/Notification Shared Pipeline
- **Symptom:** Error reporting fails in 30+ workloads
- **Root cause:** SQL syntax incompatible with Snowflake or variable reference broken
- **Fix:** Update shared pipeline SQL, verify variable resolution
- **Impact:** Unblocks error handling for ALL workloads

### Audit/Logging Framework
- **Symptom:** Job tracking fails, executionId not captured
- **Root cause:** System variable syntax changed (`${sysvar.thisPipeline.executionId}`)
- **Fix:** Update all audit components to new syntax
- **Impact:** Unblocks monitoring for all workloads

### External Access Integration
- **Symptom:** All Python components calling external APIs fail
- **Root cause:** DPC Python Pushdown runs inside Snowflake — needs External Access Integration for HTTPS
- **Fix:** Configure External Access Integration for required endpoints
- **Impact:** Unblocks all API-calling Python scripts

### Schema/Table Auto-Creation
- **Symptom:** Multiple workloads fail creating staging tables
- **Root cause:** Shared pipeline for schema management not accessible or misconfigured
- **Fix:** Verify shared pipeline path and permissions
- **Impact:** Unblocks all workloads with staging table patterns

## Rules

1. **Never fix individual workloads if 3+ share the same error** — fix the framework
2. **Fix P0 before running any workload tests** — saves massive rework
3. **After framework fix, immediately re-run ALL affected workloads** — reveals next layer
4. **Track framework fixes separately from workload fixes** — different owners, different timelines
5. **One framework fix per PR** — makes rollback safe if issues arise

## Workflow

1. Run batch validation (all workloads)
2. Aggregate and group errors
3. Identify framework-level patterns (3+ workloads)
4. Fix in P0 → P1 → P2 → P3 order
5. Re-run affected workloads after each framework fix
6. Only then proceed to individual workload debugging
