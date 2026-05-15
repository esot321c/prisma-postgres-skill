---
name: prisma-postgres
description: >
  Prisma ORM with PostgreSQL patterns, escape hatches, and migration workflows.
  Use when designing schemas, writing queries, planning migrations, or optimizing
  performance in a Prisma + PostgreSQL stack. Triggers on "prisma", "schema",
  "migrate", "index", "query", "$queryRaw", "transaction", "bulk", "database",
  "model", "relation".
---

# Prisma + PostgreSQL Skill

Opinionated patterns for building with Prisma on self-hosted PostgreSQL. Covers schema
design, indexing, migrations, transactions, and the boundary where Prisma's query API
falls short and raw SQL is the correct choice.

## Prerequisites

### UUID v7 Support

This skill assumes UUID v7 as the default primary key strategy. This requires the
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

## Skill Behavior

When first working with a project's database:

1. Check for an existing `prisma.schema` (or `schema.prisma`) and `docker-compose.yml`
2. If `pg_uuidv7` is already configured, follow UUID v7 patterns throughout
3. If Postgres is self-hosted (Docker, bare metal) but `pg_uuidv7` is missing, add it and note the change
4. If Postgres is managed and extensions are restricted, fall back to v4 and document the tradeoff in a schema comment
5. Once the ID strategy is established for a project, use it consistently. Do not mix strategies.

## Sections

When working on schema design or model definitions, reference `schema-design.md`.
When working on indexes or query performance, reference `indexing.md`.
When planning or running migrations, reference `migration-safety.md`.
When writing transactions or bulk operations, reference `transactions.md`.
When deciding whether to use $queryRaw, reference `raw-sql-boundary.md`.
