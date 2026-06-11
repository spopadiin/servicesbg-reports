# ServicesBG Latest Report

Updated: 2026-06-11T22:27:35+03:00

## Status
Plugin contracts, inter-plugin events, REST API architecture, background jobs, and queue architecture are now designed for the ServicesBG WordPress CRM marketplace migration.

No application code was written. No production services.bg system was modified.

## Current Deliverables
- `docs/plugin_contracts_events_api_jobs_v1.md`
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
- `docs/servicesbg_database_schema_v1.md`
- `docs/servicesbg_plugin_table_ownership_v1.md`
- `docs/servicesbg_mvp_scope_v1.md`

## What Was Added
- Contract types for services, commands, queries, events, jobs, and external adapters.
- Module-by-module contract inventory across core, migration, listings, providers, claims, reservations, messaging, CRM, coverage, search, reviews, AI, SEO migration, integrations, analytics, and monetization.
- Versioned event envelope with correlation, causation, source ID, and idempotency fields.
- Core inter-plugin event list and producer/consumer responsibilities.
- REST API namespace and endpoint groups under `/wp-json/servicesbg/v1`.
- Authentication and authorization model for public, provider, customer, admin, and migration APIs.
- Background job architecture with job envelope, queue categories, job types, retry policy, idempotency rules, and observability.
- Queue recommendation: own the ServicesBG queue contract in `servicesbg-core`, use Action Scheduler if approved, and add custom queue tables only if needed.

## Key Decisions
- Cross-module writes must go through service or command contracts.
- Events are explicit, namespaced, versioned, and idempotent.
- REST endpoints expose workflows, not raw database tables.
- Search remains a dedicated module with its own contracts and indexing jobs.
- SMS OTP delivery is a queued integration job, not a direct template or controller concern.
- Migration jobs remain staged, batch-based, idempotent, and rollback-aware.
- AI knowledge and training events are contractually separated from raw private messages.
- Action Scheduler is the preferred implementation candidate for queue execution, but ServicesBG owns the job contract to avoid lock-in.

## Next Step
Review and approve the contract/event/API/job boundaries before plugin scaffolding or implementation begins.

