---
name: neki
description: |
  Overview and guidance for Neki, PlanetScale's sharded Postgres product.
  Use when evaluating Postgres scaling or sharding readiness, planning
  horizontal sharding, or preparing a schema for future sharding with Neki.
  Trigger with phrases like "Neki", "shard Postgres", "Postgres sharding",
  "horizontal scaling Postgres", "sharding readiness".
allowed-tools: Read
version: 1.0.0
author: PlanetScale <skills@planetscale.com>
license: MIT
---

# PlanetScale Neki

Evaluate Postgres sharding readiness and plan for horizontal scaling with Neki.

## Overview

Neki is a **sharded Postgres** product built by [PlanetScale](https://planetscale.com/) — the company behind [Vitess](https://vitess.io/), the widely-adopted open-source database clustering system for MySQL. Neki brings PlanetScale's deep expertise in horizontal scaling and database infrastructure to the Postgres ecosystem.

**What Neki does:**

- **Sharded Postgres** — Horizontal sharding for Postgres databases, enabling applications to scale beyond the limits of a single node.
- **Managed by PlanetScale** — Built on PlanetScale's proven infrastructure and operational experience running large-scale databases.
- **High availability** — Leveraging PlanetScale's track record of delivering highly available database services.

> **Note:** Neki is not yet a released product, but will be available soon. Information here will be updated regularly. Visit [neki.dev](https://www.neki.dev/) for the latest updates.

## Prerequisites

- An existing PostgreSQL database (any version supported by PlanetScale)
- Understanding of your current schema, query patterns, and scaling constraints
- Familiarity with sharding concepts (partition keys, cross-shard queries, data locality)

## Instructions

### Step 1: Assess Sharding Readiness

Before adopting Neki, evaluate whether your Postgres schema and queries are ready for horizontal sharding.

Read `{baseDir}/references/sharding-readiness.md` for the complete sharding readiness checklist covering:
- Schema design practices that enable future sharding
- Query patterns that work well (and poorly) across shards
- Primary key and foreign key considerations
- Data locality and partition key selection

### Step 2: Identify Sharding Candidates

Review your database for tables that would benefit from sharding:
- Tables exceeding single-node capacity or I/O limits
- Tables with natural partition keys (tenant ID, user ID, region)
- Tables with predictable, even data distribution

### Step 3: Evaluate Query Compatibility

Audit your query patterns against sharding constraints:
- Identify queries that include the partition key (single-shard routing)
- Flag cross-shard queries (JOINs across partition boundaries, aggregations without partition key filters)
- Plan rewrites for incompatible patterns

### Step 4: Plan the Migration Path

Prepare your schema for a future Neki migration:
- Ensure all tables have explicit primary keys
- Add the chosen partition key column to tables that need it
- Eliminate or refactor cross-shard foreign key dependencies
- Test application behavior with connection pooling (preparation for shard-aware routing)

## Output

Structure sharding readiness assessments with:
1. Current database state (size, table count, growth rate)
2. Sharding readiness score per table (ready / needs work / not applicable)
3. Identified partition keys with justification
4. List of queries requiring rewrites
5. Migration risk assessment and timeline estimate

## Error Handling

- **No natural partition key:** Consider synthetic partition keys or evaluate whether the table needs sharding at all. Some tables work better as reference/lookup tables replicated to all shards.
- **Heavy cross-shard JOINs:** Refactor to application-level joins or denormalize data to co-locate related records on the same shard.
- **Uneven data distribution:** Choose a partition key with high cardinality and uniform distribution. Avoid keys that create hot shards (e.g., status fields, boolean flags).
- **Foreign key dependencies across shards:** Replace database-level foreign keys with application-level referential integrity checks.

## Examples

### Sharding Readiness Evaluation

```
User: We have a multi-tenant SaaS app on Postgres with 500M rows in the events table. Should we shard?

1. Read {baseDir}/references/sharding-readiness.md
2. Evaluate the events table:
   - Natural partition key: tenant_id (high cardinality, even distribution)
   - Check queries: most filter by tenant_id (good for single-shard routing)
   - Foreign keys: events -> users (same tenant, co-locatable)
3. Assessment: Good sharding candidate with tenant_id as partition key
4. Pre-migration steps: add tenant_id to composite PKs, audit cross-tenant queries
```

## Resources

All reference files are in `{baseDir}/references/`:

| File | Topic |
| --- | --- |
| `sharding-readiness.md` | Schema and query practices for sharding readiness |
