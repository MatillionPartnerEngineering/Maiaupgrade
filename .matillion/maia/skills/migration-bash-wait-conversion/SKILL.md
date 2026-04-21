---
name: migration-bash-wait-conversion
description: Detect and convert Bash Script sleep/wait commands to Snowflake SYSTEM$WAIT() or DPC-compatible alternatives. Covers retry patterns, timed delays, and polling loops.
---

# Bash Wait/Sleep → DPC Conversion Skill

## When to Activate

- Bash Script component contains `sleep`, `wait`, or timed delay commands
- Pipeline uses "Wait" components (named `Wait 1 min`, `sleep 10m`, etc.)
- Retry/polling patterns using bash loops with sleep intervals

## Detection Rules

Scan for these patterns in Bash Script component `script` parameters:

| Pattern | Example | Severity |
|---------|---------|----------|
| `sleep <N>` | `sleep 600`, `sleep 10m`, `sleep 15m` | 🔴 Blocker |
| `while...sleep` loop | `while true; do check; sleep 30; done` | 🔴 Blocker |
| Component named `Wait*` or `Sleep*` | `Wait 1 min`, `Wait 10 min` | 🔴 Blocker |
| `nc -z` with retry | Port check with sleep between retries | 🟡 Warning |

## Conversion Options (Priority Order)

### Option A: Snowflake SQL Script (Recommended — No Infrastructure)

Replace the Bash Script component with a **SQL Script** orchestration component:

```sql
CALL SYSTEM$WAIT(10, 'MINUTES');
```

Supported units: `SECONDS`, `MINUTES`, `HOURS`

**When to use:** Simple timed delays between pipeline steps.

### Option B: Retry Loop with Fixed Iterator

For retry patterns (check condition → wait → retry), use:

1. **Fixed Iterator** component (set iterations = max retries)
2. Inside loop: Run check query → If success, break; else SQL Script with `CALL SYSTEM$WAIT(30, 'SECONDS')`

**When to use:** Polling for external system readiness, waiting for data availability.

### Option C: Bash Pushdown (Last Resort)

If the wait is part of a complex bash script that cannot be decomposed:

1. Configure Bash Pushdown with external VM + SSH
2. Keep `sleep` command as-is
3. Requires infrastructure setup (SSH key, network access)

**When to use:** Only when wait is embedded in non-separable logic (e.g., mid-script file polling).

## Conversion Procedure

1. **Identify** all Bash Script components with sleep/wait
2. **Classify** each as simple-delay vs retry-loop vs complex-embedded
3. **For simple delays:**
   - Remove Bash Script component
   - Add SQL Script component with `CALL SYSTEM$WAIT(N, 'UNIT');`
   - Reconnect transitions (preserve success/failure paths)
4. **For retry loops:**
   - Design iterator-based retry pattern
   - Add condition check + wait inside loop
5. **Validate** pipeline after conversion

## Common Conversions Found in This Project

| Original | Workload | Recommended Replacement |
|----------|----------|------------------------|
| `sleep 10m` | Workload A | `CALL SYSTEM$WAIT(10, 'MINUTES');` |
| `sleep 15m` | Workload B | `CALL SYSTEM$WAIT(15, 'MINUTES');` |
| `Wait 1 min` (component) | Workload C | `CALL SYSTEM$WAIT(1, 'MINUTES');` |
| `sleep 600` | Various | `CALL SYSTEM$WAIT(10, 'MINUTES');` |

## Important Notes

- `SYSTEM$WAIT()` is a Snowflake system procedure — it runs server-side and does NOT consume DPC agent time
- Maximum wait: consult Snowflake docs (typically up to statement_timeout_in_seconds)
- For waits > 60 minutes, consider restructuring into separate scheduled pipelines
- If the bash script ONLY contains a sleep command, the entire component can be replaced with a single SQL Script
