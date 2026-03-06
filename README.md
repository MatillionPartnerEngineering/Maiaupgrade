# Matillion ETL → Data Productivity Cloud  
## Governed Migration Framework (Maia-Managed)

This repository contains a **governed, auditable framework** for migrating customers from **Matillion ETL** to **Matillion Data Productivity Cloud (DPC)**.

The framework is designed to be operated by **Maia**, Matillion’s Migration Project Manager LLM, and enforces strict separation between:

- Discovery  
- Refactor  
- Validation  
- Execution  

It supports **large-scale, multi-workload migrations** with clear gating rules, human oversight, and deterministic outcomes.

---

## 🧠 Primary Entry Point (Read First)

### .matillion/maia/rules/migration_manager_instructions.md ⭐

This file is the **single source of operational truth**.

It defines:

- Maia’s role and authority  
- Mandatory project structure  
- Phase sequencing and gating  
- What Maia may and may not do  
- When user approval is required  
- How validation, refactor, and execution interact  

**If you read only one file, read this one.**

---

## 📁 Repository Structure

```plaintext
.matillion/maia/
├── rules/
│   └── migration_manager_instructions.md    <-- Maia's Operational Brain
└── skills/
    ├── mass_validation.md                  <-- Validation Logic
    ├── migration_documentation.md          <-- Refactor Authority
    └── migration_strategy_and_plan_template.md

migration_project/
├── customer_migration_workspace/           <-- Customer-related assets
│   ├── pipeline_component_inventory.md
│   ├── MAUD.md
│   ├── shared_jobs.md
│   └── refactor_components.md
└── validation_reports/                     <-- Detailed validation reports
```

## 📘 Core Files Explained

### migration_manager_instructions.md  
**The operational brain of the framework**

- Defines who Maia is and how it behaves  
- Governs all phases of the migration lifecycle  
- Enforces read-only discovery and explicit user approval gates  
- Prevents silent or implicit refactor  
- Controls validation and execution sequencing  
- Updates strategy and tracker files automatically  
- Maintains the **To Do (Next Actions)** section in the migration plan  

---

### migration_strategy_and_plan_template.md  
**The live project ledger**

Includes:

- Project Progress Dashboard with % progress bars  
- Phase-by-phase tracking  
- Interactive Workload Migration Tracker  
- Explicit gating rules (Blockers, Secrets, Successful Run)  
- A concise **To Do (Next Actions)** section actively maintained by Maia  

This file answers:

**“What is the current state of this migration, and what should happen next?”**

---

### migration_documentation.md  
**The refactor authority**

- Lists every supported and unsupported component type  
- Defines conditions that require refactor  
- Documents approved refactor paths  
- Contains all authoritative Upgrade sections:
  - Python / Jython  
  - Bash  
  - API Extract / API Query  
  - Database Query / JDBC  
  - dbt  
  - Variables / Automatic Variables  
  - Iterators  
  - Temporary Tables  
  - Transactions  
  - Text Output  
  - Filter (Databricks)  
  - Replicate  

⚠️ **Neither Maia nor users invent refactor logic.**  
All refactor guidance must originate from this file.

---

### pipeline_component_inventory.md
**Ground truth for component presence, location, and migration classification**

Contains every component across all pipelines with:

- Pipeline path and component name
- Component ID and usage count
- Migration Type classification (Type 1 / Type 2 / Type 3)
- OOM risk scores for Python/Bash components (when enriched by FDE Context Engine)
- Refactor approach recommendations for Type 3 components

Used to:

- Locate components by exact pipeline path
- Detect Python, Jython, Bash, API, dbt, JDBC usage
- Identify shared pipelines (Type 3 `Unknown Component` entries)
- Identify ingestion and output systems
- Tie refactor findings to concrete, auditable locations

This file powers both **refactor discovery** and **validation detection**.

> **Note:** This file supersedes the legacy `component_details.csv`. The markdown format provides richer context (Type classification, risk scores) that Maia can use directly during discovery and validation.

---

### refactor_components.md  
**Single source of truth for refactor work**

Every refactor entry includes:

- Workload name  
- Component name  
- Pipeline location  
- Severity: Blocker / Warning / Advisory  
- Status: Pending / In Progress / Completed  
- Auto-link to the exact Upgrade section in migration_documentation.md  
- Referenced secrets (if applicable)  

Refactor behavior:

- Discovered by Maia  
- Performed by the user  
- Tracked and gated here  

Validation and execution are blocked by unresolved **Blockers**.

---

### mass_validation.md  
**Read-only validation rules**

Validation:

- Never performs refactor  
- Detects refactor-required conditions  
- Updates refactor_components.md  
- Generates immutable validation reports  

Detection logic explicitly references conditions defined in  
migration_documentation.md.

Validation output is **evidence**, not instruction.

---

## 📄 validation_reports  
**Immutable execution evidence**

For each workload execution, Maia generates:

validation_reports/<WORKLOAD>_Validation_Report.md

Each report includes:

- Validation scope: Pipelines, components, and checks executed  
- Failures and warnings: Blocking and non-blocking issues  
- Refactor conditions detected: Aligned to migration_documentation.md  
- Links back to refactor_components.md for traceability  
- Clear pass/fail signal indicating eligibility for a Successful Run  

Reports are append-only and must never be overwritten.

---

## 🔁 How the Framework Is Used (High-Level Flow)

### 1. Initialize
- Confirm customer and initial workload  
- Validate required files are present  

### 2. Discovery (Read-Only)
- Shared pipelines and dependencies  
- Assets and environments  
- Refactor detection only  

### 3. Refactor (User-Performed)
- Guided by Maia  
- Governed by migration_documentation.md  
- Tracked in refactor_components.md  
- All Blockers must be completed before validation  

### 4. Validation
- MassValidation rules applied  
- Validation report generated  
- Refactor conditions may be identified but not fixed  

### 5. Execution
- End-to-end pipeline run  
- **Successful Run** is the final authority for completion  

---

## 🚀 Getting Started (Internal Use)

1. Clone this repository  
2. Read migration_manager_instructions.md first  
3. Initialize customer_migration_workspace with customer artifacts  
4. Allow Maia to guide the migration  

---

Maintained by **services@matillion.com**
