# servicesbg Architecture Planning Summary

## Status
Architecture and data-model planning is complete. No application code has been written for this phase.

## Inputs Reviewed
- `AGENTS.md`
- `PROJECT.md`
- `ROADMAP.md`
- `docs/architecture_v1.md`
- `reports/source_inventory.md`
- `memory/feature_requirements_v2.json`
- `handoff/handoff.json`

## Planning Deliverables Created
- `docs/data_model_v1.md`
- `docs/migration_plan_v1.md`
- `docs/wordpress_plugin_architecture_v1.md`
- `reports/latest.md`
- `reports/latest.json`

## Source Baseline
- Platform: Flynax 4.5.2
- Database: `services_services`
- Prefix: `fl_`
- Template: `general_wide`
- Accounts: 189
- Categories: 277
- Listings: 2735 total / 2640 active
- Media root: `/opt/projects/servicesbg/archive/extracted/servicebg/files`

## Key Data Model Decisions
- WordPress users represent providers, freelancers, and customers.
- Provider/freelancer details are stored in a custom provider profile table linked to WordPress users.
- Public service listings use CPT `service_listing`.
- Service categories use hierarchical taxonomy `service_category`.
- Claims, reservations, conversations, messages, leads, coverage zones, AI knowledge, media mapping, redirect mapping, and migration traceability use custom tables.
- Every imported account, category, listing, media object, and optional message/booking record must preserve Flynax source IDs and migration batch IDs.

## Migration Plan Decisions
- Migration is staged, idempotent, and reversible by batch.
- Source archives and imported DB dumps are read-only migration inputs.
- Import order is source audit, schema setup, categories, accounts, listings, media, coverage suggestions, claim seed data, optional messages/bookings, redirects, and validation.
- Claims are seeded during migration, but SMS OTP verification and ownership transfer happen only at runtime.
- Listings without usable source phones are marked for enrichment or manual review, not automatic claim transfer.
- Media import is checksum-based and source paths are retained.
- SEO redirect mapping uses confidence levels: exact, reconstructed, manual review.

## Plugin Architecture Decisions
- Planned plugin name: `servicesbg-core`.
- Runtime features and migration tooling are separated inside the plugin architecture.
- WP-CLI commands are planned under `wp servicesbg`.
- Rollback commands operate from `wp_servicesbg_migration_map` and batch IDs.
- Claimed, edited, or customer-touched records are protected from destructive rollback.

## Required Next Step
Review and approve the data model, migration plan, and plugin architecture. After approval, run a source schema audit before writing migration/plugin code.
