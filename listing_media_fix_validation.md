# Listing Media Fix Validation

Validation script: `scripts/validate_global_media_fix.sh`

Status: passed.

## QA Listings
| Listing | Post ID | Source ID | Card image | Gallery count | Featured image |
| --- | ---: | ---: | --- | ---: | ---: |
| QA listing #1 | 3004 | 2780 | available | 3 | 3046 |
| QA listing #2 | 2416 | 2139 | available | 2 | 8319 |

## Random Published Listing Sample
Each sampled listing returned `listing_card.image.state=available`, gallery count `>=1`, and a featured image ID.

| Source ID | Post ID | Card image | Gallery count | Featured image |
| ---: | ---: | --- | ---: | ---: |
| 1979 | 2256 | available | 14 | 7883 |
| 1626 | 1904 | available | 15 | 6283 |
| 959 | 1242 | available | 1 | 4991 |
| 2194 | 2465 | available | 3 | 8411 |
| 1303 | 1582 | available | 1 | 5371 |
| 845 | 1129 | available | 1 | 4877 |
| 1478 | 1756 | available | 1 | 5621 |
| 2055 | 2332 | available | 6 | 8082 |
| 389 | 674 | available | 1 | 3747 |
| 1661 | 1939 | available | 1 | 6560 |
| 1814 | 2091 | available | 6 | 7181 |
| 905 | 1188 | available | 1 | 4937 |
| 1969 | 2246 | available | 3 | 7828 |
| 1451 | 1729 | available | 1 | 5594 |
| 1700 | 1978 | available | 3 | 6634 |
| 1283 | 1562 | available | 1 | 5346 |
| 1946 | 2223 | available | 5 | 7790 |
| 289 | 577 | available | 7 | 3552 |
| 546 | 830 | available | 1 | 4533 |
| 2145 | 2422 | available | 5 | 8335 |

## Media-Rich Listings
| Source ID | Post ID | Imported rows | View-model gallery count |
| ---: | ---: | ---: | ---: |
| 1648 | 1926 | 20 | 20 |
| 1649 | 1927 | 20 | 20 |
| 1639 | 1917 | 20 | 20 |
| 162 | 451 | 18 | 18 |
| 1640 | 1918 | 17 | 17 |
| 1840 | 2117 | 16 | 16 |
| 1624 | 1902 | 16 | 16 |
| 26 | 315 | 15 | 15 |
| 1612 | 1890 | 15 | 15 |
| 1623 | 1901 | 15 | 15 |

## HTTP Checks
Representative staging image URLs returned HTTP 200:
- `https://services.herbal.bg/wp-content/uploads/legacy-flynax/11-2018/ad2139/15431257631471108988_orig.jpg`
- `https://services.herbal.bg/wp-content/uploads/legacy-flynax/02-2022/ad2780/studio-za-profiesionalni-1344818540_orig-1024x449.jpg`

## Safety
- Production changes: no.
- External APIs: no.
- UI redesign: no.
- CSS redesign: no.
- Videos preserved separately: yes.
