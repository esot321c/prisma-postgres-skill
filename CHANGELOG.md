# Changelog

All notable changes to this project will be documented in this file.

## [1.0.0] - 2026-05-15

### Added

- Multi-platform plugin support (Claude Code, Cursor, Codex, Gemini CLI, OpenCode)
- **schema-design** skill: naming conventions, UUID v7 primary keys, enum decisions, soft delete patterns
- **indexing** skill: Prisma indexing gaps, partial/expression/GiST/BRIN indexes, N+1 join behavior
- **migration-safety** skill: safe `prisma migrate` workflows, concurrent index creation, constraint validation
- **transactions** skill: interactive vs sequential transactions, bulk create/upsert/update patterns
- **raw-sql-boundary** skill: decision logic for `$queryRaw` vs Prisma query API
- **opinionated-prisma** skill: overview, prerequisites, UUID v7 setup, skill routing
- Cross-platform session hooks (Windows + Unix)
- Marketplace metadata for plugin installation
- Per-platform installation commands (CLI syntax for Claude Code, Codex, Cursor, Gemini CLI, OpenCode)
- CLAUDE.md with repo instructions for Claude
- CONTRIBUTING.md with contributor guidelines and fork-first philosophy
- CODE_OF_CONDUCT.md (Contributor Covenant v2.0)
- CHANGELOG.md

### Changed

- Restructured from single monolithic skill to six independent skills, each in its own directory
- README rewritten with "How It Works" section, richer skill descriptions, per-platform install commands, and updating instructions
