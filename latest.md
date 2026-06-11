# ServicesBG Latest Report

Updated: 2026-06-11T22:06:31+03:00

## Status
Database schema planning, plugin table ownership, and MVP scope are complete for the ServicesBG WordPress CRM marketplace migration.

No application code was written. No production services.bg system was modified.

## Current Deliverables
- `docs/servicesbg_database_schema_v1.md`
- `docs/servicesbg_plugin_table_ownership_v1.md`
- `docs/servicesbg_mvp_scope_v1.md`
- `reports/latest.md`
- `reports/latest.json`

## Prior Planning Outputs Retained
- `docs/architecture_v1.md`
- `docs/data_model_v1.md`
- `docs/migration_plan_v1.md`
- `docs/wordpress_plugin_architecture_v1.md`
- `docs/source_schema_audit_v1.md`
- `docs/flynax_to_wp_mapping_v1.md`
- `docs/feature_gap_analysis_v1.md`
- `docs/servicesbg_feature_backlog_v1.md`
- `docs/plugin_strategy_v1.md`
- `docs/modular_plugin_architecture_v1.md`

## What Was Added
- Complete ServicesBG custom database schema by MVP, Phase 2, and Phase 3.
- WordPress native table usage, CPT, taxonomy, post meta, and user meta strategy.
- Custom table definitions with key columns, indexes, logical relationships, source ID fields, and rollback behavior.
- Dedicated `servicesbg-search` module and search-owned tables.
- Plugin/module ownership matrix for all native and custom data areas.
- MVP scope boundary and recommended implementation sequence.

## Key Decisions
- `service_listing` remains a custom post type owned by `servicesbg-listings`.
- `service_category` remains a hierarchical taxonomy owned by `servicesbg-listings`.
- Search is a separate module: `servicesbg-search`.
- Claims and SMS OTP state are owned by `servicesbg-claims`, with SMS provider calls routed through `servicesbg-integrations`.
- Coverage is owned by `servicesbg-coverage`, not listings or search.
- Reservations and capacity are owned by `servicesbg-reservations`.
- Leads and inquiry CRM are owned by `servicesbg-leads-crm`.
- AI knowledge, AI chunks, AI training events, and AI feedback are owned by `servicesbg-ai` and deferred to Phase 2.
- Reviews are owned by `servicesbg-reviews` and deferred to Phase 2.
- Analytics and dynamic pricing signals are deferred to Phase 3.
- Redirect/SEO migration traceability is owned by `servicesbg-seo-migration`.

## Source Constraints Reflected
- Flynax 4.5.2 source database: `services_services`.
- Flynax prefix: `fl_`.
- Accounts: 189.
- Categories: 277.
- Listings: 2735 total, 2640 active.
- Temporary/quick-account candidates: 86.
- Active listings without phone: 7.
- Duplicate phone groups: 253 groups covering 1770 listings.
- Listing media rows: 6139.
- Missing media references: 92, with video/YouTube references separated from missing image files.

## Next Step
Review and approve the schema and module ownership boundary before any plugin scaffolding, schema migrations, or import implementation begins.

