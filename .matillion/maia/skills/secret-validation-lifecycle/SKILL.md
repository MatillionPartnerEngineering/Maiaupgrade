---
name: secret-validation-lifecycle
description: Three-phase secret management for migrations - Inventory, Create/Register, and Runtime Validate. Use when managing credentials during ETL-to-DPC migrations. A secret is not complete until a pipeline successfully authenticates with it.
---

# Secret Validation Lifecycle

## Core Principle

**A secret is NOT complete until a pipeline successfully authenticates with it at runtime.** Existence in the secret store ≠ working credential.

## Three Phases

### Phase 1: Inventory (During Discovery)

**When:** Before migration begins, during pipeline analysis.

**Actions:**
1. Scan all pipelines for credential references (connection strings, API keys, OAuth tokens)
2. Catalog every unique secret required
3. Identify secret type (password, API key, OAuth client/secret, certificate, token)
4. Map secrets to workloads (which workloads need which secrets)
5. Identify shared secrets (one credential used by 5+ workloads)

**Output:** Secret inventory table

| Secret Name | Type | Source System | Workloads Using | Owner | Status |
|---|---|---|---|---|---|
| TICKETING_API_TOKEN | API Key | Ticketing System | TICKETING_MASTER, TICKETING_WEEKEND | Platform Team | Identified |
| CRM_OAUTH_CLIENT | OAuth | CRM Platform | CRM_MASTER | CRM Team | Identified |

### Phase 2: Create & Register (During Setup)

**When:** After environment provisioning, before workload testing.

**Actions:**
1. Create secrets in cloud provider (AWS Secrets Manager, Azure Key Vault, etc.)
2. Register secret references in DPC
3. Verify secret reference names match pipeline expectations
4. Document any name changes from ETL → DPC

**Status progression:** Identified → Created → Registered

**Common pitfalls:**
- Secret name in DPC doesn't match pipeline reference (case-sensitive!)
- Secret value has trailing whitespace or newline
- OAuth token expired between creation and first use
- Certificate format wrong (PEM vs DER)

### Phase 3: Runtime Validation (During Testing)

**When:** During workload execution testing.

**Actions:**
1. Run pipeline that uses the secret
2. Confirm successful authentication (not just "no error on secret lookup")
3. Verify data is actually returned/processed
4. Document validation date and pipeline used for validation

**Status progression:** Registered → Runtime Validated ✅

**Critical distinction:**
- ❌ "Secret exists in DPC" — NOT validated
- ❌ "Secret resolves without error" — NOT validated
- ✅ "Pipeline authenticated and retrieved data" — VALIDATED

## Tracking Template

| Secret | Type | Workloads | Phase 1 | Phase 2 | Phase 3 | Validated Date |
|---|---|---|---|---|---|---|
| JIRA_API_TOKEN | API Key | 2 | ✅ | ✅ | ✅ | 2026-03-15 |
| SF_OAUTH_CLIENT | OAuth | 3 | ✅ | ✅ | ❌ Expired | — |
| CISCO_API_KEY | API Key | 1 | ✅ | ⏳ Pending | — | — |

## Rules

1. **Never mark a secret complete until Phase 3 passes** — existence ≠ validity
2. **OAuth tokens expire** — validate within 24 hours of creation, re-validate before go-live
3. **Shared secrets are highest priority** — one shared secret blocks 5-30 workloads
4. **Secret name mismatches are the #1 config blocker** — always verify case-sensitive match
5. **Certificates have the shortest shelf life** — validate last, closest to go-live
6. **Track expiration dates** — set reminders for OAuth refresh tokens and certificates

## Batch Validation Strategy

For migrations with 30+ secrets:
1. Group by type (API keys first — simplest, OAuth last — most complex)
2. Validate shared secrets before workload-specific ones
3. Run one workload per secret group to validate the batch
4. If one fails, check all in that group (likely same root cause)

## Common Failure Patterns

| Symptom | Root Cause | Fix |
|---|---|---|
| "Secret not found" | Name mismatch (case/spelling) | Correct reference name |
| "401 Unauthorized" | Value incorrect or expired | Rotate/refresh the credential |
| "403 Forbidden" | Credential valid but insufficient permissions | Update permissions in source system |
| "Certificate error" | Wrong format or expired cert | Re-export in correct format |
| "Token expired" | OAuth refresh token not configured | Set up token refresh flow |
