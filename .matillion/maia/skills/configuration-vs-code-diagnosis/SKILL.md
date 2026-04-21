---
name: configuration-vs-code-diagnosis
description: Decision tree for classifying migration blockers as configuration (15 min fix) vs code refactor (2-3 hrs). Use when diagnosing pipeline execution failures to determine the fastest resolution path. 73% of ETL-to-DPC blockers are configuration, not code.
---

# Configuration vs Code Diagnosis

## Core Principle

**73% of migration blockers are configuration issues (15 min fix), not code refactors (2-3 hrs).** Always attempt config diagnosis first.

## Decision Tree

### Step 1: Classify the Error Type

| Error Signal | Classification | Typical Fix Time |
|---|---|---|
| "Secret not found" / "Credentials invalid" | ⚙️ CONFIG | 15-30 min |
| "No records found" / "0 rows returned" | ⚙️ CONFIG | 15 min |
| "OAuth profile not found" | ⚙️ CONFIG | 30 min |
| "Pipeline not found" / "Cannot resolve path" | ⚙️ CONFIG | 15 min |
| "Shared pipeline not accessible" | ⚙️ CONFIG | 15 min |
| "Variable not resolved" | ⚙️ CONFIG | 15 min |
| "cursor() not supported" / "connection object" | 💻 CODE | 2-3 hrs |
| "Component deprecated" / "not supported" | 💻 CODE | 1-2 hrs |
| "Module not found" / "import error" | 💻 CODE | 1-2 hrs |
| "Unsupported function" | 💻 CODE | 1-3 hrs |

### Step 2: Configuration Blockers (Fix First)

#### Secret/Credential Issues
- **Symptom:** Authentication failure, 401, "secret not found"
- **Fix:** Verify secret exists in DPC → Check name matches pipeline reference → Validate at runtime
- **Time:** 15-30 min

#### Control Table Records
- **Symptom:** "No records found", Query Result To Scalar returns 0 rows
- **Fix:** Insert SOURCE_SYSTEM_NAME record into ETL_CONTROL table
- **Time:** 15 min

#### OAuth/API Profiles
- **Symptom:** "Profile not found", API connector fails before data retrieval
- **Fix:** Recreate OAuth profile in DPC, verify callback URLs
- **Time:** 30 min

#### Pipeline Path References
- **Symptom:** Run Orchestration/Transformation cannot find target pipeline
- **Fix:** Update path reference to match DPC folder structure
- **Time:** 15 min

#### Variable Resolution
- **Symptom:** `${variable}` appears literally in SQL or component fails with unresolved reference
- **Fix:** Create project variable or update reference name (e.g., `${environment_database}` → `${database_name}`)
- **Time:** 15 min

### Step 3: Code Blockers (Fix After Config is Confirmed)

#### Python cursor() Usage
- **Symptom:** `cursor()` or `connection` object referenced in Python script
- **Fix:** Refactor to Python Pushdown pattern (Snowpark DataFrame API)
- **Time:** 2-3 hrs per component (template reduces to 30 min after first)

#### Deprecated Components
- **Symptom:** Component type not recognized or explicitly deprecated
- **Fix:** Replace with supported equivalent (e.g., modular-jdbc-input-v1 → v2)
- **Time:** 1-2 hrs

#### Unsupported Libraries
- **Symptom:** `import` fails, module not available in DPC Python environment
- **Fix:** Find DPC-compatible alternative or use External Access Integration
- **Time:** 1-3 hrs

## Rules

1. **If the error takes >30 min to diagnose, it's likely CODE** — escalate to refactor process
2. **If the error mentions authentication/paths/variables, it's CONFIG** — don't touch code
3. **Never refactor code to work around a config issue** — fix the config
4. **After fixing config, always re-run immediately** — expect the next layer (see sequential-blocker-discovery)

## Observed Metrics

- Config blockers: 73% of all blockers, avg 15 min resolution
- Code blockers: 27% of all blockers, avg 2 hrs resolution
- **Misdiagnosis cost:** 2-3 hrs wasted when code is attempted for a config issue
