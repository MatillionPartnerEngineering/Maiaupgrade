---
name: sequential-blocker-discovery
description: The 6-layer cascade order for migration blockers - fixing one always reveals the next. Use when planning fix order or when a workload requires multiple sequential fixes to reach success.
---

# Sequential Blocker Discovery

## Core Principle

**Migration blockers reveal themselves in layers.** Fixing blocker N always reveals blocker N+1. This is expected behavior, not a sign of poor analysis. Plan for 2-4 layers per workload.

## The 6-Layer Cascade (Always This Order)

```
Layer 1: Control Table / Config
  ↓ (pipeline can now start)
Layer 2: Secrets / Credentials  
  ↓ (pipeline can now authenticate)
Layer 3: Python / Component Refactor
  ↓ (components can now execute)
Layer 4: Network / External Access
  ↓ (can now reach external systems)
Layer 5: Data / Payload
  ↓ (can now process data)
Layer 6: Framework / Audit / Monitoring
  ↓ (SUCCESS ✅)
```

## Layer Details

### Layer 1: Control Table / Configuration
- **Blocks:** Pipeline won't start or immediately fails on first component
- **Symptoms:** "No records found", "Variable not resolved", "Pipeline not found"
- **Fix time:** 15 min
- **Action:** Fix and immediately re-run

### Layer 2: Secrets / Credentials
- **Blocks:** Pipeline starts but can't authenticate to source/target systems
- **Symptoms:** "401 Unauthorized", "Secret not found", "OAuth token expired"
- **Fix time:** 15-30 min
- **Action:** Validate secret at runtime, then re-run

### Layer 3: Python / Component Refactor
- **Blocks:** Authenticated but component logic is incompatible with DPC
- **Symptoms:** "cursor() not supported", "deprecated component", "module not found"
- **Fix time:** 1-3 hrs
- **Action:** Apply refactor pattern from migration_documentation.md, then re-run

### Layer 4: Network / External Access
- **Blocks:** Component runs but can't reach external resources
- **Symptoms:** "Connection timeout", "AccessDenied", "Network unreachable"
- **Fix time:** 30 min - 2 hrs (may require external team)
- **Action:** Configure External Access Integration or firewall rules, then re-run

### Layer 5: Data / Payload
- **Blocks:** Reaches data source but size/format causes failure
- **Symptoms:** "Payload too large", "Grid exceeds limit", "Invalid data format"
- **Fix time:** 1-2 hrs
- **Action:** Reduce batch size, add pagination, fix data format, then re-run

### Layer 6: Framework / Audit / Monitoring
- **Blocks:** Data processes successfully but monitoring/notification fails
- **Symptoms:** SNS error, audit table write failure, logging component error
- **Fix time:** 30 min (often framework-level fix)
- **Action:** Fix shared pipeline or audit component. This is the LAST blocker before success.

## Rules

1. **Always re-run immediately after fixing a layer** — don't investigate downstream errors until upstream is resolved
2. **Don't diagnose Layer 3+ until Layers 1-2 are clear** — most downstream errors are consequences of upstream failures
3. **Expect 2-4 layers per workload** — budget time accordingly (simple: 2 layers, complex: 4-5 layers)
4. **Layer 6 failures with all data processing succeeding = Framework issue** — fix once, unblocks all workloads
5. **Track which layer each workload is currently at** — helps predict remaining effort

## Time Budgeting by Layer Count

| Layers Remaining | Expected Time to Success | Complexity |
|---|---|---|
| 1 (Layer 6 only) | 30 min | Simple |
| 2 (Layers 5-6) | 2-3 hrs | Moderate |
| 3 (Layers 3-5) | 4-5 hrs | Complex |
| 4+ (Layers 1-4+) | 6-8 hrs | High complexity |

## Anti-Patterns

- ❌ Investigating "SNS notification failed" when the primary data pipeline also failed (Layer 6 symptom masking Layer 2-3 issue)
- ❌ Spending 2 hours on Python refactor when the control table record is missing (Layer 3 before Layer 1)
- ❌ Reporting "5 errors found" when 4 are consequences of the first (cascade confusion)
- ❌ Waiting to batch all fixes before re-running (each fix should trigger immediate retest)

## Workload Status Tracking

Use layer position in status updates:
- "TICKETING_MASTER: Layer 3 (Python cursor refactor)" — tells you Layers 1-2 are resolved
- "NETWORKING_MASTER: Layer 2 (API key validation)" — tells you config is done, creds pending
- "CRM_MASTER: Layer 6 (audit only)" — nearly complete, just monitoring to fix
