---
name: migration-python-library-compatibility
description: Detect Python library compatibility issues in migrated scripts. Covers available vs unavailable packages in DPC Python Pushdown (Snowpark), legacy library upgrades (boto→boto3), and ODBC/driver limitations.
---

# Python Library Compatibility Checker Skill

## When to Activate

- Python Script components use `import` statements for external libraries
- Scripts reference `sqlalchemy`, `pyodbc`, `boto`, `paramiko`, `pysftp`, or other third-party packages
- Python Pushdown scripts fail with `ModuleNotFoundError` or import errors
- Evaluating whether a Python script can run in Snowpark container runtime

## DPC Python Runtime Environments

### Python Pushdown (Snowpark Container Runtime)

**Available packages (confirmed):**

| Package | Version | Notes |
|---------|---------|-------|
| `requests` | 2.32.5 | HTTP client — works fully |
| `boto3` | 1.42.61 | AWS SDK — works fully |
| `snowflake-connector-python` | latest | Native Snowflake access |
| `pandas` | latest | Data manipulation |
| `numpy` | latest | Numerical computing |
| `sqlalchemy` | latest | SQL toolkit (but see ODBC note below) |
| `cryptography` | latest | Encryption/hashing |
| `json`, `csv`, `os`, `sys`, `re` | stdlib | Always available |
| `datetime`, `time`, `math` | stdlib | Always available |
| `urllib3`, `http` | stdlib | HTTP utilities |

**NOT available / problematic:**

| Package | Issue | Alternative |
|---------|-------|-------------|
| `pyodbc` | ODBC drivers not in Snowpark runtime | Use native Snowflake connector or JDBC component |
| `boto` (v1) | Deprecated, not installed | Upgrade to `boto3` |
| `cx_Oracle` | Oracle client not available | Use Database Query component with JDBC driver |
| `pymssql` | Not available in Snowpark | Use Microsoft SQL Input component |
| `pysftp` | May not be available | Use `paramiko` directly or Bash Pushdown |
| `mysql.connector` | Not confirmed | Use Database Query component with MySQL JDBC |
| Custom `.whl` packages | Cannot pip install at runtime | Pre-install via Snowpark package policy |

## Detection Procedure

1. **Extract imports** from all Python Script components:
   - Regex: `^\s*(import|from)\s+([a-zA-Z_][a-zA-Z0-9_.]*)`
   
2. **Classify each import:**
   - ✅ Available (no action)
   - ⚠️ Available but with limitations (document limitation)
   - 🔴 Not available (requires refactor)
   - 🟡 Deprecated (requires upgrade)

3. **Report findings** per component

## Common Refactor Patterns

### Pattern 1: `boto` → `boto3` Upgrade

**Before (boto v1 — deprecated):**
```python
import boto
conn = boto.connect_s3()
bucket = conn.get_bucket('my-bucket')
key = bucket.get_key('path/to/file.csv')
content = key.get_contents_as_string()
```

**After (boto3):**
```python
import boto3
s3 = boto3.client('s3')
response = s3.get_object(Bucket='my-bucket', Key='path/to/file.csv')
content = response['Body'].read().decode('utf-8')
```

### Pattern 2: `pyodbc` + `sqlalchemy` → Native Component

**Before:**
```python
import sqlalchemy
import pyodbc
engine = sqlalchemy.create_engine('mssql+pyodbc://user:pass@server/db?driver=ODBC+Driver+17')
df = pd.read_sql('SELECT * FROM table', engine)
```

**After (recommended):** Replace entire Python Script with:
- **Microsoft SQL Input v2** component (for SQL Server reads)
- **Database Query** component with uploaded JDBC driver (for other databases)
- Write results to Snowflake staging table, then continue pipeline

### Pattern 3: `paramiko` SFTP → Bash Pushdown or Python Pushdown

**If `paramiko` is available in Snowpark:**
```python
import paramiko
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect(hostname, username=user, password=pwd)
sftp = ssh.open_sftp()
sftp.get(remote_path, local_path)
```

**If not available:** Convert to Bash Pushdown with `sftp` or `scp` commands.

### Pattern 4: `pysftp` → `paramiko`

**Before:**
```python
import pysftp
with pysftp.Connection(host, username=user, password=pwd) as sftp:
    sftp.get(remote_file, local_file)
```

**After:**
```python
import paramiko
transport = paramiko.Transport((host, 22))
transport.connect(username=user, password=pwd)
sftp = paramiko.SFTPClient.from_transport(transport)
sftp.get(remote_file, local_file)
sftp.close()
transport.close()
```

## Reporting Format

| File | Component | Import | Status | Action Required |
|------|-----------|--------|--------|-----------------|
| `file.orch.yaml` | `Extract Data` | `boto` | 🟡 Deprecated | Upgrade to boto3 |
| `file.orch.yaml` | `Load SQL Server` | `pyodbc` | 🔴 Unavailable | Replace with MS SQL Input component |
| `file.orch.yaml` | `API Call` | `requests` | ✅ Available | None |

## Decision Framework: Script vs Native Component

| Scenario | Recommendation |
|----------|---------------|
| Read from external DB | Use Database Query / MS SQL Input component |
| Write to external DB | Python Pushdown with appropriate driver OR staged approach |
| REST API call | Python Pushdown with `requests` (confirmed available) |
| S3 file operations | Python Pushdown with `boto3` (confirmed available) |
| SFTP file transfer | Bash Pushdown with `sftp`/`scp` OR Python with `paramiko` |
| Complex data transformation | Python Pushdown with `pandas` |
| Simple data transformation | Use native DPC transformation components |

## Important Notes

- Always verify package availability in your specific Snowpark version before committing to a refactor path
- Snowpark package availability can vary by Snowflake account/region
- For packages not in the default Snowpark environment, check if they can be added via `CREATE FUNCTION ... PACKAGES = (...)` syntax
- Some packages have transitive dependencies that may also be unavailable
- When in doubt, test with a minimal Python Pushdown script: `import <package>; print('OK')`
