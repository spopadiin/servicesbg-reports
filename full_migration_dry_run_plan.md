# ServicesBG Full Migration Dry-Run Plan

Updated: 2026-06-29T10:10:11+03:00

## Status
Full migration dry-run planning is implemented and validated as read-only.

No full import was executed. No `phase3c_full_staging` batch row was created. No production changes were made.

## Command

```bash
wp servicesbg migration dry-run-full --format=json --path=/opt/projects/servicesbg/wp-staging
```

## Batch Key

```text
phase3c_full_staging
```

## Dependency Order
1. Preflight.
2. Staging DB backup.
3. Staging uploads backup.
4. Categories.
5. Accounts.
6. Listings.
7. Media policy.
8. Media map and attachments.
9. Claim seeds.
10. Coverage suggestions.
11. Search index.
12. Redirect candidates.
13. Validation report.

## Expected Counts

| Item | Count |
| --- | ---: |
| Categories | 277 |
| Accounts/providers | 189 |
| Listings | 2735 |
| Active listings | 2640 |
| Media references | 6139 |
| SEO URL candidates | 2944 |
| Claim seed candidates | 2633 |
| Coverage seed candidates | 2735 |

## Estimated Batches

| Batch type | Batch size | Estimated batches |
| --- | ---: | ---: |
| Categories | 100 | 3 |
| Accounts/providers | 100 | 2 |
| Listings | 250 | 11 |
| Media references | 500 | 13 |
| Redirect candidates | 500 | 6 |

## Media Policy Outcomes

| Outcome | Count |
| --- | ---: |
| Skip/log secondary image missing | 0 |
| Promote first existing image | 0 |
| Assign generic placeholder | 0 |
| Preserve video reference without attachment | 40 |
| Critical missing media directory | 0 |

Placeholder strategy:
- `servicesbg_generic_placeholder_reference`
- Final asset not required for this dry run.
- Later category-specific placeholders supported.

## Rollback Guard
Rollback dry-run command:

```bash
wp servicesbg migration rollback-full --dry-run --format=json --path=/opt/projects/servicesbg/wp-staging
```

Guard behavior:
- Full rollback can target only `phase3c_full_staging`.
- `phase3a_priming_sample` is protected.
- Future destructive rollback requires explicit full batch id.
- Current command is dry-run only.
- Current full batch candidates: 0.
- Would delete counts are all 0 because no full migration batch exists.

## Blockers
None for the dry-run planner.

## Warnings
- `video_references_are_not_attachment_files`
- `no_full_migration_batch_exists` from rollback dry-run, expected before full import.

## Safety
- `writes_performed: false`
- `full_import_executed: false`
- `external_apis_called: false`
- `production_modified: false`
