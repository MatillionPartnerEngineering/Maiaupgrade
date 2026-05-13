---
name: migration-database-query
description: Migrate Database Query components to DPC, including JDBC driver setup,
  manifest configuration, and connection migration.
schema_version: 1
phases:
- refactor
- validation
detection_rules:
- id: database-query-migration
  title: Database Query / JDBC component migration
  reference: .matillion/maia/skills/migration-database-query/SKILL.md
  body_anchor: database-query-migration
  severity: blocker
  applies_when:
    component_types:
    - database-query
---

<a id="database-query-migration"></a>
# Upgrade: Database Query

Reference: https://docs.matillion.com/metl/docs/migration-database-query/

DPC has a **Database Query** component equivalent to Matillion ETL’s Database Query component.

- Vendor restrictions prevent shipping drivers for every database.
- DPC supports uploading additional drivers via a mechanism.

## Upgrade path

The source database determines the upgrade path.

### Upgraded automatically

Database Query components with the following sources migrate automatically:

- Amazon Redshift
- IBM DB2 for i
- MariaDB
- Microsoft SQL Server
- Oracle
- PostgreSQL
- Sybase ASE
- Snowflake
- SQL Server (Microsoft driver)

### Upgrade with a driver upload (Hybrid SaaS only)

If not supported by default:

- The component is automatically changed to **JDBC** and properties configured where possible.
- You must upload a suitable JDBC driver for each source.
- You must create a **manifest file** per JDBC component documentation.

> **Note**  
> This option is available for **Hybrid SaaS deployments only**.

### MySQL

- DPC can’t ship/use the official MySQL driver.
- Uses MariaDB driver (compatible).
- MySQL Database Query components are changed to refer to MariaDB driver during migration.
- If you need the official MySQL driver:
  - Use JDBC and provide your own driver.
