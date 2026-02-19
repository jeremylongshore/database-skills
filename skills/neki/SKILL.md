---
name: neki
description: |
  Evaluate and plan Postgres sharding readiness for PlanetScale Neki.
  Use when assessing whether a Postgres database is ready for horizontal
  sharding, planning schema changes for shard compatibility, or preparing
  a migration path to Neki. Trigger with phrases like "Neki", "shard Postgres",
  "Postgres sharding", "horizontal scaling Postgres", "sharding readiness".
allowed-tools: Read
version: 1.0.0
author: PlanetScale <skills@planetscale.com>
license: MIT
---

# PlanetScale Neki

Assess Postgres sharding readiness and plan for horizontal scaling with Neki.

## Contents

- [Overview](#overview) | [Prerequisites](#prerequisites) | [Instructions](#instructions)
- [Output](#output) | [Error Handling](#error-handling) | [Examples](#examples) | [Resources](#resources)

## Overview

Neki is a **sharded Postgres** product built by [PlanetScale](https://planetscale.com/) — the company behind [Vitess](https://vitess.io/), the widely-adopted open-source database clustering system for MySQL. Neki brings PlanetScale's deep expertise in horizontal scaling to the Postgres ecosystem.

Neki provides horizontal sharding for Postgres databases, managed by PlanetScale's proven infrastructure with high availability. Visit [neki.dev](https://www.neki.dev/) for the latest updates.

## Prerequisites

- An existing PostgreSQL database
- Understanding of current schema, query patterns, and scaling constraints
- Familiarity with sharding concepts (partition keys, cross-shard queries, data locality)

## Instructions

### Step 1: Assess Sharding Readiness

1. Read `{baseDir}/references/sharding-readiness.md` for the complete checklist.
2. Audit schema design practices against sharding requirements.
3. Review query patterns for shard-compatibility.

### Step 2: Identify Sharding Candidates

1. Flag tables exceeding single-node capacity or I/O limits.
2. Identify natural partition keys (tenant ID, user ID, region).
3. Verify even data distribution across candidate keys. Alternatively, consider composite partition keys for skewed data.

### Step 3: Evaluate Query Compatibility

1. Identify queries that include the partition key (single-shard routing).
2. Flag cross-shard queries (JOINs across boundaries, aggregations without partition filters).
3. Plan rewrites for incompatible patterns. Alternatively, denormalize data to co-locate related records.

### Step 4: Plan the Migration Path

1. Add explicit primary keys to all tables.
2. Add the partition key column where needed.
3. Eliminate or refactor cross-shard foreign key dependencies.
4. Test application behavior with connection pooling.

## Output

1. Current database state (size, table count, growth rate)
2. Sharding readiness score per table (ready / needs work / not applicable)
3. Identified partition keys with justification
4. Queries requiring rewrites
5. Migration risk assessment

## Error Handling

- **No natural partition key:** Consider synthetic keys or designate table as unsharded reference table.
- **Heavy cross-shard JOINs:** Refactor to application-level joins or denormalize. Alternatively, use materialized views.
- **Uneven distribution:** Choose high-cardinality keys. Avoid hot-shard keys (status fields, booleans).
- **Foreign key dependencies:** Replace database-level FKs with application-level referential integrity.

## Examples

**Example: SaaS sharding evaluation**

```
Scenario: Multi-tenant app, 500M rows in events table
1. Read {baseDir}/references/sharding-readiness.md
2. Evaluate: tenant_id has high cardinality, even distribution
3. Most queries filter by tenant_id (good single-shard routing)
4. Foreign keys: events -> users (same tenant, co-locatable)
5. Assessment: Good candidate. Pre-migration: add tenant_id to composite PKs
```

## Resources

Reference files in `{baseDir}/references/`: sharding-readiness (1 file).
