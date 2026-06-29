# ServicesBG Full Staging Migration Validation

Updated: 2026-06-29T12:05:51+03:00

## Status
Full staging migration validation passed.

Batch:
- Key: `phase3c_full_staging`
- ID: `8`
- Status: `completed`

Backups created before apply:
- `/opt/projects/servicesbg/backups/staging-db-20260629_114444.sql.gz`
- `/opt/projects/servicesbg/backups/staging-wp-files-uploads-20260629_115823.tgz`

## Source Counts

| Source item | Count |
| --- | ---: |
| Categories | 277 |
| Accounts | 189 |
| Listings | 2735 |
| Active listings | 2640 |
| Listing media rows | 6139 |
| Video references | 43 |
| Claim seed candidates | 2633 |
| Coverage seed candidates | 2735 |
| Category redirect candidates | 277 |
| Page redirect candidates | 30 |
| Listing redirect candidates | 2637 |
| SEO URL candidates | 2944 |

## Destination Counts

| Destination item | Count |
| --- | ---: |
| Category map rows | 277 |
| Account map rows | 189 |
| Listing map rows | 2735 |
| Claim map rows | 2633 |
| Coverage map rows | 2735 |
| Media map rows | 6139 |
| Video reference rows | 43 |
| Redirect map rows | 2944 |
| Claim rows | 2633 |
| Coverage rows | 2735 |
| Search listing rows | 2735 |

## Relationship Checks

| Check | Result |
| --- | ---: |
| Category hierarchy orphans | 0 |
| Listings without mapped provider | 0 |
| Listings without mapped category | 0 |

## Validation Checks

| Check | Status |
| --- | --- |
| Categories match | passed |
| Accounts match | passed |
| Listings match | passed |
| Media references match | passed |
| Video references preserved | passed |
| Claims match | passed |
| Coverage match | passed |
| Search index match | passed |
| Redirects match | passed |
| Category hierarchy valid | passed |
| Provider relationships valid | passed |
| Listing-category relationships valid | passed |
| Theme rendering valid | passed |
| Production guard | passed |

## Theme Rendering
- Theme: `servicesbg-theme`
- Theme active: yes
- Sample listing id: `2461`
- Sample permalink: `https://services.herbal.bg/?service_listing=%d0%b7%d0%b0%d0%bd%d0%b8%d0%bc%d0%b0%d0%bb%d0%bd%d1%8f-%d0%be%d0%b1%d1%80%d0%b0%d0%b7%d0%be%d0%b2%d0%b0%d1%82%d0%b5%d0%bb%d0%b5%d0%bd-%d1%86%d0%b5%d0%bd%d1%82%d1%8a%d1%80`

## Safety
- Full migration scope: staging only.
- Production modified: no.
- External APIs called: no.
- Production redirects created: no.
- `rollback-full --dry-run` remains guarded.
