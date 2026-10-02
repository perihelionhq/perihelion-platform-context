---
name: review-data-migration
description: Review a code change for schema dump drift, swapped ID or enum mappings, irreversible migrations, missing backfills, deploy-window breaks, orphaned references, broken dual-writes, missing transactions, unsafe hot-table indexes, silent data loss, and missing post-deploy verification. Use when reviewing database migrations, schema changes, or data backfills.
---

# Review lens: Data Migration

Review a migration or schema change for what happens in the deploy window: old code on the new schema, new code on old data, and partial failures that leave inconsistent state.

## Scope

- **Schema drift** Columns, tables, indexes or version bumps in `db/schema.rb` or `db/structure.sql` that no migration in this change explains, usually carried in from another branch. The dump's version stamp should match the change's newest migration.
- **Swapped mappings** `1 => TypeA, 2 => TypeB` in code when production has the reverse. Check each `CASE` or `IF` branch and constant hash entry individually.
- **Irreversible migrations** Column drops, precision-losing type changes and data deletes without a rollback plan, or a `down` that is missing or does not restore.
- **Missing backfill** A new `NOT NULL` column without a default or backfill fails on existing rows.
- **Deploy-window breaks** A rename or drop before every code path stops reading the old shape, or a constraint existing rows violate. The fix is expand and contract: add, migrate readers and writers, remove only once nothing reads the old shape.
- **Orphaned references** After a drop or rename, stale columns or associations in serializers, jobs, admin, rake tasks, and `includes` or `joins`.
- **Broken dual-write** During a transition both old and new columns must be populated, or a rollback sees NULLs.
- **Transactions and indexes** Multi-table backfills without suitable transaction scope; indexes on large tables created without concurrent or online creation where available.
- **Silent data loss** `text` to `varchar(n)` truncation, float to integer precision loss.
- **Verification** For non-trivial transforms, read-only SQL that proves correctness after deploy (mapping counts grouped by old and new column, NULL counts on recent rows, dual-write checks) and rollback or feature-flag guardrails, or a ticket that defers them.

## Method

When a schema dump is in the change, check drift first. Diff the dump and `db/migrate/` against the review's base ref, never an assumed `main`, and cross-reference every dump change against the migrations in this change.

Then check migration correctness, then verification and rollback. Do not trust fixtures; production data shapes differ.

## Threshold

Report destructive DDL, `NOT NULL` without backfill, drift with no matching migration, swapped mappings, and orphaned references you can name. Report data impact inferred from application code without visible migration handling only when it is severe.

Do not report nullable column additions, new tables with defaults, indexes on new or small tables, test fixtures, seeds or test database setup, or purely additive schema that does not touch existing rows. Do not check drift when no schema dump is in the change.

## Reporting

- For drift, list the unrelated objects on the dump path, and give the fix: restore the dump from the base ref (`git checkout <base> -- db/schema.rb`) and rerun `bin/rails db:migrate`.
- For a migration risk, name the rows or code paths affected and what fails or is lost, and when: during deploy, after deploy, or on rollback.
- For a risky transform without verification, give sample read-only SQL that would prove it.
