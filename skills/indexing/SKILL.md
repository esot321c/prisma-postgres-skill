---
name: indexing
description: >
  PostgreSQL indexing patterns and Prisma's limitations. Use when adding indexes,
  diagnosing slow queries, dealing with N+1 problems, or needing partial, expression,
  GiST, or BRIN indexes. Triggers on "index", "@@index", "performance", "slow query",
  "EXPLAIN", "N+1", "partial index", "GIN", "GiST", "BRIN".
---

# Indexing & Query Performance

## Rules

- Always add `@@index` on foreign key fields. Prisma does not create these automatically.
- Run `EXPLAIN ANALYZE` on any query that touches more than ~1000 rows or joins 3+ tables before assuming Prisma's generated SQL is fine.
- Enable query logging in development (`prisma.$on('query')`) to see what Prisma actually generates.

## What Prisma Handles Well

- B-tree indexes via `@@index` and `@@unique`.
- Composite indexes via `@@index([fieldA, fieldB])`.
- GIN indexes on `Json` fields via `@@index([field], type: Gin)` (Prisma 5.x+).

## What Prisma Cannot Express (Use Custom Migrations)

### Partial Indexes

For tables with soft deletes or status-filtered queries where most reads exclude a subset of rows.

```sql
-- In a custom migration (prisma migrate --create-only, then edit the SQL)
CREATE INDEX idx_clients_email_active ON clients(email) WHERE deleted_at IS NULL;
```

Without this, every query filtering `WHERE deleted_at IS NULL` scans deleted rows too.

### Expression Indexes

For case-insensitive lookups or computed values.

```sql
CREATE INDEX idx_clients_email_lower ON clients(LOWER(email));
```

Prisma has no way to express `LOWER()` in an `@@index`. Without this, case-insensitive email lookups do a sequential scan or rely on `ILIKE` which cannot use a standard B-tree index.

### GiST Indexes

For range types, geometric data, or PostGIS. Not supported in Prisma schema at all.

```sql
CREATE INDEX idx_matters_date_range ON matters USING gist(tsrange(start_date, end_date));
```

### BRIN Indexes

For large append-only tables (audit logs, events) where data is naturally ordered by insertion time.

```sql
CREATE INDEX idx_audit_log_created ON audit_log USING brin(created_at);
```

Far smaller than a B-tree on a million-row log table. Prisma has no `type: Brin` option.

## N+1 and Join Behavior

Prisma's `include` generates separate queries per relation by default.

```typescript
// This generates 1 query for clients + 1 query for matters + 1 query for documents
const clients = await prisma.client.findMany({
  include: { matters: { include: { documents: true } } }
});
```

For hot paths, enable `relationJoins` in the Prisma preview features to use lateral joins instead. Verify it is active in your project before assuming nested includes are single-query.

If `relationJoins` is unavailable or insufficient, use `$queryRaw` with an explicit JOIN:

```typescript
const results = await prisma.$queryRaw<ClientWithMatters[]>`
  SELECT c.*, json_agg(m.*) as matters
  FROM clients c
  LEFT JOIN matters m ON m.client_id = c.id
  WHERE c.deleted_at IS NULL
  GROUP BY c.id
  ORDER BY c.created_at DESC
  LIMIT 50
`;
```

## Correct

```prisma
model Matter {
  id        String    @id @default(dbgenerated("uuid_generate_v7()")) @db.Uuid
  clientId  String    @map("client_id") @db.Uuid
  client    Client    @relation(fields: [clientId], references: [id])
  status    String    @default("open")
  createdAt DateTime  @default(now()) @map("created_at")
  updatedAt DateTime  @updatedAt @map("updated_at")
  deletedAt DateTime? @map("deleted_at")

  @@index([clientId])           // explicit FK index
  @@index([status, createdAt])  // composite for filtered + sorted queries
  @@map("matters")
}
```

Plus a custom migration for the partial index:

```sql
CREATE INDEX idx_matters_active_by_client ON matters(client_id, created_at)
  WHERE deleted_at IS NULL;
```

## Incorrect

```prisma
model Matter {
  id        String   @id @default(uuid())
  clientId  String   @map("client_id") @db.Uuid
  client    Client   @relation(fields: [clientId], references: [id])
  status    String   @default("open")
  createdAt DateTime @default(now()) @map("created_at")

  // no @@index on clientId, no composite index
  // no custom migration for partial index on soft delete
  @@map("matters")
}
```

Every `findMany({ where: { clientId } })` does a sequential scan. Filtered queries on active records scan the full table including soft-deleted rows.
