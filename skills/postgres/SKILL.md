---
name: postgres
description: |
  PostgreSQL best practices, query optimization, connection troubleshooting,
  and performance tuning. Use when working with Postgres databases, diagnosing
  slow queries, tuning vacuuming, managing replication, or troubleshooting
  connections. Trigger with phrases like "Postgres query", "PostgreSQL index",
  "slow Postgres", "VACUUM", "pg_stat", "WAL tuning", "PlanetScale Postgres",
  "pscale CLI".
allowed-tools: Read, Bash
version: 1.0.0
author: PlanetScale <skills@planetscale.com>
license: MIT
---

# PostgreSQL

Diagnose, optimize, and operate PostgreSQL databases using local reference documentation and evidence-based validation.

## Overview

This skill covers the full PostgreSQL stack: schema design, indexing, query
patterns, partitioning, MVCC, vacuuming, WAL, replication, monitoring, backup,
and PlanetScale-specific features. Every recommendation links to a local
reference file that Claude can read for full detail. Always validate changes
with `EXPLAIN ANALYZE` before applying to production.

## Prerequisites

- PostgreSQL 13+ (note version-specific feature availability)
- Access to run `EXPLAIN ANALYZE` on target queries
- `psql` or compatible client for interactive queries
- For PlanetScale Postgres: `pscale` CLI installed and authenticated

## Instructions

### Step 1: Identify the Problem Category

Determine which area of PostgreSQL the issue falls into:

- **Schema/design issues** — table structure, data types, constraints
- **Query performance** — slow queries, missing indexes, anti-patterns
- **Operational issues** — vacuum, replication, WAL, connections, monitoring
- **PlanetScale-specific** — connection pooling, extensions, CLI, insights

### Step 2: Read Relevant References

Load the reference files that match the problem area.

**Schema and indexing:**

- Read `{baseDir}/references/schema-design.md` for table design, PKs, data types, foreign keys
- Read `{baseDir}/references/indexing.md` for index types, composite indexes, performance
- Read `{baseDir}/references/index-optimization.md` for unused/duplicate index queries and audit
- Read `{baseDir}/references/partitioning.md` for large tables, time-series, data retention

**Query patterns and optimization:**

- Read `{baseDir}/references/query-patterns.md` for SQL anti-patterns, JOINs, pagination, batch queries
- Read `{baseDir}/references/optimization-checklist.md` for pre-optimization audit and readiness checks

**MVCC, vacuum, and transactions:**

- Read `{baseDir}/references/mvcc-vacuum.md` for dead tuples, long transactions, XID wraparound prevention
- Read `{baseDir}/references/mvcc-transactions.md` for isolation levels, XID wraparound, serialization errors

**Architecture and operations:**

- Read `{baseDir}/references/process-architecture.md` for multi-process model, connection pooling, auxiliary processes
- Read `{baseDir}/references/memory-management-ops.md` for shared/private memory, OS page cache, OOM prevention
- Read `{baseDir}/references/wal-operations.md` for WAL internals, checkpoint tuning, durability, crash recovery
- Read `{baseDir}/references/replication.md` for streaming replication, slots, sync commit, failover
- Read `{baseDir}/references/storage-layout.md` for PGDATA structure, TOAST, fillfactor, tablespaces
- Read `{baseDir}/references/monitoring.md` for pg_stat views, logging, pg_stat_statements, host metrics
- Read `{baseDir}/references/backup-recovery.md` for pg_dump, pg_basebackup, PITR, WAL archiving

**PlanetScale-specific:**

- Read `{baseDir}/references/ps-connection-pooling.md` for PgBouncer, pool sizing, pooled vs direct connections
- Read `{baseDir}/references/ps-extensions.md` for supported extensions and compatibility
- Read `{baseDir}/references/ps-connections.md` for connection troubleshooting, drivers, SSL
- Read `{baseDir}/references/ps-insights.md` for slow query analysis, MCP server, pscale CLI insights
- Read `{baseDir}/references/ps-cli-commands.md` for pscale CLI reference: branches, deploy requests, auth
- Read `{baseDir}/references/ps-cli-api-insights.md` for query insights via `pscale api` and schema analysis

### Step 3: Propose the Fix

Propose the smallest change that solves the problem:

1. State the root cause with evidence from `EXPLAIN ANALYZE` or `pg_stat_*` views
2. Describe the proposed change and its trade-offs
3. Include rollback steps for production changes
4. Provide post-deploy verification queries

### Step 4: Validate with Evidence

Run `EXPLAIN ANALYZE` before and after the change to confirm improvement:

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) <query>;
```

For operational changes, check relevant `pg_stat_*` views to confirm the fix.

## Output

Structure all recommendations with:
1. Root cause analysis backed by `EXPLAIN ANALYZE` or `pg_stat_*` evidence
2. The proposed change with rationale
3. Before/after `EXPLAIN ANALYZE` comparison
4. Trade-offs and risks
5. Rollback steps for production changes

## Error Handling

- **Connection failures:** Check SSL requirements, connection limits, and pooler configuration. Read `{baseDir}/references/ps-connections.md`.
- **OOM kills:** Review `work_mem`, `shared_buffers`, and connection count. Read `{baseDir}/references/memory-management-ops.md`.
- **Vacuum not running:** Check for long-running transactions blocking vacuum. Read `{baseDir}/references/mvcc-vacuum.md`.
- **XID wraparound warnings:** Urgent — run manual vacuum on affected tables. Read `{baseDir}/references/mvcc-transactions.md`.
- **Replication lag:** Check WAL sender/receiver status and network. Read `{baseDir}/references/replication.md`.
- **Serialization failures:** Retry the transaction. Read `{baseDir}/references/mvcc-transactions.md`.

## Examples

### Index Audit

```
User: Our Postgres database has 200+ indexes and queries are still slow.

1. Read {baseDir}/references/index-optimization.md
2. Run the unused index query from the reference
3. Run the duplicate index query from the reference
4. Drop confirmed-unused indexes
5. Read {baseDir}/references/indexing.md for missing index patterns
6. Run EXPLAIN ANALYZE on the slow queries to identify gaps
```

### Slow Query Diagnosis

```
User: This query takes 8 seconds:
  SELECT * FROM events WHERE user_id = 123 AND created_at > '2025-01-01' ORDER BY created_at;

1. Read {baseDir}/references/query-patterns.md
2. Run EXPLAIN (ANALYZE, BUFFERS) on the query
3. Read {baseDir}/references/indexing.md
4. Propose: CREATE INDEX idx_events_user_created ON events(user_id, created_at)
5. Re-run EXPLAIN ANALYZE to confirm improvement
```

## Resources

All reference files are in `{baseDir}/references/`:

**Generic PostgreSQL:**

| File | Topic |
| --- | --- |
| `schema-design.md` | Tables, PKs, data types, foreign keys |
| `indexing.md` | Index types, composite indexes |
| `index-optimization.md` | Unused/duplicate index audit |
| `partitioning.md` | Large tables, time-series |
| `query-patterns.md` | SQL anti-patterns, pagination |
| `optimization-checklist.md` | Pre-optimization audit |
| `mvcc-vacuum.md` | Dead tuples, vacuum tuning |
| `mvcc-transactions.md` | Isolation, XID wraparound |
| `process-architecture.md` | Multi-process model |
| `memory-management-ops.md` | Memory layout, OOM prevention |
| `wal-operations.md` | WAL, checkpoints, crash recovery |
| `replication.md` | Streaming replication, failover |
| `storage-layout.md` | PGDATA, TOAST, fillfactor |
| `monitoring.md` | pg_stat views, logging |
| `backup-recovery.md` | pg_dump, PITR, WAL archiving |

**PlanetScale-specific:**

| File | Topic |
| --- | --- |
| `ps-connection-pooling.md` | PgBouncer, pool sizing |
| `ps-extensions.md` | Supported extensions |
| `ps-connections.md` | Connection troubleshooting, SSL |
| `ps-insights.md` | Slow query analysis, MCP server |
| `ps-cli-commands.md` | pscale CLI reference |
| `ps-cli-api-insights.md` | Query insights via pscale API |
