# Listing Media Comparison

Listing: source `fl_listings.ID=2780`, staging `service_listing` post `3004`.

## Summary
| Check | Legacy/source | Staging after correction | Status |
| --- | ---: | ---: | --- |
| `fl_listings.Photos_count` | 3 | 3 gallery attachments | OK |
| `fl_listing_photos` rows | 3 | 3 media map rows | OK |
| Local source files | 3 found | 3 imported | OK |
| Featured image | `Main_photo=02-2022/ad2780/studio-za-profiesionalni-1344818540.jpg` | attachment `3046`, source photo `6006` | OK |
| Gallery order | `6006`, `6005`, `6007` | `6006`, `6005`, `6007` | OK |
| Missing image files | 0 | 0 | OK |
| Missing attachment mapping | 3 before correction | 0 after correction | OK |

## Row Comparison
| Legacy/source photo ID | Source DB value | Source file | Migration destination | View model | Theme output | Status |
| ---: | --- | --- | --- | --- | --- | --- |
| 6006 | `Position=1`, `Original=02-2022/ad2780/studio-za-profiesionalni-1344818540_orig.jpg` | exists, checksum `73d6a94238b380c4c33c78a4343869acf87dd9167878c45a3bbb8504ce1c431e` | media map row `5808`, attachment `3046`, gallery position `1`, featured image | gallery item `source_photo_id=6006`, `position=1` | rendered in `Снимки`; also featured image | OK |
| 6005 | `Position=2`, `Original=02-2022/ad2780/studio-za-profiesionalni-449175707_orig.jpg` | exists, checksum `04f1a5fdc2eaacc8fa503e15a5b5d5d1d58cbbcb4cc073a6b62cacfc4b469f1a` | media map row `5809`, attachment `3047`, gallery position `2` | gallery item `source_photo_id=6005`, `position=2` | rendered in `Снимки` | OK |
| 6007 | `Position=3`, `Original=02-2022/ad2780/studio-za-profiesionalni-2120299126_orig.png` | exists, checksum `d6772922e16081b60c6955c56807cfe2bef076d51949aee7ac4ee33e8ef4539e` | media map row `5810`, attachment `3048`, gallery position `3` | gallery item `source_photo_id=6007`, `position=3` | rendered in `Снимки` | OK |

## Root Cause
Images were missing because the full staging migration recorded `wp_servicesbg_media_map` rows but deferred WordPress attachment import. The map rows had `status=mapped_local_file` and `target_attachment_id=NULL`. The theme also had no gallery rendering path from the listing detail body model.

## Correction
The audited staging listing now has imported WP attachments, media map attachment IDs, a featured image, source photo metadata, gallery position metadata, and a rendered gallery.
