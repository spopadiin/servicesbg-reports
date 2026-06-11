# ServicesBG MVP Scope v1

## Purpose
This document defines the database and module scope for the ServicesBG WordPress CRM marketplace MVP, and separates Phase 2 and Phase 3 schema from the launch-critical path.

## MVP Definition
The MVP should migrate services.bg from Flynax into WordPress with traceable service listings, providers/freelancers, categories, media, redirects, claims by SMS OTP, reservation-ready provider calendars, internal messaging foundations, lead CRM, coverage zones, and dedicated search.

The MVP does not require final monetization, advanced AI learning, full review operations, or external analytics dashboards.

## MVP Principles
- Preserve every Flynax ID needed for audit and rollback.
- Import active listings publicly only after validation.
- Keep expired/trash/incomplete listings non-public unless business approval changes the policy.
- Treat temporary Flynax accounts as unverified ownership.
- Treat duplicate phones as provider grouping signals, not automatic ownership proof.
- Keep search as a dedicated module.
- Keep business logic out of theme and WordPress core.
- Keep all migration batches reversible unless an object has been claimed, edited, reserved, or otherwise protected.

## MVP Modules

| Module | MVP status | Reason |
| --- | --- | --- |
| `servicesbg-core` | Required | Shared audit, schema versions, capabilities, status vocabulary. |
| `servicesbg-migration` | Required | Staged Flynax imports and rollback. |
| `servicesbg-providers` | Required | Provider/freelancer profile ownership and source account traceability. |
| `servicesbg-listings` | Required | Service listing CPT, category taxonomy, listing meta. |
| `servicesbg-claims` | Required | Claim imported listings by SMS OTP. |
| `servicesbg-reservations` | Required | Calendar/availability/reservation foundation. |
| `servicesbg-messaging` | Required | Internal conversation/message foundation. |
| `servicesbg-leads-crm` | Required | Inquiry and lead records. |
| `servicesbg-coverage` | Required | Provider/listing service areas. |
| `servicesbg-search` | Required | Dedicated listing/provider/category/coverage search index. |
| `servicesbg-seo-migration` | Required | Redirect map and SEO migration validation. |
| `servicesbg-integrations` | Required but narrow | SMS adapter for claims; optional email/geocode stubs. |
| `servicesbg-admin` | Required | Admin review screens and operational tooling. |
| `servicesbg-reviews` | Phase 2 | Important marketplace trust layer, but not required for first migration cutover. |
| `servicesbg-ai` | Phase 2 | Architecture must reserve schema; MVP may expose only simple non-training assistant later. |
| `servicesbg-analytics` | Phase 3 | Search query logs and audit are enough for MVP. |
| `servicesbg-monetization` | Phase 3 | Future conversion/pricing model depends on reservation and inquiry history. |

## MVP Schema

### Native WordPress
- `wp_users` for imported provider/freelancer/customer accounts.
- `wp_usermeta` for source account IDs and provider/customer import metadata.
- `wp_posts` for `service_listing` and media attachments.
- `wp_postmeta` for listing source IDs, source status, phone, pricing, location, claim summary, import metadata.
- `wp_terms`, `wp_term_taxonomy`, `wp_term_relationships`, `wp_termmeta` for `service_category`.
- `wp_options` for module schema versions and feature flags.

### Required MVP Custom Tables

| Table | MVP purpose |
| --- | --- |
| `wp_servicesbg_import_batches` | Track staged imports, validation runs, dry runs, rollbacks. |
| `wp_servicesbg_migration_map` | Preserve Flynax-to-WordPress/custom target mappings. |
| `wp_servicesbg_audit_log` | Admin/import/claim/reservation/message audit. |
| `wp_servicesbg_provider_profiles` | Provider/freelancer profile data and source account traceability. |
| `wp_servicesbg_claims` | SMS OTP claim workflow and ownership transfer state. |
| `wp_servicesbg_availability_rules` | Provider/listing recurring availability. |
| `wp_servicesbg_capacity_exceptions` | One-off availability/capacity overrides. |
| `wp_servicesbg_reservations` | Reservation requests and confirmed slots. |
| `wp_servicesbg_conversations` | Conversation threads. |
| `wp_servicesbg_messages` | Message rows; historical import conditional on privacy approval. |
| `wp_servicesbg_leads` | CRM lead/inquiry records. |
| `wp_servicesbg_coverage_zones` | Provider/listing service areas and imported suggestions. |
| `wp_servicesbg_media_map` | Source media to WP attachment traceability. |
| `wp_servicesbg_redirect_map` | Old Flynax URL to new WordPress URL mapping. |
| `wp_servicesbg_search_index` | Dedicated denormalized search index. |
| `wp_servicesbg_search_queries` | Search quality/query logs. |

## MVP Data Migration Scope

### Import in MVP
- `fl_categories` to `service_category`.
- `fl_account_types` as role/profile mapping reference.
- `fl_accounts` to users/provider profiles where needed.
- `fl_listings` to `service_listing`.
- `fl_listing_photos` to attachments/media map where files exist; video references as metadata.
- Listing/account location fields to listing meta, provider profile, and coverage suggestions.
- Original category/listing/page paths to redirect map where confidence is sufficient.
- Claim-ready phone fields with raw and normalized values.

### Conditional in MVP
- `fl_messages` into conversations/messages only after privacy/business approval.
- `fl_contacts` into leads only after privacy/business approval.
- Zero-listing accounts only if linked to messages, contacts, payment history, or business retention policy.
- Payment/plan data only as migration map/reference, not operational billing.

### Excluded from MVP Operational Behavior
- Full AI training and provider-specific learning.
- Public reviews and review moderation.
- Dynamic pricing or conversion billing.
- Advanced analytics dashboards.
- External search engine dependency.
- Automatic phone enrichment as verified data.

## MVP Treatment of Known Source Issues

| Source issue | MVP treatment |
| --- | --- |
| 86 temporary/quick-account candidates | Import ownership as unverified; require claim or manual review before trusted provider control. |
| 105 zero-listing accounts | Import only if linked to required history or approved complete account retention. |
| 7 active listings without phone | Mark `manual_review`; do not allow SMS claim until a phone is verified. |
| 253 duplicate-phone groups covering 1770 listings | Allow same verified phone to claim related listings, but retain duplicate group signal and admin review path. |
| 5 duplicate category key groups | Generate unique WordPress slugs using parent path/source ID; preserve original keys in term meta. |
| 92 missing media references | Separate YouTube/video references from true missing files; log failures in media map. |
| Booking tables empty | Build custom reservation schema; do not migrate nonexistent booking history. |
| Large location/multifield tables | Import referenced values or keep as lookup/reference; do not blindly create 125k taxonomy terms. |
| Expired/trash/incomplete listings | Preserve source IDs and URL traceability; keep non-public unless approved otherwise. |

## Phase 2 Scope

Phase 2 adds trust, AI learning, and deeper CRM workflows after migration stability.

Schema:
- `wp_servicesbg_reviews`
- `wp_servicesbg_lead_events`
- `wp_servicesbg_provider_verifications`
- `wp_servicesbg_ai_knowledge_sources`
- `wp_servicesbg_ai_knowledge_chunks`
- `wp_servicesbg_ai_training_events`
- `wp_servicesbg_ai_feedback`
- `wp_servicesbg_notifications`

Capabilities:
- Public review workflow with moderation.
- Provider verification beyond SMS claim.
- Lead lifecycle timeline and follow-up.
- Provider/category/global AI knowledge layers.
- Approved AI training events.
- SMS/email notification queue beyond claim OTP.

## Phase 3 Scope

Phase 3 adds marketplace intelligence, future monetization, and deeper external integrations.

Schema:
- `wp_servicesbg_analytics_events`
- `wp_servicesbg_pricing_signals`
- `wp_servicesbg_coverage_discoveries`
- `wp_servicesbg_external_integration_events`
- `wp_servicesbg_payment_audit`

Capabilities:
- Marketplace funnel analytics.
- Dynamic pricing/conversion signal model.
- Coverage suggestion automation from searches, leads, and reservations.
- Durable external API/webhook audit.
- Future monetization and historical payment audit.

## Recommended Implementation Sequence
1. Confirm schema ownership and MVP phase boundary.
2. Scaffold module skeletons and activation schema versioning only after architecture approval.
3. Implement migration preflight validators for Flynax counts, source hashes, and media root checks.
4. Implement `servicesbg-migration`, `servicesbg-listings`, and `servicesbg-providers` schema.
5. Import categories, accounts, provider profiles, and listings in dry-run mode.
6. Import media maps and attachments in staged batches.
7. Build redirect map and validate top URL patterns before cutover.
8. Implement claims/SMS OTP with a sandbox SMS provider adapter.
9. Implement coverage zones, reservation tables, lead tables, messaging tables, and dedicated search index.
10. Run rollback tests on each import batch.
11. Run validation reports against source counts, source IDs, relationships, public listing counts, media failures, redirects, and claim readiness.
12. Freeze MVP schema before any production migration attempt.

## MVP Exit Criteria
- All active Flynax listings are either imported as public `service_listing` posts or explicitly mapped to a blocked/manual-review status.
- Every Flynax listing ID has a migration map row.
- Every imported listing preserves source account and category IDs.
- Every imported category preserves source category ID and hierarchy trace.
- Every imported provider/freelancer preserves source account ID.
- Media import failures are counted and explainable.
- Redirect map covers listing/category/page URLs at an approved confidence level.
- SMS claim flow works against a non-production SMS provider or sandbox.
- Reservation, coverage, lead, messaging, and search tables can be created and validated without production side effects.
- Rollback has been tested on at least one dry-run batch for categories, listings, media, redirects, and custom CRM tables.

