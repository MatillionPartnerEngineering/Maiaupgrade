---
name: migration-test-execution
description: Phased test execution methodology for migrated workloads — entry point identification, execution order, failure classification, timeout handling, and result documentation.
---

# Migration Test Execution Methodology

## When to Use
- After all refactoring and validation is complete for a workload
- When running first end-to-end tests of migrated pipelines
- When classifying and triaging test failures

## Why This Matters
Running all pipelines at once produces cascading failures that are hard to diagnose. Prior migrations have proven that **phased, entry-point-first testing** cuts diagnosis time by 70% and prevents false negatives from dependency failures.

## Phase 1: Identify Entry Points

Entry points are top-level orchestration pipelines that are scheduled or triggered externally.

### How to Find Them
1. **Look for pipelines NOT called by other pipelines** — Search for `run-orchestration` and `run-transformation` components; any pipeline not referenced as a target is likely an entry point
2. **Check the migration strategy** — Entry points are usually documented
3. **Look for scheduling metadata** — Cron expressions, API triggers

### Document Entry Points

| Entry Point | Type | Children | Estimated Runtime | Dependencies |
|-------------|------|----------|-------------------|--------------|
| Main_Load | orch | 12 | 30 min | S3, Oracle |
| Dim_Refresh | orch | 5 | 10 min | Snowflake only |

## Phase 2: Execution Order

**Rule: Test from simplest to most complex.**

1. **Snowflake-only pipelines first** — No external dependencies; isolates DPC issues
2. **Single-source pipelines next** — One external connection; isolates auth/network
3. **Multi-source pipelines last** — Complex dependencies; build on prior successes

Within each tier, run entry points that share infrastructure (same DB, same S3 bucket) sequentially to avoid account lockouts.

## Phase 3: Execute and Classify Results

### Failure Classification System

| Pattern | Symptom | Root Cause | Owner | Resolution Time |
|---------|---------|-----------|-------|-----------------|
| **AUTH** | ORA-01017, HTTP 401, "invalid credentials" | Wrong secret or account locked | Customer DBA | 1-24 hrs |
| **INFRA** | "table does not exist", "schema not found" | Missing Snowflake objects | Customer Infra | 30 min - 2 hrs |
| **NETWORK** | Connection timeout, "host unreachable" | Firewall, decommissioned host | Customer Infra | 4-48 hrs |
| **IAM** | "Access Denied", "not authorized" | Missing S3/cloud permissions | Customer Cloud | 1-24 hrs |
| **SYNTAX** | SQL error, type mismatch | Component config issue | Migration Team | 1-4 hrs |
| **DATA** | "Duplicate row", constraint violation | DEV data artifacts | Migration Team | 30 min |
| **TIMEOUT** | Execution exceeded time limit | Long-running pipeline (not a bug) | N/A | Mark as expected |

### Critical Rule: Primary Blocker Identification

**The PRIMARY blocker is ALWAYS the first error in the execution flow.**

When an entry point fails:
1. Find the **first component** that errored
2. That is the **primary blocker**
3. All downstream failures are **secondary symptoms**
4. Fix ONLY the primary blocker, then re-run

Example: If Oracle auth fails → S3 load skipped → SNS alert fails → The Oracle auth is the PRIMARY blocker. Don't fix SNS.

## Phase 4: Document Results

### Per-Entry-Point Template

```markdown
## [Entry Point Name]
- **Status:** ✅ PASSED / ❌ FAILED / ⏳ BLOCKED
- **Runtime:** X min Y sec
- **Children:** X/Y passed
- **Primary Blocker:** [if failed]
- **Category:** AUTH / INFRA / NETWORK / IAM / SYNTAX / DATA / TIMEOUT
- **Owner:** [who must fix]
- **Notes:** [context]
```

### Workload Summary Template

```markdown
| Entry Point | Status | Children | Blocker | Category | Owner |
|-------------|--------|----------|---------|----------|-------|
```

## Phase 5: Re-Test After Fix

- Re-run ONLY the entry point that was fixed
- If it passes, update status to ✅
- If new failure appears, classify and document
- **Never mark a workload complete until ALL entry points pass**

## Key Lessons

1. **Timeouts ≠ failures** — Long-running pipelines (30-60 min) are expected; don't waste time debugging timeouts on known-heavy workloads
2. **Oracle account lockouts cascade** — One bad password attempt locks the account for ALL 15+ pipelines using it. Test Oracle auth with a single pipeline first.
3. **DEV data duplication is expected** — Seeding + re-running loads creates duplicates. Document as DEV artifact, not a bug.
4. **Parent-injected variables need defaults** — Child pipelines called by iterators may have no default values. Add defaults for isolated testing.
5. **Track runtime for capacity planning** — Production scheduling needs actual DEV runtimes × data volume factor

## Output
Write results to: `migration_project/validation_reports/[WORKLOAD]_Test_Execution_Report_[DATE].md`
