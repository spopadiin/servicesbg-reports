# ServicesBG Phase 3A Priming Migration Cycle MVP v1

## Purpose
Phase 3A proves the migration pipeline with a small, controlled, reversible Flynax-to-WordPress sample cycle before any UI work or full migration.

This phase reads from the preserved Flynax source database and writes only to the staging WordPress environment.

## Scope
Sample migration only:
- up to 10 categories,
- up to 20 provider/account records,
- up to 50 listings,
- representative media map rows when source media exists,
- sample SEO redirect candidates,
- claim seed records where listing phone exists,
- coverage seed records where source location data exists,
- search index rebuild for imported listings.

No full migration and no production migration are implemented.

## Source and Target
Source:
- database: `services_services`
- table prefix: `fl_`
- media root: `/opt/projects/servicesbg/archive/extracted/servicebg/files`
- source dump: `/opt/projects/servicesbg/db/imports/localhost.sql.gz`

Target:
- staging WordPress root: `/opt/projects/servicesbg/wp-staging`
- WordPress plugins under `/opt/projects/servicesbg/repo/app/wp-content/plugins`

Production guard:
- sample apply refuses to run when `home_url()` looks like production instead of staging/local.

## WP-CLI Commands
Implemented in `servicesbg-migration`:

- `wp servicesbg migration preflight`
- `wp servicesbg migration dry-run --sample`
- `wp servicesbg migration apply-sample`
- `wp servicesbg migration rollback-sample`
- `wp servicesbg migration validate-sample`

`dry-run` requires `--sample`. Full migration commands are intentionally not implemented.

## Batch Tracking
Sample batch key:
- `phase3a_priming_sample`

Dry-run batches use timestamped keys:
- `phase3a_priming_sample_dry_run_YYYYMMDDHHMMSS`

Batch records are stored in:
- `wp_servicesbg_import_batches`

Each batch stores:
- source DB,
- source dump path,
- source media root,
- mode,
- status,
- dry-run flag,
- summary JSON.

## Migration Map Usage
Every imported WordPress/custom-table object is mapped through:
- `wp_servicesbg_migration_map`

Mapped target types:
- `term`
- `user`
- `listing`
- `claim`
- `coverage_zone`

Map rows preserve:
- source system,
- source table,
- source ID,
- source parent ID where available,
- target type/table/ID,
- owner plugin,
- rollback action,
- status.

The map is the duplicate-prevention and rollback source of truth.

## Imported Sample Objects
Categories:
- Source: `fl_categories`
- Target: `service_category`
- Preserved meta includes source category ID, parent ID, key, path, type, and batch ID.

Accounts:
- Source: `fl_accounts`
- Target: `wp_users`
- Role mapping:
  - `provider_company` -> `service_provider`
  - `account_freelancer` -> `service_freelancer`
  - `visitor_registered` -> `service_customer`
- Preserved user meta includes source account ID, account type, phone, location, quick-account marker, and batch ID.
- Flynax passwords are not migrated.

Listings:
- Source: `fl_listings`
- Target: `service_listing`
- Preserved post meta includes source listing/account/category IDs, Flynax status, import batch, claim status, phone, price, service fields, partner fields, location JSON, and original URL candidate.

Claim seeds:
- Target: `wp_servicesbg_claims`
- Created only when the source listing has a normalized phone.
- No OTPs are created and no ownership transfer occurs.

Coverage seeds:
- Target: `wp_servicesbg_coverage_zones`
- Created only when source location fields or coordinates exist.
- Status is `needs_review`; imported source location is not treated as verified coverage.

Media sample:
- Target: `wp_servicesbg_media_map`
- Records representative source media paths/checksums/status only.
- No WordPress attachment import is performed in Phase 3A.

SEO redirect sample:
- Target: `wp_servicesbg_redirect_map`
- Records sample old-path to new-URL candidates.
- No redirect handler or production redirect activation is implemented.

Search index:
- Imported listings are indexed through `servicesbg-search`.
- Search index rows are removed during rollback.

## Duplicate Prevention
Duplicate prevention uses active migration map rows for categories, accounts, listings, claim seeds, and coverage seeds.

Media sample rows are deduplicated by source photo ID.

Redirect sample rows are deduplicated by batch and source listing ID.

Re-running `apply-sample` after a successful apply creates no duplicate WordPress objects or sample map rows.

## Rollback Safety
Rollback uses the active sample batch and its migration map rows.

Rollback removes:
- imported listings,
- imported users,
- imported terms,
- claim seed rows,
- coverage seed rows,
- media sample rows,
- redirect sample rows,
- listing search index rows.

Rollback marks migration map rows as `rolled_back`.

Rollback does not run a broad delete. It only deletes objects mapped to the Phase 3A sample batch and skips rows marked rollback-protected.

## Validation
Created script:
- `scripts/validate_priming_migration.sh`

Validation checks:
- required plugins are active,
- source DB is readable,
- staging WP DB is writable,
- production guard passes,
- dry-run sample counts are capped,
- sample objects are imported,
- migration map rows are created,
- claim seed rows are created,
- coverage seed rows are created,
- representative media rows are created,
- redirect sample rows are created,
- search index refresh works for imported listings,
- re-apply is idempotent,
- rollback removes active sample objects,
- no completed sample batch remains after final rollback.

Validation result:
- `./scripts/validate_priming_migration.sh` passed against `/opt/projects/servicesbg/wp-staging`.

## Explicitly Not Implemented
- full migration,
- production migration,
- production redirects,
- WordPress attachment media import,
- historical messages import,
- real SMS,
- real email,
- real payments,
- external APIs,
- UI/theme work.
