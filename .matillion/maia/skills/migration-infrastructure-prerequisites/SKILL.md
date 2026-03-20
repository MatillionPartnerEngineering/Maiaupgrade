---
name: migration-infrastructure-prerequisites
description: Pre-execution checklist for infrastructure dependencies — Snowflake schemas, control tables, network connectivity, IAM permissions, and external system access required before pipeline testing.
---

# Migration Infrastructure Prerequisites

## When to Use
- Before first test execution of any workload
- When pipelines fail with "table does not exist", "schema does not exist", or connection timeout errors
- During Phase 0 of workload onboarding

## Why This Matters
Missing infrastructure is the **second most common blocker** after credentials. In prior migrations, missing control tables (e.g., `ETL_STATUS`), schemas, and network routes to legacy hosts each blocked entire workloads. These are cheap to fix but expensive to discover at runtime.

## Pre-Execution Checklist

### 1. Snowflake Objects

- [ ] **Target database exists** in the DPC environment
- [ ] **All referenced schemas exist** — scan for unique schema references:
  ```
  Search pattern: schema: "[^\[\$].*"  (excludes [Environment Default] and ${var})
  ```
- [ ] **Control/ETL tables exist** — look for tables used for job tracking, status flags, watermarks:
  ```
  Common patterns: ETL_STATUS, ETL_CONTROL, JOB_LOG, WATERMARK, AUDIT_LOG
  ```
- [ ] **Staging tables exist** (or will be created by pipeline) — verify CREATE TABLE components run before INSERT/LOAD

### 2. External Database Connectivity

- [ ] **All JDBC endpoints reachable** from DPC agent — scan for hostname:port patterns
- [ ] **Database accounts unlocked and active** — especially Oracle (ORA-28000 = locked)
- [ ] **Firewall rules allow DPC agent IP range** — customer infra team must whitelist

### 3. Cloud Services (AWS/Azure/GCP)

- [ ] **S3 buckets accessible** — verify IAM role has required permissions:
  ```
  Common missing permissions:
  - s3:GetObjectTagging
  - s3:PutObjectTagging
  - s3:ListBucket
  - s3:GetObject / s3:PutObject
  ```
- [ ] **SNS topics exist** (if used for alerting) — or parameterize/disable for DEV
- [ ] **IAM roles attached** to DPC execution environment

### 4. Network & DNS

- [ ] **All hostnames resolve** — scan for host/server parameters; verify DNS
- [ ] **Decommissioned hosts identified** — legacy endpoints may no longer exist
- [ ] **VPN/PrivateLink configured** (if required for on-prem sources)

## Discovery Procedure

### Automated Scan
```
Search pipeline files for:
- database: "..."     → List unique databases
- schema: "..."       → List unique schemas
- host|server: "..."  → List unique endpoints
- s3://               → List unique S3 paths
- jdbc:               → List unique JDBC URLs
- port: "..."         → List unique ports
```

### Cross-Reference with Warehouse
```sql
-- Check if schemas exist
SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA WHERE CATALOG_NAME = '<database>';

-- Check if control tables exist
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = '<schema>' AND TABLE_NAME LIKE '%ETL%';
```

## Escalation Template

When infrastructure is missing, create a customer action item:

```
**Action Required:** [Create schema / Grant IAM permission / Verify network route]
**Resource:** [Schema name / S3 bucket / Hostname:port]
**Blocks:** [X pipelines in Y workload]
**Owner:** [Customer Infra / DBA / Cloud team]
**Priority:** [P1-HIGH if blocks >5 pipelines, P2 otherwise]
```

## Key Lessons

1. **Control tables are invisible until runtime** — They don't appear in component parameters; only discovered when pipeline fails
2. **Schema creation is a 30-second fix** — But waiting for customer approval can take 24+ hours. Batch all schema requests.
3. **Decommissioned hosts are common** — Legacy ETL often references endpoints that no longer exist. Confirm status early.
4. **S3 tagging permissions are frequently missing** — Standard S3 read/write policies often omit `GetObjectTagging`/`PutObjectTagging`
5. **Oracle account lockouts cascade** — One failed connection attempt can lock the account, blocking 15+ pipelines
