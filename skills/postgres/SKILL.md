---
name: postgres
description: |
  Optimize and troubleshoot PostgreSQL databases including query tuning,
  vacuuming, replication, and connection management. Use when working with
  Postgres databases, diagnosing slow queries, tuning VACUUM, managing WAL
  and replication, or troubleshooting connections. Trigger with phrases like
  "Postgres query", "PostgreSQL index", "slow Postgres", "VACUUM", "pg_stat",
  "WAL tuning", "PlanetScale Postgres", "pscale CLI".
allowed-tools: Read, Bash
version: 1.0.0
author: PlanetScale <skills@planetscale.com>
license: MIT
---

# PostgreSQL

Diagnose, optimize, and operate PostgreSQL databases using local reference documentation and evidence-based validation.

## Contents

- [Overview](#overview) | [Prerequisites](#prerequisites) | [Instructions](#instructions)
- [Output](#output) | [Error Handling](#error-handling) | [Examples](#examples) | [Resources](#resources)

## Overview

Analyze and optimize the full PostgreSQL stack: schema design, indexing, query patterns, partitioning, MVCC, vacuuming, WAL, replication, monitoring, backup, and PlanetScale-specific features. Validate changes with `EXPLAIN ANALYZE` before production.

## Prerequisites

- PostgreSQL 13+ (note version-specific features)
- Access to run `EXPLAIN ANALYZE` on target queries
- For PlanetScale Postgres: `pscale` CLI installed and authenticated

## Instructions

### Step 1: Identify the Problem

1. Determine the category: schema/design, query performance, operational, or PlanetScale-specific.

### Step 2: Read Relevant References

**Schema and indexing:**
Read `{baseDir}/references/schema-design.md`, `{baseDir}/references/indexing.md`, `{baseDir}/references/index-optimization.md`, `{baseDir}/references/partitioning.md`

**Query patterns:**
Read `{baseDir}/references/query-patterns.md`, `{baseDir}/references/optimization-checklist.md`

**MVCC and transactions:**
Read `{baseDir}/references/mvcc-vacuum.md`, `{baseDir}/references/mvcc-transactions.md`

**Operations:** Read `{baseDir}/references/process-architecture.md`, `{baseDir}/references/memory-management-ops.md`, `{baseDir}/references/wal-operations.md`, `{baseDir}/references/replication.md`, `{baseDir}/references/storage-layout.md`, `{baseDir}/references/monitoring.md`, `{baseDir}/references/backup-recovery.md`

**PlanetScale-specific:** Read `{baseDir}/references/ps-connection-pooling.md`, `{baseDir}/references/ps-extensions.md`, `{baseDir}/references/ps-connections.md`, `{baseDir}/references/ps-insights.md`, `{baseDir}/references/ps-cli-commands.md`, `{baseDir}/references/ps-cli-api-insights.md`

### Step 3: Propose the Fix

1. State root cause with `EXPLAIN ANALYZE` or `pg_stat_*` evidence.
2. Describe the proposed change and trade-offs.
3. Alternatively, suggest a less invasive approach when the fix has high risk.
4. Provide rollback steps for production changes.

### Step 4: Validate with Evidence

1. Run `EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)` before and after.
2. Alternatively, use `pg_stat_statements` to verify aggregate improvement.
3. Check `pg_stat_*` views to confirm operational fixes.

## Output

1. Root cause with `EXPLAIN ANALYZE` or `pg_stat_*` evidence
2. Proposed change with rationale
3. Before/after comparison
4. Trade-offs, rollback steps, and post-deploy verification

## Error Handling

- **Connection failures:** Check SSL, limits, pooler config. Read `{baseDir}/references/ps-connections.md`.
- **OOM kills:** Review `work_mem`, `shared_buffers`, connection count. Read `{baseDir}/references/memory-management-ops.md`.
- **Vacuum stalled:** Check for long transactions blocking vacuum. Read `{baseDir}/references/mvcc-vacuum.md`.
- **XID wraparound:** Urgent — run manual vacuum. Read `{baseDir}/references/mvcc-transactions.md`.
- **Replication lag:** Check WAL sender/receiver. Read `{baseDir}/references/replication.md`.
- **Serialization failures:** Retry the transaction. Alternatively, switch to `READ COMMITTED` if serializable is not required.

## Examples

**Example: index audit**

```sql
-- Step 1: Read {baseDir}/references/index-optimization.md
-- Step 2: Query unused indexes from pg_stat_user_indexes
SELECT indexrelname, idx_scan FROM pg_stat_user_indexes WHERE idx_scan = 0;
-- Step 3: Drop confirmed-unused indexes
-- Step 4: Read {baseDir}/references/indexing.md for missing patterns
```

**Example: slow query diagnosis**

```sql
-- Step 1: Read {baseDir}/references/query-patterns.md
-- Step 2: EXPLAIN (ANALYZE, BUFFERS) on the slow query
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM events WHERE user_id = 123 AND created_at > '2025-01-01';
-- Step 3: Propose composite index
CREATE INDEX idx_events_user_created ON events(user_id, created_at);
```

## Resources

All reference files in `{baseDir}/references/`: schema-design, indexing, index-optimization, partitioning, query-patterns, optimization-checklist, mvcc-vacuum, mvcc-transactions, process-architecture, memory-management-ops, wal-operations, replication, storage-layout, monitoring, backup-recovery, ps-connection-pooling, ps-extensions, ps-connections, ps-insights, ps-cli-commands, ps-cli-api-insights (21 files).
