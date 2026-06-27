# ServicesBG Latest Report

Updated: 2026-06-27T09:55:00+03:00

## Status
Phase 3A priming migration cycle MVP is implemented and validated.

This phase proves a small, controlled, reversible migration cycle from the Flynax source database into the staging WordPress environment. It imports only capped sample data, validates migration maps and search indexing, proves idempotent re-apply, and rolls the sample back safely.

No full migration, production migration, real SMS, real email, real payments, external APIs, or UI/theme work were implemented.

## Deliverables
- `app/wp-content/plugins/servicesbg-migration/src/Service/PrimingMigrationService.php`
- `app/wp-content/plugins/servicesbg-migration/src/CLI/Commands.php`
- `app/wp-content/plugins/servicesbg-migration/src/Schema/VersionManager.php`
- `docs/phase3a_priming_migration_cycle_v1.md`
- `scripts/validate_priming_migration.sh`
- `reports/latest.md`
- `reports/latest.json`

## WP-CLI Commands
- `wp servicesbg migration preflight`
- `wp servicesbg migration dry-run --sample`
- `wp servicesbg migration apply-sample`
- `wp servicesbg migration rollback-sample`
- `wp servicesbg migration validate-sample`

## Validated
- `./scripts/validate_priming_migration.sh` passed against `/opt/projects/servicesbg/wp-staging`.
- Source DB `services_services` is readable.
- Staging WordPress DB is writable.
- Production guard passes.
- Dry-run sample counts are capped.
- Sample apply imports 10 categories, 20 accounts, and 50 listings.
- Claim seed records are created where source phones exist.
- Coverage seed records are created where source location data exists.
- Representative media sample rows are tracked.
- Sample SEO redirect candidates are tracked.
- Migration map rows are created for imported objects.
- Search index refresh works for imported listings.
- Re-applying the sample is idempotent.
- Rollback removes active sample objects and leaves no completed sample batch.

## Sample Counts
- categories: 10
- accounts: 20
- listings: 50
- claim seeds: 50
- coverage seeds: 50
- media sample rows: 20
- redirect candidates: 20
- search indexed listings: 50

## Architecture Rules Preserved
- Sample import is batch-tracked.
- Flynax source IDs are preserved on objects and in migration maps.
- Duplicate prevention uses migration map rows.
- Rollback deletes only objects mapped to the sample batch.
- Imported coverage is marked `needs_review`, not verified service coverage.
- Claim seeds do not create OTPs or transfer ownership.
- Media is mapped as a representative sample only; attachments are not imported.
- Redirects are candidates only; no production redirects are activated.

## Explicitly Not Done
- no full migration
- no production migration
- no real SMS
- no real email
- no real payments
- no external APIs
- no UI/theme work
- no production changes

## Next Step
Review the Phase 3A validation report and decide which importer area should be expanded next: media attachments, full category hierarchy, account/profile normalization, or listing field completeness.
