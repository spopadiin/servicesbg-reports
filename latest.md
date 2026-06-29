# ServicesBG Latest Report

Updated: 2026-06-29T10:10:11+03:00

## Status
Phase 3C-0 Full Staging Migration Preflight is complete and validated against staging.

This phase prepares for a future full staging migration without importing full data. It adds and validates backup commands, source readability checks, source counts, media readiness, SEO/URL counts, staging target readiness, disk-space checks, migration map table checks, rollback safety checks, and batch estimates.

## Deliverables
- `app/wp-content/plugins/servicesbg-migration/src/Service/FullMigrationPreflightService.php`
- `docs/phase3c_full_staging_migration_preflight_v1.md`
- `docs/current/migration_status.md`
- `docs/current/validation_matrix.md`
- `scripts/preflight_full_migration.sh`
- `scripts/validate_full_migration_preflight.sh`
- `reports/latest.md`
- `reports/latest.json`

## Preflight Results
- Source DB readable: yes.
- Source dump readable: yes, `/opt/projects/servicesbg/db/imports/localhost.sql.gz`.
- Source archive/media paths readable: yes.
- Staging WP DB writable: yes, temporary write probe passed.
- Backup root writable: yes, `/opt/projects/servicesbg/backups`.
- Free space on project/staging/backup volume: about 8.8 GiB.
- Migration map tables present: import batches, migration map, media map, redirect map.
- Rollback safety: sample batch clear, full batch key clear, active Flynax map rows clear.
- Full apply remains blocked until full dry-run planner and full rollback command are implemented.

## Source Counts
- Categories: 277.
- Accounts/providers: 189 total; 76 company providers, 99 freelancers, 13 registered visitors.
- Listings: 2735 total; 2640 active.
- Media references: 6139 listing photo rows; 6099 local files found; 40 missing local files.
- SEO aliases/URLs: 277 category paths, 48 pages, 30 active page paths, 2637 active listing URL candidates.

## Batch Estimate
- Categories: 3 batches at 100.
- Accounts/providers: 2 batches at 100.
- Listings: 11 batches at 250.
- Active listings: 11 batches at 250.
- Media references: 13 batches at 500.

## Blockers
- `media_missing_files_require_policy`: 40 media rows reference missing local files.
- Full import is also guarded by design until full dry-run and full rollback commands exist.

## Validation
- `./scripts/preflight_full_migration.sh` passed against `/opt/projects/servicesbg/wp-staging`.
- No full import was run.
- No `phase3c_full_staging` batch row was created.
- No external APIs were called.
- No production changes were made.

## Explicitly Not Done
- no full staging import
- no full migration apply command
- no full migration rollback command
- no WordPress attachment import
- no production redirects
- no external API calls
- no production changes
