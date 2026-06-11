# servicesbg Latest Report

## Status
Architecture/data-model planning deliverables created. Ready for review before any application coding.

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

## Source Counts
- Accounts: 189
- Categories: 277
- Listings total: 2735
- Active listings: 2640

## New Deliverables
- `docs/data_model_v1.md`
- `docs/migration_plan_v1.md`
- `docs/wordpress_plugin_architecture_v1.md`

## Key Decisions Documented
- Use WordPress users for providers, freelancers, and customers.
- Use CPT `service_listing` for public service listings.
- Use hierarchical taxonomy `service_category` for imported Flynax categories.
- Use custom tables for claims, reservations, conversations, leads, coverage zones, AI knowledge, media mapping, redirects, and migration traceability.
- Preserve Flynax IDs and batch IDs on imported objects.
- Seed claim eligibility during migration, but perform SMS OTP verification only at runtime.
- Keep imports staged, idempotent, and reversible by migration batch.

## Next Step
Review and approve the data model and migration/plugin architecture. After approval, perform source schema audit before writing migration/plugin code.
Thu Jun 11 08:33:37 PM EEST 2026
