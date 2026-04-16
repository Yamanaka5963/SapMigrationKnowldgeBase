# 35 - SAP HANA System Views Reference

## Overview

SAP HANA exposes metadata, monitoring, and administrative information through **System Views** — read-only virtual tables in the `SYS` schema. These are essential for Basis engineers and DBAs managing an S/4HANA landscape.

System views are accessed via SQL (HANA Studio, HANA Cockpit, or DBeaver connected to HANA). They always sit in the `SYS` schema.

```sql
SELECT * FROM SYS.M_TABLES WHERE SCHEMA_NAME = 'SAPHANADB';
```

## Key System View Categories

### Schema & Table Information

| View | Description |
|---|---|
| `SYS.SCHEMAS` | All schemas in the HANA database |
| `SYS.TABLES` | All tables (type, schema, record count metadata) |
| `SYS.TABLE_COLUMNS` | Column definitions for all tables |
| `SYS.VIEWS` | All views |
| `SYS.INDEXES` | All indexes |
| `SYS.M_TABLES` | Runtime table info: memory size, record count, table type |
| `SYS.M_TABLE_PERSISTENCE_STATISTICS` | Table persistence stats (disk vs. memory) |

### Memory & Performance Monitoring

| View | Description |
|---|---|
| `SYS.M_MEMORY_OVERVIEW` | Overall HANA memory usage summary |
| `SYS.M_HOST_MEMORY` | Memory usage per host |
| `SYS.M_CS_TABLES` | Column store table details — memory consumption, compression |
| `SYS.M_RS_TABLES` | Row store table details |
| `SYS.M_TABLE_COLUMNS_STATISTICS` | Column-level statistics (distinct values, compression ratio) |
| `SYS.M_EXPENSIVE_STATEMENTS` | Recently executed expensive SQL statements |
| `SYS.M_SQL_PLAN_CACHE` | SQL plan cache — executed statements and performance |

### Sessions & Connections

| View | Description |
|---|---|
| `SYS.M_CONNECTIONS` | Active database connections |
| `SYS.M_SESSIONS` | Active sessions |
| `SYS.M_BLOCKED_TRANSACTIONS` | Blocked transactions — useful for deadlock investigation |
| `SYS.M_LOCK_WAITS_STATISTICS` | Lock wait history |

### Services & Landscape

| View | Description |
|---|---|
| `SYS.M_SERVICES` | Running HANA services (indexserver, nameserver, etc.) |
| `SYS.M_HOST_INFORMATION` | Host-level system info (CPU, memory, OS) |
| `SYS.M_SYSTEM_OVERVIEW` | High-level system health dashboard |
| `SYS.M_LANDSCAPE_HOST_CONFIGURATION` | Scale-out landscape topology |
| `SYS.M_DATABASE` | Database-level information (system ID, version, start time) |

### Backup & Recovery

| View | Description |
|---|---|
| `SYS.M_BACKUP_CATALOG` | Backup history and catalog |
| `SYS.M_BACKUP_CATALOG_FILES` | Physical backup file locations |
| `SYS.M_LOG_SEGMENTS` | Redo log segment status |

### Users & Security

| View | Description |
|---|---|
| `SYS.USERS` | All database users |
| `SYS.ROLES` | All roles |
| `SYS.GRANTED_ROLES` | Role assignments per user |
| `SYS.GRANTED_PRIVILEGES` | Privilege assignments per user |
| `SYS.M_PASSWORD_POLICY` | Current password policy settings |
| `SYS.AUDIT_LOG` | Audit trail (if audit policy is active) |

## SAP Application Schema in S/4HANA

The S/4HANA application data (ABAP workload tables like `ACDOCA`, `MATDOC`, etc.) lives in the **SAP application schema** — typically named after the SAP system ID (SID), e.g., `SAPHANADB` or `SAP<SID>`.

```sql
-- List all schemas
SELECT SCHEMA_NAME FROM SYS.SCHEMAS;

-- Find S/4HANA application tables in the app schema
SELECT TABLE_NAME, RECORD_COUNT, TABLE_SIZE
FROM SYS.M_TABLES
WHERE SCHEMA_NAME = 'SAP<SID>'
ORDER BY TABLE_SIZE DESC;
```

## Useful Diagnostic Queries

### Top 20 tables by memory size
```sql
SELECT TOP 20 TABLE_NAME, SCHEMA_NAME,
       ROUND(TABLE_SIZE / 1024 / 1024, 2) AS SIZE_MB,
       RECORD_COUNT
FROM SYS.M_TABLES
WHERE SCHEMA_NAME = 'SAP<SID>'
ORDER BY TABLE_SIZE DESC;
```

### Currently running expensive statements
```sql
SELECT TOP 20 STATEMENT_STRING, DURATION_MICROSEC,
       START_TIME, USER_NAME
FROM SYS.M_EXPENSIVE_STATEMENTS
ORDER BY DURATION_MICROSEC DESC;
```

### Active blocking locks
```sql
SELECT * FROM SYS.M_BLOCKED_TRANSACTIONS;
```

## References

- SAP HANA SQL and System Views Reference (official PDF, HANA 2.0 SPS03): https://help.sap.com/doc/9b40bf74f8644b898fb07dabdd2a36ad/2.0.03/en-US/SAP_HANA_SQL_and_System_Views_Reference_en.pdf
- SAP Help — System Views Reference, HANA Platform (official): https://help.sap.com/docs/SAP_HANA_PLATFORM/4fe29514fd584807ac9f2a04f6754767/20cbb10c75191014b47ba845bfe499fe.html
- SAP Help — System Views Reference, HANA Cloud (official): https://help.sap.com/docs/hana-cloud-database/sap-hana-cloud-sap-hana-database-sql-reference-guide/system-views-reference
