# ServicesBG Latest Report

Updated: 2026-06-29T12:05:51+03:00

## Status
Phase 3C-2 Full Staging Migration Apply and Validation is complete.

The first full migration was applied to staging only. No production migration, production redirects, external APIs, SMS, email, payments, calendar calls, or LLM calls were executed.

## Safety
Fresh backups were created before apply:
- DB: `/opt/projects/servicesbg/backups/staging-db-20260629_114444.sql.gz`
- WP files/uploads: `/opt/projects/servicesbg/backups/staging-wp-files-uploads-20260629_115823.tgz`

Pre-apply dry-run passed:
- blockers: none
- batch key: `phase3c_full_staging`

## Deliverables
- `app/wp-content/plugins/servicesbg-migration/src/Service/FullMigrationApplyService.php`
- `app/wp-content/plugins/servicesbg-migration/src/CLI/Commands.php`
- `docs/phase3c2_full_staging_migration_v1.md`
- `reports/full_staging_migration_validation.md`
- `scripts/validate_full_staging_migration.sh`
- `reports/latest.md`
- `reports/latest.json`

## Full Batch
- Batch key: `phase3c_full_staging`
- Batch id: `8`
- Status: `completed`
- Started: `2026-06-29 08:58:38` UTC
- Finished: `2026-06-29 09:02:03` UTC

## Imported Counts
- Categories: 277.
- Accounts/providers: 189.
- Listings: 2735.
- Media references: 6139.
- Video references preserved: 43.
- Claim seeds: 2633.
- Coverage seeds: 2735.
- Search indexed listings: 2735.
- Redirect candidates: 2944.

## Validation
`./scripts/validate_full_staging_migration.sh` passed against `/opt/projects/servicesbg/wp-staging`.

Validated:
- source vs destination counts
- category hierarchy
- provider/account mapping
- listing mapping
- listing-provider and listing-category relationships
- media reference rows
- video references preserved as metadata
- claim seed rows
- coverage seed rows
- search index rows
- redirect candidate rows
- theme rendering against migrated data
- no production changes

## Explicitly Not Done
- no production migration
- no production redirects
- no external APIs
- no real SMS/email/payment/calendar/LLM calls
- no destructive full rollback execution
