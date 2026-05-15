# prisma-postgres

An opinionated Claude Code skill for building with Prisma ORM on self-hosted PostgreSQL.

## What This Covers

- **Schema Design** — Naming conventions, primary key strategy (UUID v7), enum decisions, soft delete patterns
- **Indexing & Query Performance** — Prisma's indexing limitations, partial/expression/GiST/BRIN indexes via custom migrations, N+1 join behavior
- **Migration Safety** — `prisma migrate` workflows, safe patterns for adding required columns, concurrent index creation, constraint validation
- **Transactions & Bulk Operations** — Interactive vs sequential transactions, bulk create with relations, bulk upsert via raw SQL
- **Raw SQL Boundary** — Decision logic for when to use `$queryRaw` vs Prisma's query API

## Prerequisites

Assumes self-hosted PostgreSQL (Docker) with the `pg_uuidv7` extension. See [SKILL.md](prisma-postgres/SKILL.md) for setup instructions.

## Installation

### Claude Code

Copy the `prisma-postgres/` folder into your skills directory:

```bash
cp -r prisma-postgres/ ~/.claude/skills/prisma-postgres/
```

### Project-Level

Or add it to a specific project:

```bash
cp -r prisma-postgres/ ./your-project/.claude/skills/prisma-postgres/
```

## Structure

```
prisma-postgres/
  SKILL.md                # Entry point, prerequisites, skill behavior
  schema-design.md        # Naming, IDs, enums, soft delete
  indexing.md             # Index types, Prisma gaps, N+1 patterns
  migration-safety.md     # Safe migration workflows
  transactions.md         # Transaction types, bulk operations
  raw-sql-boundary.md     # When to escape to $queryRaw
```

## Opinions

This skill makes decisions rather than presenting options:

- UUID v7 via `pg_uuidv7`, not v4/CUID/ULID
- String fields for status/category, database enums only for type discriminators
- Slugs for user-facing URLs, UUIDs for internal keys
- Normalize by default, denormalize only with measured justification
- `$queryRaw` only when Prisma changes algorithmic complexity, not for marginal speed gains

## License

MIT