---
name: migration-create-table-partial-grid-variable
description: Detect and remediate partial grid variable definitions in Create Table components during Matillion ETL to DPC migration. Expands partial column-definition grids to full schema.
---

# Migration Create Table Partial Grid Variable Normalization

## When to Use
- When a migrated **Create Table** component fails when driven by a grid variable
- When the source METL job succeeds but the DPC version fails with grid-related errors
- When DPC validation reports errors such as "default value is null" or "required column metadata missing"
- When a grid variable defines only a partial subset of column metadata fields

## Why This Matters
In METL, Create Table components can use grid variables with only a subset of column-definition fields (e.g., `column_name`, `data_type`, `size`, `position`), and omitted optional fields are tolerated. In DPC, the component enforces stricter requirements — omitted fields may evaluate to `null` in a way that causes runtime failure. Additionally, a **validation/runtime inconsistency** exists: a full grid with blank optional fields may succeed at runtime but still show validation errors.

## Detection Logic

### Identify Affected Components
Flag when:
- A **Create Table** component is configured from a grid variable
- The grid variable defines only a subset of available column-definition fields
- DPC validation and/or execution fails or behaves inconsistently

### Determine if Grid is Partial
If the grid includes only fields like:
- `column_name`
- `data_type`
- `size`
- `position`

but does **not** include other expected fields, classify it as a **partial column-definition grid**.

## Root Cause

DPC distinguishes between:

| Condition | Behavior |
|---|---|
| Field **not present** in grid schema | May trigger runtime or validation errors |
| Field present but **blank/empty** | Usually accepted during execution |

This differs from METL, where omitted optional attributes are tolerated.

## Expected Full Column Schema

DPC expects the grid schema to include **all supported column-definition fields**, even if optional:

- `column_name`
- `data_type`
- `size`
- `position`
- `nullable`
- `default_value`
- `precision`
- `scale`
- `primary_key`
- `unique`
- `description`

Optional attributes should be **defined in the grid structure but left empty** when not needed.

### Example Acceptable Pattern

| column_name | data_type | size | position | nullable | default_value |
|---|---|---|---|---|---|
| C1 | VARCHAR | 20 | 1 | | |
| C2 | INTEGER | | 2 | | |

## Remediation

1. **Expand the grid variable** to include all expected column-definition fields
2. **Set optional fields to blank/empty** rather than omitting them
3. **Validate after expansion** — note that component validation may still flag warnings on blank optional fields
4. **Runtime execution is authoritative** — if execution succeeds with blank optional fields, validation warnings can be documented as known behavior

## Validation vs Runtime Inconsistency

| Stage | Result |
|---|---|
| Component validation | Error or warning (on blank optional fields) |
| Job execution | Successful table creation |

Migration tooling should treat **runtime execution as authoritative** when optional metadata fields are intentionally blank.

## Key Considerations

1. **METL allows partial grids; DPC does not** — This is a behavioral difference, not a bug
2. **Expand, don't rewrite** — Preserve existing column definitions; only add missing fields as blank
3. **Validation warnings may persist** — Document as known behavior if runtime succeeds
4. **Apply before first execution** — Detect and remediate during Phase 3 discovery to avoid runtime failures