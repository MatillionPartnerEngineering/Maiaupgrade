---
name: migration-python-credentials-refactor
description: Detect and refactor hardcoded credentials (passwords, API keys, tokens, connection strings) embedded inside Python Script component bodies. Covers secrets extraction to DPC Secret Manager and code refactoring patterns.
---

# Python Code Credential Refactor Skill

## When to Activate

- Python Script components contain hardcoded passwords, API keys, or tokens in script body
- Connection strings with embedded credentials (JDBC URLs, SQLAlchemy URIs)
- Bearer tokens, authorization headers with literal values
- Any `password =`, `api_key =`, `token =` assignments with literal strings

## Detection Rules

Scan Python Script component `script` parameter for these regex patterns:

| Pattern | Example | Severity |
|---------|---------|----------|
| `password\s*=\s*["'][^"']+["']` | `password = "MyP@ss123"` | 🔴 Critical |
| `api_key\s*=\s*["'][^"']+["']` | `api_key = "BnCkjep80F..."` | 🔴 Critical |
| `Bearer\s+[A-Za-z0-9._-]{20,}` | `"Bearer eyJhbG..."` | 🔴 Critical |
| `://[^:]+:[^@]+@` | `postgresql://user:pass@host` | 🔴 Critical |
| `token\s*=\s*["'][^"']+["']` | `token = "e7191a37-..."` | 🔴 Critical |
| `Authorization.*["'][^"']{20,}["']` | Header with long literal | 🟡 Warning |
| `secret\s*=\s*["'][^"']+["']` | `secret = "abc123"` | 🔴 Critical |

## Refactoring Strategy

### Step 1: Inventory Credentials

For each detected credential:
- Identify the **purpose** (API auth, database connection, SFTP, etc.)
- Determine a **secret reference name** (e.g., `vendor_api_key`, `reporting_db_password`)
- Note the **variable name** used in the script

### Step 2: Create Secrets in DPC

Customer action required:
1. Navigate to **Project Settings → Secrets**
2. Create each secret with the reference name
3. Set the actual credential value

### Step 3: Refactor Python Script

#### Pattern A: Using Pipeline Variables (Recommended)

**Before:**
```python
api_key = "BnCkjep80FqidzjsZ8uIvKqRi3uvN01P"
headers = {"Authorization": f"Bearer {api_key}"}
```

**After:**
1. Add a pipeline variable `v_vendor_api_key` (type: TEXT, visibility: SECRET)
2. Set the variable default to reference the DPC secret
3. In the Python script, use the variable via Matillion context:

```python
api_key = context.getVariable('v_vendor_api_key')
headers = {"Authorization": f"Bearer {api_key}"}
```

#### Pattern B: Using Snowflake Secrets (For Python Pushdown)

If the script runs as Python Pushdown:

```python
import _snowflake
api_key = _snowflake.get_generic_secret_string('my_secret_name')
```

Requires Snowflake secret object created via:
```sql
CREATE SECRET my_secret_name TYPE = GENERIC_STRING SECRET_STRING = '...';
```

#### Pattern C: Connection String Decomposition

**Before:**
```python
conn_str = "postgresql://admin:P@ssw0rd@prod-db.example.com:5432/mydb"
```

**After:**
```python
db_host = context.getVariable('v_db_host')
db_user = context.getVariable('v_db_user')
db_pass = context.getVariable('v_db_password')  # Secret-backed variable
db_name = context.getVariable('v_db_name')
conn_str = f"postgresql://{db_user}:{db_pass}@{db_host}:5432/{db_name}"
```

## Reporting Format

For each Python Script component with credentials, document:

| File | Component | Credential Type | Secret Name | Status |
|------|-----------|----------------|-------------|--------|
| `path/to/file.orch.yaml` | `API Call` | API Key | `vendor_api_key` | Pending |

## Common Credentials Found in This Project

| Type | Occurrences | Workloads |
|------|-------------|----------|
| Vendor API Key | 20+ refs / 7 files | Data Ingestion |
| Reporting API Key | 5+ refs | Reporting |
| SQL Server passwords | 10+ refs | Reporting, Analytics |
| Bearer JWT tokens | 3+ refs | Data Ingestion |
| Database connection strings | 15+ refs | Reporting, Analytics |

## Important Notes

- **Never log or print credential values** during refactor verification
- The `migration-credentials-audit` skill covers YAML-level secrets; THIS skill covers **code-level** credentials
- Pipeline variables with secret backing are resolved at runtime — they won't appear in logs
- Always verify the script still functions after refactor by running the pipeline in a test environment
- Some credentials may be shared across multiple scripts — use a single secret reference for deduplication
