---
name: migration-credentials-audit
description: Pre-execution audit of all hardcoded credentials, passwords, and secret references across migrated pipelines. Produces a structured inventory and migration plan for DPC Secret Manager.
---

# Migration Credentials Audit

## When to Use
- Before first pipeline execution on any workload
- When encountering authentication failures (ORA-01017, ORA-28000, HTTP 401)
- When onboarding a new workload into the migration project

## Why This Matters
In prior migrations, **200+ password references across 80+ files** have been discovered in a single workload. Unaudited credentials are the #1 blocker category, accounting for more failed first-runs than any other issue. Establishing this audit as **Phase 0** eliminates the most common class of failures before they occur.

## Audit Procedure

### Step 1: Scan for All Credential Patterns
Search all pipeline files (`.orch.yaml`, `.tran.yaml`) for:

```
Patterns to search (regex):
- password:          # YAML password parameters
- secretReference:   # Existing secret refs
- TEXT_SECRET_REF    # Secret reference type parameters
- pwd|passwd         # Abbreviated password fields
- apiKey|api_key     # API credentials
- token              # Auth tokens
- connectionString   # May contain embedded credentials
```

### Step 2: Classify Each Reference

| Classification | Description | Action |
|----------------|-------------|--------|
| **Hardcoded Plain Text** | Actual password value in YAML | 🔴 CRITICAL — Replace immediately with secret ref |
| **Legacy Secret Ref** | References ETL secret name | 🟡 Map to DPC secret name |
| **Already DPC Secret** | Correct DPC secret reference | ✅ Verify secret exists in DPC |
| **Unused/Orphan** | Referenced but pipeline is deprecated | ⚪ Confirm no longer needed |

### Step 3: Build Inventory Table

Produce a table with these columns:

| File | Component | Parameter | Current Value/Ref | Classification | DPC Secret Name | Status |
|------|-----------|-----------|-------------------|----------------|-----------------|--------|

### Step 4: Secret Creation Checklist

For each unique secret needed:
- [ ] Secret name confirmed with customer
- [ ] Secret created in DPC Secret Manager
- [ ] All pipeline references updated to new name
- [ ] Test connection verified

## Key Lessons

1. **Batch credential updates by secret, not by file** — One secret may appear in 30+ files (e.g., `source_db_password` appeared in 25 files with 33 references)
2. **Customer must create secrets** — Migration team cannot create secrets; track as customer action item
3. **Verify secret names exactly** — DPC secret names are case-sensitive and may differ from ETL names (e.g., `Legacy_DB_Prod` → `source_db_password`)
4. **Some secrets are obsolete** — A significant portion of identified secrets may no longer be referenced after cleanup. Always verify before asking customer to create.
5. **Python scripts hide credentials too** — Don't just scan YAML; check `.py` files and inline Python/Bash script content within components

## Output
Write inventory to: `migration_project/validation_reports/Credentials_Secrets_Inventory.md`
