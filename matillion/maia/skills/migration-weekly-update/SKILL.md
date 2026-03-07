# 📧 Template: Weekly Migration Status Update

## Purpose
Generate a consistent, stakeholder-friendly weekly status update for the Matillion ETL → DPC migration project.

## Instructions for Maia

1. **Reference Files:**
   - `customer_migration_workspace/[CUSTOMER]_Migration_Strategy.md` - phases, blockers, workload tracker
   - `validation_reports/` - workload-specific validation results
   - `customer_migration_workspace/refactor_components.md` - refactor tracking

2. **Data Sources:**
   - Pull successful run count from Interactive Workload Migration Tracker
   - Pull blocker details from To Do section and workload status column
   - Calculate week-over-week delta for progress metrics

3. **Output Format:** Copy the template below and populate with current data

---

# Weekly Migration Status Update

## 📝 Subject Line
`Migration Update: Matillion ETL → DPC | [Customer] | Week of [DATE]`

---

## 🔎 Executive Summary

> *[Maia: 2-3 sentences summarizing migration health, key wins, and primary focus area]*

**Example:**
> Migration is progressing well with 50% of workloads now running successfully end-to-end. Six new workloads achieved successful runs this week. Primary focus remains on resolving AWS IAM cross-account permissions and JDBC driver configuration.

---

## 📊 Dashboard

| Metric | This Week | Last Week | Delta |
|--------|-----------|-----------|-------|
| **Overall Completion** | [X]% | [Y]% | +[Z]% |
| **Successful Runs** | [A]/[B] | [C]/[B] | +[D] |
| **Active Blockers** | [E] | [F] | [+/-G] |
| **Workloads In Progress** | [H] | [I] | [+/-J] |

**Progress Bar:**
`[■■■■■■■■■□] XX%`

---

## 🎉 Wins This Week

*[Maia: List 3-5 key achievements from the past week]*

| # | Achievement | Impact |
|---|-------------|--------|
| 1 | **[Workload] achieved successful run** | [Brief impact description] |
| 2 | **[Blocker] resolved** | Unblocked [X] workloads |
| 3 | **[Milestone] completed** | [Brief impact description] |
| 4 | **[Task] finished** | [Brief impact description] |
| 5 | **[Validation] passed** | [Brief impact description] |

---

## 🚀 Workload Status Summary

### By Status Category

| Status | Count | Workloads |
|--------|-------|----------|
| ✅ **Successful Run** | [X] | [List first 5, then "+ N more"] |
| 🔴 **Blocked** | [Y] | [List first 5, then "+ N more"] |
| ⚠️ **In Progress** | [Z] | [List all] |
| 🟦 **Archived** | [W] | [List all] |

### New Successful Runs This Week

| Workload | Date | Notes |
|----------|------|-------|
| [WORKLOAD_1] | [DATE] | [Brief description of resolution] |
| [WORKLOAD_2] | [DATE] | [Brief description of resolution] |

---

## 🛑 Active Blockers

*[Maia: List blockers by category, with owner and required action]*

### 🔴 Infrastructure/IAM (Requires Infra Team)

| Blocker | Affected Workloads | Owner | Action Required |
|---------|-------------------|-------|----------------|
| [Blocker description] | [List] | [Name/Team] | [Specific action] |

### 🟡 Configuration (Requires DPC Admin)

| Blocker | Affected Workloads | Owner | Action Required |
|---------|-------------------|-------|----------------|
| [Blocker description] | [List] | [Name/Team] | [Specific action] |

### 🟠 External Dependencies (Requires Vendor/Support)

| Blocker | Affected Workloads | Owner | Action Required |
|---------|-------------------|-------|----------------|
| [Blocker description] | [List] | [Name/Team] | [Specific action] |

---

## ✅ Next Week Focus

*[Maia: Pull from To Do section of Migration Strategy, prioritized]*

### Priority 1: Critical Path
- [ ] **[Action]** - [Brief description] (Owner: [Name])
- [ ] **[Action]** - [Brief description] (Owner: [Name])

### Priority 2: High Impact
- [ ] **[Action]** - [Brief description] (Owner: [Name])
- [ ] **[Action]** - [Brief description] (Owner: [Name])

### Priority 3: Maintenance
- [ ] **[Action]** - [Brief description] (Owner: [Name])

---

## 📅 Key Dates & Milestones

| Milestone | Target Date | Status |
|-----------|-------------|--------|
| 50% Successful Runs | [DATE] | ✅ Achieved / 🔄 In Progress |
| 75% Successful Runs | [DATE] | ⬜ Pending |
| 100% Successful Runs | [DATE] | ⬜ Pending |
| UAT Sign-off | [DATE] | ⬜ Pending |
| Production Cutover | [DATE] | ⬜ Pending |

---

## 🆘 Escalations & Risks

*[Maia: Only include if there are items requiring leadership attention]*

| Risk/Escalation | Impact | Mitigation | Owner |
|-----------------|--------|------------|-------|
| [Description] | [High/Medium/Low] | [Proposed action] | [Name] |

---

## 📎 Attachments & References

- Migration Strategy: `customer_migration_workspace/[CUSTOMER]_Migration_Strategy.md`
- Validation Reports: `validation_reports/`
- Refactor Tracking: `customer_migration_workspace/refactor_components.md`

---

## Template Usage Notes

### For Maia:
1. **Always calculate deltas** - stakeholders want to see week-over-week progress
2. **Be specific on blockers** - include error messages, dates, and owners
3. **Highlight wins first** - set positive tone before discussing blockers
4. **Keep executive summary brief** - 2-3 sentences maximum
5. **Use consistent status icons:**
   - ✅ Complete/Success
   - 🔴 Blocked
   - ⚠️ In Progress/Warning
   - 🟡 Pending Action
   - 🟦 Archived/Out of Scope

### For Users:
- Request weekly update with: *"Generate weekly update for [CUSTOMER] migration"*
- Specify date range if needed: *"Generate weekly update for week of [DATE]"*
- Request specific sections: *"Just give me the blocker summary"*