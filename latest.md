# ServicesBG Latest Report

Updated: 2026-06-29T10:49:04+03:00

## Status
Phase 3C-1 Media Missing File Policy, Full Dry-Run Planner, and Rollback Guard is complete and validated against staging.

No full import was run. No `phase3c_full_staging` batch row was created. No production changes were made.

## Deliverables
- `app/wp-content/plugins/servicesbg-migration/src/Service/FullMigrationPlanningService.php`
- `app/wp-content/plugins/servicesbg-migration/src/CLI/Commands.php`
- `docs/phase3c1_media_policy_dry_run_rollback_v1.md`
- `docs/current/migration_status.md`
- `docs/current/validation_matrix.md`
- `reports/migration_media_policy_report.md`
- `reports/full_migration_dry_run_plan.md`
- `scripts/validate_full_migration_dry_run.sh`
- `reports/latest.md`
- `reports/latest.json`

## WP-CLI
- `wp servicesbg migration inspect-media`
- `wp servicesbg migration media-policy-report`
- `wp servicesbg migration dry-run-full`
- `wp servicesbg migration rollback-full --dry-run`

## Media Policy
- Missing media references inspected: 40.
- Missing image files: 0.
- Video references preserved as metadata without attachment import: 40.
- Secondary image missing policy: skip and log.
- Primary image missing with other images policy: promote first existing image.
- Only image missing policy: assign `servicesbg_generic_placeholder_reference`.
- Missing media directory policy: critical blocker.

## Full Dry-Run Plan
- Batch key: `phase3c_full_staging`.
- Expected counts: 277 categories, 189 accounts/providers, 2735 listings, 6139 media references.
- SEO URL candidates: 2944.
- Claim seed candidates: 2633.
- Coverage seed candidates: 2735.
- Estimated batches: 3 category, 2 account/provider, 11 listing, 13 media, 6 redirect.
- Blockers: none.
- Warning: `video_references_are_not_attachment_files`.

## Rollback Guard
- `rollback-full` requires `--dry-run`.
- Full rollback can target only `phase3c_full_staging`.
- `phase3a_priming_sample` is protected.
- Future destructive rollback requires explicit full batch id.

## Validation
- `./scripts/validate_full_migration_dry_run.sh` passed against `/opt/projects/servicesbg/wp-staging`.
- The validator confirmed media inspection, media policy, full dry-run planning, rollback-full dry-run guard, no full batch creation, no full import, and no production changes.

## Explicitly Not Done
- no full staging import
- no full migration apply command
- no destructive full rollback command
- no WordPress attachment import
- no production redirects
- no external API calls
- no production changes
