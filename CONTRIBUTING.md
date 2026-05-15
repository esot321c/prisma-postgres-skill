# Contributing

## This Plugin Is Opinionated By Design

Every pattern in this plugin represents a deliberate choice. UUID v7 over v4. Strings over database enums. Snake_case mapping. Partial indexes via custom migrations. These are not suggestions — they are the plugin's position.

If you disagree with a decision, fork the plugin and change it for your use case. Do not open PRs to make decisions configurable or to add "it depends" qualifications. The value of this plugin is that it makes decisions so the user doesn't have to.

## Pull Request Requirements

- One concern per PR. Do not bundle unrelated changes.
- Describe the problem you experienced, not just the change you made.
- Show before/after examples if modifying skill content.
- Skills shape agent behavior — changes require testing across multiple sessions.

## What Will Not Be Accepted

- PRs that make opinionated decisions optional or configurable
- Generic "improvements" that water down specific guidance
- Changes that add support for CUID, ULID, or other ID strategies alongside UUID v7
- Database enum advocacy for status/category fields
- Bulk reformatting or restructuring without functional change

## Adding a New Skill

New skills must cover a specific Prisma + PostgreSQL concern that is not already addressed. Each skill lives in its own directory under `skills/` with a `SKILL.md` entry point.

Required frontmatter:

```yaml
---
name: skill-name
description: >
  When to use this skill and what triggers it.
---
```

## Structure

```
skills/
  opinionated-prisma/     # Overview, prerequisites, skill routing
  schema-design/       # Naming, IDs, enums, soft delete
  indexing/            # Index types, Prisma gaps, N+1
  migration-safety/    # Safe migration workflows
  transactions/        # Transaction types, bulk operations
  raw-sql-boundary/    # When to use $queryRaw
```

Each skill is independently invocable. The `opinionated-prisma` skill serves as the entry point that routes to the appropriate sub-skill.
