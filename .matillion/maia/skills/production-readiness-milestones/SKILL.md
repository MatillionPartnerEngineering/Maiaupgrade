---
name: production-readiness-milestones
description: 7-step sequence from dev validation through ETL decommission for production cutover. Use when planning or tracking the path from successful migration testing to live production deployment.
---

# Production Readiness Milestones

## Core Principle

**Migration success ≠ production readiness.** A workload passing in dev is only 60% of the journey. Establish production milestones DURING migration, not after.

## The 7-Step Sequence

```
1. Dev Validation        ──→  All workloads pass in dev environment
2. UAT Environment Setup ──→  UAT provisioned with prod-like config  
3. UAT Testing           ──→  All workloads pass in UAT
4. Prod Agent Provision   ──→  Production agent deployed and connected
5. Prod Secrets Config    ──→  Production credentials validated
6. Scheduler Integration  ──→  RunMyJobs/Cron/Scheduler connected
7. Cutover & Decommission ──→  ETL disabled, DPC live, rollback window closed
```

## Milestone Details

### Milestone 1: Dev Validation ✅
**Gate criteria:**
- All active workloads execute end-to-end without error (parent + children + shared)
- No blocker-level refactor items remaining
- All secrets runtime-validated in dev
- Performance within acceptable bounds (no 15-hour jobs unless expected)

**Deliverable:** All workload validation reports show SUCCESS

### Milestone 2: UAT Environment Setup
**Gate criteria:**
- UAT environment provisioned in DPC
- UAT database/schema created with prod-like structure
- UAT secrets created (may be same as prod or separate)
- UAT agent connected and healthy

**Deliverable:** UAT environment accessible and configured

### Milestone 3: UAT Testing
**Gate criteria:**
- All workloads run successfully in UAT
- Data validation: Row counts match expected
- Performance validation: Execution times within SLA
- Error handling validated: Notifications fire correctly

**Deliverable:** UAT sign-off from business stakeholders

### Milestone 4: Production Agent Provisioning
**Gate criteria:**
- Production agent deployed in customer infrastructure
- Network connectivity to all required data sources confirmed
- Agent health checks passing
- Warehouse connectivity verified

**Deliverable:** Agent status GREEN in DPC console

### Milestone 5: Production Secrets Configuration
**Gate criteria:**
- All production secrets created in cloud provider
- All secrets registered in DPC production environment
- Runtime validation of each secret (actual authentication test)
- OAuth tokens refreshed and not expiring before go-live

**Deliverable:** Secret validation checklist 100% complete

### Milestone 6: Scheduler Integration
**Gate criteria:**
- Scheduler (RunMyJobs/native DPC/cron) configured with correct schedules
- API triggers tested end-to-end
- Schedule matches existing ETL cadence (or improved)
- Dependency chains preserved (workload A before B)
- Alerting configured for missed/failed schedules

**Deliverable:** All schedules active in test mode (not yet live)

### Milestone 7: Cutover & Decommission
**Gate criteria:**
- Final parallel run: ETL and DPC run simultaneously, results compared
- ETL schedules disabled
- DPC schedules activated
- Rollback plan documented and tested
- Rollback window defined (typically 1-2 weeks)
- Matillion ETL instance decommissioned after rollback window closes

**Deliverable:** Production live confirmation + decommission ticket

## Rollback Plan

Every cutover must have a rollback plan:

1. **Trigger:** Define what constitutes a rollback trigger (e.g., >3 critical failures in first 48 hrs)
2. **Process:** Re-enable ETL schedules, disable DPC schedules
3. **Data:** Ensure no data loss during DPC operation (idempotent loads)
4. **Timeline:** Rollback window = 1-2 weeks post-cutover
5. **Decision maker:** Named person who authorizes rollback

## Rules

1. **Start Milestone 2-3 planning when dev is at 70%+** — don't wait for 100%
2. **Production secrets are separate from dev secrets** — never share credentials across environments
3. **Schedule integration is the most common delay** — start early, requires external team coordination
4. **Parallel run is non-negotiable** — at least 1 full cycle of both systems running simultaneously
5. **Decommission only after rollback window closes** — premature decommission = no safety net

## Timeline Template

| Milestone | Typical Duration | Dependencies |
|---|---|---|
| Dev Validation | 2-6 weeks | Migration team |
| UAT Setup | 2-3 days | Platform team |
| UAT Testing | 1 week | Migration + Business teams |
| Prod Agent | 1-2 days | Infrastructure team |
| Prod Secrets | 2-3 days | Security + Platform teams |
| Scheduler | 3-5 days | Operations team |
| Cutover | 1 day + 2 week rollback window | All teams |

**Total: 4-9 weeks from dev complete to fully decommissioned**

## Anti-Patterns

- ❌ "Dev works, let's go to prod" — skipping UAT
- ❌ Configuring prod secrets months before go-live (they expire)
- ❌ No parallel run — "we'll just switch over"
- ❌ Decommissioning ETL on cutover day (no rollback possible)
- ❌ Treating scheduler integration as a 30-minute task (it's 3-5 days)
