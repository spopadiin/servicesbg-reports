# servicesbg Source Schema Audit v1 Report

Detailed document: `docs/source_schema_audit_v1.md`

## Status
Completed from the preserved Flynax SQL dump and extracted media tree. No application code or production changes were made.

## Key Results
- Accounts: 189
- Account types used: `provider_company` 76, `account_freelancer` 99, `visitor_registered` 13, `dealer` 1
- Temporary/quick-account candidates: 86
- Accounts with zero listings: 105
- Categories: 277
- Duplicate category key groups: 5
- Listings: 2735 total / 2640 active / 59 expired / 35 trash / 1 incomplete
- Listings without phone: 7 active listings
- Duplicate listing phone groups: 253 groups covering 1770 listings
- Listing media rows: 6139
- Missing media references: 92
- Filesystem listing media directories: 2296
- Messages: 894
- Contacts: 433
- Booking requests/booked intervals: 0
- Location/multifield data: 125016 `fl_data_formats` rows and 124475 `fl_geo_mapping` rows
- Languages: Bulgarian active, English approval
- Language keys: 133215
- Plugin registry rows: 28

## Migration Impact
- Preserve every Flynax source ID in target object meta and migration map rows.
- Import active listings publicly only after validation; keep expired/trash/incomplete listings non-public or audit-only until approved.
- Treat missing-phone listings and duplicate-phone groups as claim workflow risks.
- Separate image media from YouTube/video references.
- Rebuild reservations because booking request/history tables are empty.
- Use location and geocode data as coverage suggestions, not verified provider coverage.

