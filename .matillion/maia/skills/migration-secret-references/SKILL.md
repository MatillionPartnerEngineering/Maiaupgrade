---
name: secret-reference-fix
description: Identifies and fixes incorrectly formatted secret references in database-query components migrated from Matillion ETL (METL) to Data Productivity Cloud (DPC). Covers the secretReferenceNameId migration issue and the missing concurrencyMethod parameter.
---

# Secret Reference Fix for Migrated Pipelin

For security reasons, credentials such as secrets and passwords are **not migrated** from Matillion ETL to the Data Productivity Cloud.

- Any secrets or other credentials you have set up in Matillion ETL must be recreated manually in the Data Productivity Cloud to allow pipelines to run.
- Read **Secrets and secret definitions** and **Cloud provider credentials** for details.

Passwords can’t be entered directly into Data Productivity Cloud components (by design). All passwords must be stored in **secrets**, which the component references.

- Secrets are stored in:
  - DPC secret manager in a **Full SaaS** environment, or
  - your own cloud platform’s secret manager in a **Hybrid SaaS** environment.

## Problem

When pipelines are migrated from Matillion ETL (METL) to Data Productivity Cloud (DPC), the `password` parameter on `database-query` components is often imported in an **incorrect object format** that DPC does not resolve at runtime. This causes a generic **"An error occurred while staging"** error.

A secondary issue is that some migrated components are **missing the required `concurrencyMethod` parameter**, which also triggers the same generic staging error.

## Root Cause

The METL migration exports secrets using a nested object format:

```yaml
# ❌ WRONG — METL migration format (does NOT work at runtime in DPC)
password:
  secretReferenceNameId: "MY_SECRET_NAME"
```

DPC expects `TEXT_SECRET_REF` parameters as a **plain string** matching the secret definition name:

```yaml
# ✅ CORRECT — DPC native format
password: "MY_SECRET_NAME"
```

The `secretReferenceNameId` object format is saved to the YAML but is **not resolved by the DPC runtime**, causing the component to fail with a staging error. Manually selecting the secret from the UI dropdown works because the UI writes the correct plain string format.

## Detection

### Step 1: Find all `secretReferenceNameId` occurrences

Search across pipeline files for the incorrect format:

```
Pattern: secretReferenceNameId
Glob: **/*.orch.yaml
```

Any match indicates a password parameter that needs to be converted.

### Step 2: Find missing `concurrencyMethod`

For `database-query` components, check that every component with a `concurrencyValue` parameter also has a `concurrencyMethod` parameter. If `concurrencyMethod` is missing, it must be added.

## Fix

### Fix 1: Convert secret references to plain string format

Replace every occurrence of:

```yaml
        password:
          secretReferenceNameId: "SECRET_NAME"
```

With:

```yaml
        password: "SECRET_NAME"
```

This can be done with a find-and-replace per secret name. For example:

- `password:\n          secretReferenceNameId: "JDE_PRD_SEC_DEF"` → `password: "JDE_PRD_SEC_DEF"`
- `password:\n          secretReferenceNameId: "HI_JDE_PRD_DEF"` → `password: "HI_JDE_PRD_DEF"`

Use `replaceAll: true` to fix all occurrences in a file at once.

### Fix 2: Add missing `concurrencyMethod`

For any `database-query` component that has `concurrencyValue` but no `concurrencyMethod`, add:

```yaml
        concurrencyMethod: "Absolute"
```

Directly after the `concurrencyValue` line. The default value is `"Absolute"`. The other valid option is `"STV_SLICES"` (for Amazon Redshift).

## Verification

### Using dynamic lookup to find valid secret names

To confirm valid secret names for a component, use the dynamic lookup on the `password` parameter. The lookup returns plain string options representing available project-level secret definitions.

### Mapping secrets to connections

When multiple secrets exist, match the correct secret to each component based on its connection details:

| Connection URL Pattern | Typical Secret | Notes |
|---|---|---|
| `jdbc:as400://QCDPRD.GSF.CC/...` | `JDE_PRD_SEC_DEF` | QCD production |
| `jdbc:as400://JDEPD.gsf.cc/...` | `HI_JDE_PRD_DEF` | HI production |

*(Add rows for other connection/secret mappings as discovered)*

### Run the pipeline

After applying fixes, run the pipeline to confirm all `database-query` components succeed. A successful run means the secrets are properly resolved and staging completes.

## Scope

This issue affects **all `database-query` components** migrated from METL that use the `secretReferenceNameId` format. Other component types with `TEXT_SECRET_REF` parameters (e.g., `modular-*` connectors) may also be affected — apply the same plain string format.

## Key Takeaway

For any `TEXT_SECRET_REF` parameter in DPC pipelines, always use the **plain string format** with the secret definition name. Never use the `secretReferenceNameId` nested object format.
