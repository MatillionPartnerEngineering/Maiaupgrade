---
name: migration-environment-hardcoding
description: Detect and remediate hardcoded environment references (PRODUCTION database names, S3 paths, SNS topics) in migrated pipelines. Replace with parameterized variables or Environment Defaults.
---

# Migration Environment Hardcoding Remediation

## When to Use
- After initial workload import into DPC
- When pipelines reference wrong database/schema in DEV environment
- When scan reveals `PRODUCTION` or environment-specific strings in pipeline files

## Why This Matters
In prior migrations, **200+ hardcoded PRODUCTION references** have been found across a single workload. These cause pipelines to silently target production data from DEV environments — a critical data safety issue.

## Scan Procedure

### Step 1: Search for Hardcoded References

Search all `.orch.yaml` and `.tran.yaml` files for:

```
Patterns (case-insensitive):
- PRODUCTION          # Hardcoded database name
- PROD                # Abbreviated production refs (careful: may match legitimate names)
- s3://[bucket-name]  # Hardcoded S3 paths
- arn:aws:sns         # Hardcoded SNS topic ARNs
- jdbc:               # Hardcoded JDBC connection strings
- [specific DB names] # Customer-specific production database names
```

### Step 2: Classify Each Reference

| Type | Example | Remediation |
|------|---------|-------------|
| **Database name** | `database: "PRODUCTION"` | Replace with `"[Environment Default]"` or `"${environment}"` variable |
| **Schema name** | `schema: "PROD_FINANCE"` | Replace with `"[Environment Default]"` or parameterize |
| **S3 path** | `s3://prod-bucket/data/` | Parameterize bucket name with variable |
| **SNS topic** | `arn:aws:sns:...:prod-alerts` | Parameterize with variable; may be out-of-scope for DEV |
| **JDBC URL** | `jdbc:oracle:thin:@prodhost:1521` | Replace with secret reference or variable |
| **Legitimate use** | Column named `PRODUCTION_DATE` | ✅ No action — false positive |

### Step 3: Remediation Options

**Option A: Environment Default (Preferred)**
```yaml
# Before
database: "PRODUCTION"
# After
database: "[Environment Default]"
```

**Option B: Pipeline Variable**
```yaml
# Before
database: "PRODUCTION"
# After
database: "${target_database}"
```

**Option C: Conditional (rare)**
For references that must vary by logic, not environment — use a variable with runtime override.

### Step 4: Bulk Remediation
- Group by pattern (e.g., all `PRODUCTION` → `[Environment Default]`)
- Apply find-replace across all files in workload
- Track count: files modified, references replaced
- Validate after replacement

## Key Lessons

1. **200+ references is normal for a large workload** — Don't underestimate the scope
2. **False positives are common** — `PRODUCTION` in column names, comments, or string literals should NOT be replaced
3. **SNS topics may be intentionally out-of-scope** — Production alerting from DEV is usually undesirable; parameterize but leave disabled
4. **Some references are in Python/Bash scripts** — Inline code within components may contain hardcoded paths; these need manual review
5. **Run this BEFORE credentials audit** — Environment refs can mask credential issues (wrong DB = different auth)

## Output
Write inventory to: `migration_project/validation_reports/[WORKLOAD]_PRODUCTION_Hardcoding_Inventory.md`
