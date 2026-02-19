---
name: vitess
description: |
  Configure and optimize Vitess deployments for horizontal MySQL scaling.
  Use when working with Vitess databases, sharding, VSchema configuration,
  keyspace management, or MySQL scaling issues. Trigger with phrases like
  "Vitess", "VSchema", "VTGate", "keyspace", "shard", "vindex",
  "PlanetScale sharding", "cross-shard query", "Vitess migration".
allowed-tools: Read, Bash
version: 1.0.0
author: PlanetScale <skills@planetscale.com>
license: MIT
---

# Vitess

Plan, configure, and optimize Vitess deployments for horizontal MySQL scaling.

## Contents

- [Overview](#overview) | [Prerequisites](#prerequisites) | [Instructions](#instructions)
- [Output](#output) | [Error Handling](#error-handling) | [Examples](#examples) | [Resources](#resources)

## Overview

Vitess is a MySQL-compatible, cloud-native database system built at YouTube. PlanetScale runs Vitess as a managed service. Core capabilities: horizontal sharding transparent to applications, VTGate connection pooling, automatic HA with failure detection, query rewriting and caching, cross-shard schema management, and materialized views via VStream.

Key concepts: **Keyspace** (logical database), **Shard** (horizontal partition), **VSchema** (table-to-shard mapping), **Vindex** (sharding function), **VTGate** (query router), **Online DDL** (non-blocking migrations).

PlanetScale specifics: Git-like branching, deploy requests for schema changes, MySQL protocol on port 3306/443, SSL required.

## Prerequisites

- Vitess cluster or PlanetScale database access
- MySQL-compatible client or `pscale` CLI
- Understanding of keyspace topology and sharding strategy

## Instructions

### Step 1: Understand Architecture

1. Read `{baseDir}/references/architecture.md` for VTGate, VTTablet, Topology Service, VTOrc, and request flow.

### Step 2: Configure VSchema

1. Read `{baseDir}/references/vschema.md` for table-to-shard mapping, vindex selection, sequences, and routing rules.
2. Alternatively, use lookup vindexes for non-primary-key access patterns.

### Step 3: Optimize Query Routing

1. Read `{baseDir}/references/query-serving.md` for single-shard vs scatter-gather routing, compatibility, and `EXPLAIN` usage.
2. Filter by vindex column to avoid scatter queries. Alternatively, create materialized views for cross-shard access patterns.

### Step 4: Plan Schema Changes

1. Read `{baseDir}/references/schema-changes.md` for Online DDL strategies (`vitess`, `gh-ost`, `pt-osc`), migration lifecycle, and monitoring.

### Step 5: Data Migration

1. Read `{baseDir}/references/vreplication.md` for MoveTables, Reshard, Materialize, VDiff, and VStream.
2. Run VDiff to verify data integrity after migration.

## SQL Compatibility

Standard DML, DDL, joins, subqueries, CTEs (recursive in v21+), window functions all work. Limitations: no stored procedures/triggers through VTGate, no `LOCK TABLES`/`GET_LOCK`, cross-shard `SELECT FOR UPDATE` not atomic, cross-shard joins are scatter-gather (filter by vindex), foreign keys limited (prefer app-level integrity), `AUTO_INCREMENT` per-shard (use vindexes or app-generated IDs).

## Output

1. Current state analysis (keyspace topology, VSchema, query patterns)
2. Proposed change with rationale
3. `EXPLAIN` output showing query routing
4. Migration plan with rollback steps

## Error Handling

- **Cross-shard scatter:** Add vindex column filters. Read `{baseDir}/references/query-serving.md`. Alternatively, restructure queries to target single shards.
- **VSchema misconfiguration:** Verify vindex definitions match column types. Read `{baseDir}/references/vschema.md`.
- **Migration failures:** Troubleshoot via migration status checks. Read `{baseDir}/references/schema-changes.md`.
- **Data inconsistency:** Run VDiff to diagnose. Read `{baseDir}/references/vreplication.md`.
- **Unsupported SQL:** Rewrite stored procedures as application logic. Rewrite correlated subqueries as joins.

## Examples

**Example: VSchema sharding**

```sql
-- Scenario: Shard orders table by customer_id
-- Step 1: Read {baseDir}/references/vschema.md
-- Step 2: Define primary vindex (hash on customer_id)
-- Step 3: Add sequence table for order_id auto-increment
-- Step 4: Read {baseDir}/references/vreplication.md for Reshard workflow
-- Step 5: Run VDiff after migration to verify integrity
```

**Example: query routing analysis**

```sql
-- Slow query on sharded keyspace (no vindex filter = scatter)
EXPLAIN SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at LIMIT 100;
-- Fix: Add customer_id filter for single-shard routing
-- Alternative: Create materialized view for status-based lookups
```

## Resources

Reference files in `{baseDir}/references/`: vschema, schema-changes, vreplication, architecture, query-serving (5 files).
