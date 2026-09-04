---
name: opinionated-prisma
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

This plugin assumes UUID v7 as the default primary key strategy.

**PostgreSQL 18+** supports UUID v7 natively via the built-in `uuidv7()` function.
No extension, no init script. Prefer Postgres 18 for greenfield projects:

```prisma
id String @id @default(dbgenerated("uuidv7()")) @db.Uuid
```

**PostgreSQL 17 and below** require the `pg_uuidv7` extension, which provides the
same capability under the name `uuid_generate_v7()`. Add to your Postgres
initialization:

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

If you are on managed Postgres below 18 that does not allow `pg_uuidv7`, fall back
to `gen_random_uuid()` (v4) and accept the index locality tradeoff. Do not use CUID
or ULID as a workaround.

Examples throughout these skills use the native `uuidv7()`; substitute
`uuid_generate_v7()` on Postgres 17 and below.

## Initial Setup Behavior

When first working with a project's database:

1. Check for an existing `prisma.schema` (or `schema.prisma`) and `docker-compose.yml`
2. Determine the Postgres major version (`SELECT version();`, or the image tag in `docker-compose.yml`)
3. On Postgres 18+, use native `uuidv7()`. An existing project already on `pg_uuidv7` keeps working — don't churn it — but new schemas use the built-in
4. On Postgres 17 or below, self-hosted (Docker, bare metal): add the `pg_uuidv7` extension if missing, note the change, and use `uuid_generate_v7()`
5. On managed Postgres 17 or below with restricted extensions, fall back to v4 and document the tradeoff in a schema comment
6. Once the ID strategy is established for a project, use it consistently. Do not mix strategies.
