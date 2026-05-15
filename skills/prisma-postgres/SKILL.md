---
name: prisma-postgres
description: >
  Overview and prerequisites for Prisma ORM + PostgreSQL patterns.
  Use when first setting up a database, establishing ID strategy, or needing
  guidance on which specific skill to use. Triggers on "prisma", "database",
  "postgresql", "postgres".
---

# Prisma + PostgreSQL

Opinionated patterns for building with Prisma on self-hosted PostgreSQL. This plugin
provides separate skills for each concern — use the one that matches your current task.

## Available Skills

| Skill | Use When |
|---|---|
| `schema-design` | Designing models, choosing field types, naming conventions, primary keys, enums, soft delete |
| `indexing` | Adding indexes, diagnosing slow queries, N+1 problems, partial/expression/GiST/BRIN indexes |
| `migration-safety` | Planning or running `prisma migrate`, adding columns to large tables, constraint changes |
| `transactions` | Writing transactions, bulk creates, bulk upserts, choosing interactive vs sequential |
| `raw-sql-boundary` | Deciding whether to use `$queryRaw`, window functions, CTEs, full-text search, JSONB operators |

Invoke the specific skill via the Skill tool when the task narrows to one of these areas.

## Prerequisites

### UUID v7 Support

This plugin assumes UUID v7 as the default primary key strategy. This requires the
`pg_uuidv7` extension.

Add to your Postgres initialization:

```sql
-- init.sql
CREATE EXTENSION IF NOT EXISTS "pg_uuidv7";
```

For Docker Compose, mount an init script:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
  - ./init.sql:/docker-entrypoint-initdb.d/init.sql
```

Then in Prisma schemas, use:

```prisma
id String @id @default(dbgenerated("uuid_generate_v7()")) @db.Uuid
```

If you are on managed Postgres that does not support `pg_uuidv7`, fall back to
`gen_random_uuid()` (v4) and accept the index locality tradeoff. Do not use CUID
or ULID as a workaround.

## Initial Setup Behavior

When first working with a project's database:

1. Check for an existing `prisma.schema` (or `schema.prisma`) and `docker-compose.yml`
2. If `pg_uuidv7` is already configured, follow UUID v7 patterns throughout
3. If Postgres is self-hosted (Docker, bare metal) but `pg_uuidv7` is missing, add it and note the change
4. If Postgres is managed and extensions are restricted, fall back to v4 and document the tradeoff in a schema comment
5. Once the ID strategy is established for a project, use it consistently. Do not mix strategies.
