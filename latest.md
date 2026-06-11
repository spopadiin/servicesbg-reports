# servicesbg Latest Report

## Status
Flynax source schema audit completed from the preserved SQL dump and extracted media tree. No application code or production changes were made.

## Completed
- Public services.bg site inspected.
- Dev/migration VPS environment prepared.
- Dedicated Linux user `servicesbg` created.
- PHP, MariaDB, Nginx, WP-CLI installed.
- Flynax DB imported as `services_services`.
- Source archive extracted.
- Flynax version identified: 4.5.2.
- Active template identified: `general_wide`.
- Media root identified: `files/`.
- Source inventory created.
- V2 feature requirements created.
- Architecture v1 created.
- AGENTS.md created.
- ROADMAP.md created.
- Handoff JSON created.
- GitHub reporting pipeline prepared.
- WordPress CRM data model v1 created.
- Migration implementation plan v1 created.
- WordPress plugin architecture v1 created.
- Architecture planning summary report created for publication.
- Flynax source schema audit v1 created.
- Flynax-to-WordPress mapping v1 created.

## Source Counts
- Accounts: 189
- Categories: 277
- Listings total: 2735
- Active listings: 2640

## New Deliverables
- `docs/data_model_v1.md`
- `docs/migration_plan_v1.md`
- `docs/wordpress_plugin_architecture_v1.md`
- `reports/architecture_planning_summary.md`
- `docs/source_schema_audit_v1.md`
- `docs/flynax_to_wp_mapping_v1.md`

## Source Schema Audit Results
- Accounts: 189 rows across `provider_company` 76, `account_freelancer` 99, `visitor_registered` 13, and `dealer` 1.
- Temporary/quick-account candidates: 86.
- Accounts with zero listings: 105.
- Listings: 2735 total; 2640 active, 59 expired, 35 trash, 1 incomplete.
- Listings without phone: 7 active listings.
- Duplicate listing phone groups: 253 groups covering 1770 listings.
- Categories: 277 rows; 5 duplicate category key groups.
- Listing media rows: 6139; missing media references: 92; filesystem listing media directories: 2296.
- Messages: 894 rows.
- Booking requests/booked intervals: 0; booking plugin tables exist but contain no reservation history.
- Location/multifield reference data: 125016 `fl_data_formats` rows and 124475 `fl_geo_mapping` rows.
- Language data: Bulgarian active, English approval; 133215 language key rows.
- Plugins: 28 plugin registry rows.

## Key Decisions Documented
- Use WordPress users for providers, freelancers, and customers.
- Use CPT `service_listing` for public service listings.
- Use hierarchical taxonomy `service_category` for imported Flynax categories.
- Use custom tables for claims, reservations, conversations, leads, coverage zones, AI knowledge, media mapping, redirects, and migration traceability.
- Preserve Flynax IDs and batch IDs on imported objects.
- Seed claim eligibility during migration, but perform SMS OTP verification only at runtime.
- Keep imports staged, idempotent, and reversible by migration batch.

## Next Step
Review and approve the source schema audit and mapping. After approval, implementation can start with migration tooling/schema setup in a staged, reversible WordPress plugin.
