# ServicesBG Plugin Table Ownership v1

## Purpose
This document assigns ownership of every ServicesBG data area to a WordPress plugin/module. It prevents business logic from drifting into the theme, WordPress core, or unrelated modules.

## Ownership Rules
- Every custom table has one write owner plugin.
- Other modules may read through public service APIs, not by owning writes.
- Migration importers write through owner module services where practical; bulk import exceptions must still be logged in `wp_servicesbg_migration_map`.
- Deactivating a plugin must not delete production data.
- Schema changes must be versioned per owning plugin.
- WordPress core owns storage mechanics for native tables, but ServicesBG plugins own the business meaning of their rows and meta keys.

## Module List

| Module | Responsibility |
| --- | --- |
| `servicesbg-core` | Shared schema/version registry, audit logging, capabilities, common status vocabulary. |
| `servicesbg-migration` | Staged imports, migration maps, media source maps, batch rollback orchestration. |
| `servicesbg-providers` | Provider/freelancer profiles, source account mapping, verification data. |
| `servicesbg-listings` | `service_listing` CPT, `service_category` taxonomy, listing meta policy. |
| `servicesbg-claims` | Claim listing by phone, SMS OTP claim state, ownership transfer workflow. |
| `servicesbg-reservations` | Availability, capacity, reservation requests and confirmed bookings. |
| `servicesbg-messaging` | Conversations and messages. |
| `servicesbg-leads-crm` | Leads, inquiries, lead lifecycle events, CRM workflows. |
| `servicesbg-coverage` | Coverage zones, future inferred coverage suggestions. |
| `servicesbg-search` | Dedicated search indexing, search query logging, search result ranking inputs. |
| `servicesbg-reviews` | Provider/listing reviews and moderation workflow. |
| `servicesbg-ai` | AI knowledge layers, training events, provider assistants, AI feedback. |
| `servicesbg-seo-migration` | Redirect maps, migrated URL validation, SEO handoff to Rank Math/Yoast if selected. |
| `servicesbg-integrations` | SMS, email, maps/geocoding, webhook, LLM, and notification provider adapters. |
| `servicesbg-analytics` | Marketplace event tracking, reporting, pricing signals. |
| `servicesbg-monetization` | Future payment/conversion/audit model. |
| `servicesbg-admin` | Admin screens and operational tools; does not own core data tables except UI settings. |

## WordPress Native Object Ownership

| Object/table | Business owner | Notes |
| --- | --- | --- |
| `wp_users` provider/freelancer users | `servicesbg-providers` | WordPress stores accounts; provider module defines roles/profile semantics. |
| `wp_users` customer users | `servicesbg-leads-crm` with `servicesbg-providers` account helpers | Customer creation may come from leads/reservations/messages. |
| `wp_usermeta` `_servicesbg_source_account_id` and provider keys | `servicesbg-providers` | Migration may populate during import. |
| `wp_posts` `service_listing` | `servicesbg-listings` | Post type is plugin-owned. |
| `wp_postmeta` listing source/pricing/location/claim hint keys | `servicesbg-listings` | Claim module owns claim table state; listing meta only mirrors summary state. |
| `wp_terms`, `wp_term_taxonomy` `service_category` | `servicesbg-listings` | Hierarchical imported categories. |
| `wp_termmeta` source category traceability | `servicesbg-listings` | Migration populates through listing/category import. |
| `wp_posts` attachments | WordPress core plus `servicesbg-migration` media map | WordPress owns media objects; media source traceability is custom. |
| `wp_options` plugin schema options | Each plugin | Option names must be module-prefixed. |

## Custom Table Ownership Matrix

| Table | Owner plugin | Phase | Primary writers | Primary readers | Notes |
| --- | --- | --- | --- | --- | --- |
| `wp_servicesbg_import_batches` | `servicesbg-migration` | MVP | Migration tooling | Admin, all import validators | Batch root for staged/reversible migration. |
| `wp_servicesbg_migration_map` | `servicesbg-migration` | MVP | Migration tooling | All modules, admin, rollback | Canonical Flynax-to-WP traceability. |
| `wp_servicesbg_audit_log` | `servicesbg-core` | MVP | All modules through core logger | Admin/reporting | Cross-module immutable audit events. |
| `wp_servicesbg_provider_profiles` | `servicesbg-providers` | MVP | Providers, migration | Listings, claims, reservations, CRM, search | Source account IDs preserved here. |
| `wp_servicesbg_claims` | `servicesbg-claims` | MVP | Claims workflow | Listings, providers, admin, audit | Owns SMS OTP claim state. |
| `wp_servicesbg_availability_rules` | `servicesbg-reservations` | MVP | Reservation/provider calendar UI | Search, leads, admin | Recurring provider/listing capacity. |
| `wp_servicesbg_capacity_exceptions` | `servicesbg-reservations` | MVP | Reservation/provider calendar UI | Search, leads, admin | Closures and temporary overrides. |
| `wp_servicesbg_reservations` | `servicesbg-reservations` | MVP | Reservation workflow | Leads, analytics, providers, admin | Future monetization input. |
| `wp_servicesbg_conversations` | `servicesbg-messaging` | MVP | Messaging workflow | Leads, AI, admin | Thread-level state. |
| `wp_servicesbg_messages` | `servicesbg-messaging` | MVP | Messaging workflow/import if approved | AI, leads, admin | Imported Flynax messages require privacy approval. |
| `wp_servicesbg_leads` | `servicesbg-leads-crm` | MVP | Forms, messaging, reservations, import if approved | Providers, analytics, admin, AI | CRM source of truth for inquiries. |
| `wp_servicesbg_coverage_zones` | `servicesbg-coverage` | MVP | Coverage UI/import suggestions | Search, listings, leads | Coverage is not owned by search/listings. |
| `wp_servicesbg_media_map` | `servicesbg-migration` | MVP | Media importer | Listings, admin, validators | WordPress attachments remain native. |
| `wp_servicesbg_redirect_map` | `servicesbg-seo-migration` | MVP | Redirect importer/SEO validator | Admin, redirect handler, analytics | Can integrate with Rank Math/Yoast but owns migrated URL map. |
| `wp_servicesbg_search_index` | `servicesbg-search` | MVP | Search indexer | Search UI/API | Dedicated module; rebuilt from authoritative modules. |
| `wp_servicesbg_search_queries` | `servicesbg-search` | MVP | Search UI/API | Analytics, admin | MVP search quality feedback. |
| `wp_servicesbg_reviews` | `servicesbg-reviews` | Phase 2 | Review workflow | Listings, providers, search, admin | Custom table for listing/provider/reservation traceability. |
| `wp_servicesbg_lead_events` | `servicesbg-leads-crm` | Phase 2 | CRM workflows | Analytics, admin | Timeline and future pricing signal source. |
| `wp_servicesbg_provider_verifications` | `servicesbg-providers` | Phase 2 | Verification workflow | Listings, claims, search, admin | Separate from basic SMS claim. |
| `wp_servicesbg_ai_knowledge_sources` | `servicesbg-ai` | Phase 2 | AI/admin/provider tools | AI runtime, admin | Provider/category/global knowledge layers. |
| `wp_servicesbg_ai_knowledge_chunks` | `servicesbg-ai` | Phase 2 | AI indexing pipeline | AI runtime | Embedding refs may point to external vector store. |
| `wp_servicesbg_ai_training_events` | `servicesbg-ai` | Phase 2 | Approved learning workflows | AI runtime, admin | Explicit training/learning audit. |
| `wp_servicesbg_ai_feedback` | `servicesbg-ai` | Phase 2 | AI feedback UI/runtime | AI admin/reporting | Quality control loop. |
| `wp_servicesbg_notifications` | `servicesbg-integrations` | Phase 2 | Integration adapters | Claims, reservations, messaging, admin | SMS/email delivery audit. |
| `wp_servicesbg_analytics_events` | `servicesbg-analytics` | Phase 3 | Event collectors | Reporting, monetization | Broad marketplace analytics. |
| `wp_servicesbg_pricing_signals` | `servicesbg-analytics` | Phase 3 | Aggregation jobs | Monetization, admin | Dynamic conversion/pricing input. |
| `wp_servicesbg_coverage_discoveries` | `servicesbg-coverage` | Phase 3 | Coverage inference jobs | Providers, search, admin | Suggested areas, not authoritative zones. |
| `wp_servicesbg_external_integration_events` | `servicesbg-integrations` | Phase 3 | API adapters/webhooks | Admin, audit, analytics | Durable external API traceability. |
| `wp_servicesbg_payment_audit` | `servicesbg-monetization` | Phase 3 | Migration/future billing | Analytics, admin | Future monetization and historical plan/payment audit. |

## Domain Ownership Details

### Listings
- Plugin: `servicesbg-listings`.
- Uses native `wp_posts`, `wp_postmeta`, `wp_terms`, `wp_term_taxonomy`, `wp_termmeta`.
- Does not own reservations, claims, coverage, search, or leads.
- Must expose listing read models/events for search, coverage, claims, leads, and AI.

### Provider and Freelancer Profiles
- Plugin: `servicesbg-providers`.
- Owns `wp_servicesbg_provider_profiles` and provider verification rows.
- WordPress users remain native.
- Flynax account IDs must be preserved in both user meta and provider profile rows.

### Claim Listing by SMS OTP
- Plugin: `servicesbg-claims`.
- Owns `wp_servicesbg_claims`.
- Uses `servicesbg-integrations` for SMS delivery adapters.
- Claim transfers must update listing author only after OTP verification and conflict checks.

### Reservations, Availability, and Capacity
- Plugin: `servicesbg-reservations`.
- Owns availability, capacity exceptions, and reservation tables.
- Reads provider profiles, listings, claims, coverage, and leads.
- Booking plugin source data is effectively empty, so target reservation logic is custom.

### Messaging
- Plugin: `servicesbg-messaging`.
- Owns conversation/message tables.
- Reads listings/providers/leads and may create/update leads through CRM APIs.
- Historical Flynax `fl_messages` import requires privacy approval.

### Leads and CRM
- Plugin: `servicesbg-leads-crm`.
- Owns leads and lead event timelines.
- Reads listings, providers, reservations, messages, search source context.
- Lead creation must preserve contact source IDs when importing `fl_contacts`.

### Coverage
- Plugin: `servicesbg-coverage`.
- Owns coverage zones and coverage discoveries.
- Search consumes coverage but does not own or mutate it.
- Imported Flynax location/geodata creates suggestions or seed zones, not automatically verified service coverage.

### Search
- Plugin: `servicesbg-search`.
- Owns search index and query log tables.
- Search is dedicated because it spans listings, categories, providers, coverage, claims, reviews, and future ranking signals.
- It must rebuild from source-of-truth modules and never become the source of truth.
- Search implementation may use WordPress SQL/fulltext in MVP, then external search such as OpenSearch/Meilisearch later if needed. The module owns that adapter boundary.

### Reviews
- Plugin: `servicesbg-reviews`.
- Owns reviews table in Phase 2.
- Reviews should not be implemented as theme comments because provider/listing/reservation traceability and moderation are core marketplace data.

### AI Knowledge and Training
- Plugin: `servicesbg-ai`.
- Owns knowledge sources, chunks, training events, and feedback.
- Uses `servicesbg-integrations` for external LLM/vector providers.
- Private imported messages cannot be used for training unless approved and anonymized.

### Analytics
- Plugin: `servicesbg-analytics`.
- Owns analytics events and pricing signals in Phase 3.
- MVP uses lightweight search query logging and operational audit only.

### Redirects and SEO Migration
- Plugin: `servicesbg-seo-migration`.
- Owns redirect map and migration validation.
- SEO plugin candidates such as Rank Math or Yoast may own page metadata, sitemaps, and UI, but migrated redirect traceability stays in ServicesBG tables.

### External Integrations
- Plugin: `servicesbg-integrations`.
- Owns notification queue and external integration event audit.
- Provides adapters for SMS gateways, email delivery, map/geocoding APIs, LLM APIs, and webhooks.
- Business workflows call integration services rather than embedding provider-specific calls.

## Cross-Module Write Policy

| Scenario | Allowed writer | Required side effects |
| --- | --- | --- |
| Import Flynax listing | `servicesbg-migration` through `servicesbg-listings` | Migration map, listing source meta, audit if non-idempotent. |
| Claim listing ownership | `servicesbg-claims` | Claim row, audit log, listing author/status update through listings API. |
| Send claim SMS | `servicesbg-claims` via `servicesbg-integrations` | Claim OTP state, notification/integration event. |
| Create inquiry lead from form | `servicesbg-leads-crm` | Lead row, optional conversation, audit/anti-spam result. |
| Create reservation from lead | `servicesbg-reservations` | Reservation row, lead relationship update through CRM API. |
| Index listing for search | `servicesbg-search` | Search index row only; no mutation of listing authority. |
| Add coverage zone | `servicesbg-coverage` | Coverage row, search reindex event. |
| AI answer in conversation | `servicesbg-ai` via `servicesbg-messaging` | Message row, AI feedback/training references only if approved. |
| SEO redirect hit tracking | `servicesbg-seo-migration` | Redirect map hit counters; analytics event later if Phase 3 enabled. |

## Vendor Lock-In and Data Ownership
- Custom marketplace data stays in ServicesBG-owned tables.
- External APIs may be used for SMS, maps, email, LLMs, vectors, and future search, but provider IDs/responses must be recorded in integration-owned audit tables.
- Existing WordPress plugins may be used for SEO, caching, backups, and simple form rendering, but core marketplace records must not be trapped in proprietary plugin schemas.
- Search must retain a ServicesBG-owned SQL index even if an external search engine is added later.

