# Global Media Audit

## Before Fix
| Check | Count |
| --- | ---: |
| Source `fl_listing_photos` rows | 6139 |
| Source image rows | 6096 |
| Source video rows | 43 |
| Source listings with image rows | 2292 |
| Source listings with any media | 2293 |
| Local archive files under `/opt/projects/servicesbg/archive/extracted/servicebg/files` | 27755 |
| Staging media-map rows | 6139 |
| Staging local image map rows | 6096 |
| Staging imported attachment map rows | 3 |
| Staging video reference rows | 43 |
| Staging listings with imported image attachments | 1 |

## Root Cause
The migration created media-map rows but did not globally import WordPress attachments. The previous Phase 3C-3B fix created attachments only for source listing `2780`, so most listing media remained invisible to the view model and theme.

## Reference Failing Listing
| Item | Value |
| --- | --- |
| Staging post ID | `2416` |
| Source listing ID | `2139` |
| Source photos | `5498`, `5499` |
| Before fix | view model `gallery=[]`, no featured image |
| After fix | attachments `8319`, `8320`, featured image `8319`, gallery count `2` |

## After Fix
| Check | Count |
| --- | ---: |
| Staging media-map rows | 6139 |
| Imported attachment map rows | 5794 |
| Video reference rows | 43 |
| Listings with imported image rows | 2231 |
| Published listings with visible imported images | 2181 |
| Published listings with source images but zero attachments | 0 |
| Listing child attachments | 5794 |
| Listings with featured image | 2231 |

## Non-Imported Rows
| Category | Count | Treatment |
| --- | ---: | --- |
| Video references | 43 | Preserved as `video_reference`; not treated as missing images. |
| Source images without migrated staging listing | 302 rows | Not imported because no target `service_listing` post exists to own/display them. |
| Missing local image files | 0 | No blocker. |
| Import failures | 0 | No blocker. |
