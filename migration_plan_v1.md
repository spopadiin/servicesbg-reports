# servicesbg Migration Implementation Plan v1

## Purpose
This document defines a staged, reversible migration plan from Flynax 4.5.2 to a WordPress CRM/service marketplace.

It is a planning artifact only. Application coding starts only after the data model and architecture are approved.

## Constraints
- Work only inside `/opt/projects/servicesbg`.
- Do not modify production services.bg.
- Do not overwrite source archives or imported DB dumps.
- Preserve traceability from every Flynax ID to every WordPress ID.
- Keep every migration stage reversible.
- Document every migration decision.
- Do not mix this project with tradingbot.
- Do not use UNAI context.

## Source Inputs
- Source archive: `/opt/projects/servicesbg/archive/uploads/servicebg.tar.gz`
- Extracted source: `/opt/projects/servicesbg/archive/extracted/servicebg`
- Media root: `/opt/projects/servicesbg/archive/extracted/servicebg/files`
- Source DB dump: `/opt/projects/servicesbg/db/imports/localhost.sql.gz`
- Imported DB: `services_services`
- DB prefix: `fl_`

## Migration Strategy
The migration will be built as WP-CLI commands in a custom plugin after architecture approval. Commands must be idempotent and batch-aware.

Recommended import order:
1. Preflight source verification
2. WordPress schema setup
3. Categories
4. Users/accounts/provider profiles
5. Service listings
6. Listing-category ownership/link validation
7. Media
8. Coverage zone suggestions
9. Claims seed data
10. Optional conversations/messages
11. Optional booking/reservation history
12. SEO redirect mapping
13. Validation reports

## Phase 0: Architecture Approval
Status: current phase.

Deliverables:
- `docs/data_model_v1.md`
- `docs/migration_plan_v1.md`
- `docs/wordpress_plugin_architecture_v1.md`
- `reports/latest.md`
- `reports/latest.json`

Exit criteria:
- Data model approved.
- WordPress plugin architecture approved.
- Source questions prioritized.
- No production systems touched.

## Phase 1: Source Schema Audit
Goal: confirm exact table schemas and relationships before writing importers.

Tasks:
- Capture `SHOW TABLES LIKE 'fl_%'`.
- Capture `DESCRIBE` output for `fl_accounts`, `fl_categories`, `fl_listings`.
- Identify actual primary keys and relation columns.
- Locate message, booking, review/testimonial, media-reference, and URL/slug tables.
- Sample 20 listings across active, expired, trash, and incomplete statuses.
- Sample accounts for `provider_company`, `account_freelancer`, and `visitor_registered`.
- Sample category hierarchy depth and slug/key fields.
- Confirm source encoding and date/time formats.

Outputs:
- `reports/source_schema_audit.md`
- Updated `reports/latest.md` and `reports/latest.json`

Validation:
- Counts match existing inventory: 189 accounts, 277 categories, 2735 listings, 2640 active.
- No source archive or DB dump is modified.

## Phase 2: WordPress Schema Setup
Goal: create target structures without importing live data.

Tasks:
- Create `servicesbg-core` plugin.
- Register `service_listing` CPT.
- Register `service_category` taxonomy.
- Register custom roles and capabilities.
- Create custom tables:
  - `wp_servicesbg_import_batches`
  - `wp_servicesbg_migration_map`
  - `wp_servicesbg_provider_profiles`
  - `wp_servicesbg_claims`
  - `wp_servicesbg_availability_rules`
  - `wp_servicesbg_reservations`
  - `wp_servicesbg_conversations`
  - `wp_servicesbg_messages`
  - `wp_servicesbg_leads`
  - `wp_servicesbg_coverage_zones`
  - `wp_servicesbg_media_map`
  - `wp_servicesbg_redirect_map`
  - AI knowledge tables
  - audit/event table
- Add schema version tracking.

Rollback:
- Deactivate plugin.
- Drop only plugin-owned custom tables for a specific empty/test environment after confirmation.
- Delete imported WordPress objects only by migration batch map.

Validation:
- Plugin activation creates expected tables.
- Plugin deactivation does not delete data.
- Schema setup can be run repeatedly without duplicating tables or roles.

## Phase 3: Category Import
Goal: import all Flynax categories into `service_category`.

Tasks:
- Read `fl_categories`.
- Normalize names, slugs, parent IDs, and active flags.
- Insert parent terms before child terms.
- Store `_servicesbg_source_category_id`, source parent ID, source key/path, and batch ID.
- Write records to `wp_servicesbg_migration_map`.
- Generate initial category redirect candidates.

Rollback:
- Delete terms created by the batch using migration map.
- Remove related redirect candidates for that batch.

Validation:
- 277 categories imported or explicitly skipped with reasons.
- Parent-child counts match source.
- No orphan terms unless source has orphan categories, which must be reported.

## Phase 4: Account and Provider Import
Goal: import accounts into WordPress identities and provider profiles.

Tasks:
- Read `fl_accounts`.
- Map account types:
  - `provider_company` to role `service_provider`
  - `account_freelancer` to role `service_freelancer`
  - `visitor_registered` to role `service_customer`
- Generate safe usernames if source usernames conflict or are invalid.
- Import email only if valid and unique; otherwise store source email in profile/audit and create a placeholder login identity.
- Normalize phone numbers and store raw phone.
- Create provider profile rows for provider/freelancer accounts.
- Store source account ID in user meta and migration map.

Rollback:
- Delete users created by the migration batch only if they have not been claimed or modified after import.
- Disable rather than delete claimed/modified accounts.

Validation:
- 189 source accounts accounted for as imported/skipped/needs_review.
- Role counts match account-type counts.
- Duplicate email/phone conflicts reported.

## Phase 5: Service Listing Import
Goal: import Flynax listings into `service_listing` CPT.

Tasks:
- Read `fl_listings`.
- Create one post per listing.
- Map title, description, service fields, price, duration, conditions, certificates, partner, phone, location, and geodata.
- Assign provider owner by source account mapping, or importer placeholder owner if unresolved.
- Assign category by source category mapping.
- Map active listings to public status and non-active listings to review/private statuses based on approved policy.
- Set claim status:
  - `unclaimed_imported` for imported provider-owned listings not yet verified in WordPress.
  - `manual_review` for missing phone, owner conflict, invalid phone, or unresolved owner.
- Store all source IDs and batch IDs.

Rollback:
- Delete posts created by the batch if not claimed or edited after import.
- Preserve migration map with rollback status.

Validation:
- 2735 listings accounted for.
- 2640 active listings are public or queued according to approved policy.
- Every listing has source ID, owner mapping status, category mapping status, and claim status.
- Location/geodata preservation report created.

## Phase 6: Media Import
Goal: import listing and account media into the WordPress Media Library.

Tasks:
- Walk media root in read-only mode.
- Match listing media using `files/YYYY-MM/adXXXX/` and source listing IDs.
- Match account media from `files/account-media/`.
- Compute checksums before import.
- Import images/files through WordPress media APIs.
- Attach listing media to `service_listing`.
- Set first valid listing image as featured image.
- Create records in `wp_servicesbg_media_map`.

Rollback:
- Delete attachments created by the batch using media map if not reused by later batches.

Validation:
- Count imported, duplicate, missing, unsupported, and failed files.
- Every imported attachment has source path and checksum.
- Media failures do not block listing import but are reported.

## Phase 7: Claims by SMS OTP
Goal: seed imported listings for claim workflow.

Tasks:
- Normalize listing phones.
- Mark claim eligibility.
- Create no OTPs during migration.
- Add claim status meta to listings.
- Flag listings with missing/invalid phones for enrichment or manual review.
- Define staff queue views for owner conflicts and missing phones.

Runtime workflow after launch:
1. Provider opens listing claim page.
2. System checks claim eligibility and phone.
3. SMS OTP is sent to listing phone.
4. Provider enters code before expiration.
5. Ownership transfers to verified provider account.
6. Claim row and audit event are finalized.

Validation:
- Every imported listing has a claim status.
- No automatic ownership transfer occurs during import.
- No enriched phone is treated as verified.

## Phase 8: Reservation and Calendar Setup
Goal: prepare calendar functionality for every provider.

Tasks:
- Create default availability rules for imported providers only after policy approval.
- Import historical/future booking plugin data only if source schema is confirmed and data is reliable.
- Link reservation requests to leads and conversations.
- Build capacity model with provider default capacity and optional listing-specific rules.

Validation:
- Providers can have availability with no listing duplication.
- Reservation state transitions are auditable.
- Overlap/capacity rules are testable.

## Phase 9: Conversations and Messaging
Goal: preserve useful messaging data and implement improved internal messaging.

Tasks:
- Locate Flynax message tables.
- Decide whether to import historical messages based on source data quality and privacy review.
- Create conversations linked to listing/provider/customer/lead.
- Import messages with source message IDs where approved.
- Mark AI/system/imported messages explicitly.

Validation:
- Imported messages preserve sender, recipient, timestamp, and source ID where available.
- Conversation counts and orphan messages are reported.
- AI-generated content is distinguishable from human messages.

## Phase 10: AI Chatbot Learning Layers
Goal: prepare a controlled knowledge model for provider chatbots.

Tasks:
- Create global knowledge sources for platform policies.
- Create category knowledge source records from category descriptions and approved staff content.
- Create provider knowledge source records from imported provider profiles and listings.
- Add provider FAQ/document/website ingestion as post-launch workflow.
- Store source status: draft, approved, rejected, stale.

Validation:
- No private conversation text becomes training/knowledge without approval.
- Every knowledge source has owner scope and provenance.
- Provider-specific assistant can be disabled per provider.

## Phase 11: Coverage Zones
Goal: migrate existing location data and prepare improved coverage logic.

Tasks:
- Preserve listing/provider country, region, city, latitude, and longitude.
- Create suggested coverage zones from imported location data where confidence is high.
- Mark ambiguous zones as `needs_review`.
- Support radius, city, region, polygon, and travel-fee zone types.
- Later, use lead/reservation data for coverage discovery suggestions.

Validation:
- Every listing has preserved imported location fields.
- Coverage suggestions retain source listing/account IDs.
- No imported suggestion silently becomes a verified provider policy unless confidence criteria are approved.

## Phase 12: Leads / Inquiries CRM
Goal: create the CRM foundation for conversion tracking.

Tasks:
- Create lead records from new inquiry forms, reservations, and messages.
- Import historical inquiries only if relevant source tables exist and are approved.
- Link every lead to provider/listing/customer when possible.
- Track lead source and status.
- Prepare fields for future conversion score and dynamic pricing.

Validation:
- Lead creation is automatic for inquiry and reservation flows.
- Lead status transitions are auditable.
- Lead records support anonymous and registered customers.

## Phase 13: SEO and Redirect Mapping
Goal: preserve discoverability and minimize broken URLs.

Tasks:
- Identify Flynax URL rules from config, templates, and sample public URLs.
- Generate old-to-new redirects for listings, categories, providers, and key static pages.
- Store confidence values: exact, reconstructed, manual_review.
- Export redirect map for web server or WordPress redirect handler.
- Crawl target staging site and validate destination status.

Validation:
- Every public imported listing has a redirect candidate or explicit exception.
- Every imported category has a redirect candidate or explicit exception.
- Conflicting redirects are reported.
- Redirects use 301 only after final staging validation.

## Phase 14: Dry Runs
Goal: prove importers before full migration.

Dry run sequence:
1. Schema-only run
2. 10-category sample
3. 20-account sample
4. 50-listing sample with media
5. Full category/account/listing import without media
6. Full import with media
7. Redirect validation crawl

Each dry run must produce:
- batch ID
- input counts
- imported counts
- skipped counts
- failed counts
- validation errors
- rollback test result

## Phase 15: Full Migration
Goal: run complete staged migration against the approved WordPress staging environment.

Tasks:
- Freeze source snapshot for final migration.
- Run preflight checks.
- Execute import phases in approved order.
- Run full validation suite.
- Produce final redirect map.
- Produce cutover readiness report.

Exit criteria:
- Counts reconciled.
- Critical validation issues resolved.
- Rollback plan tested.
- SEO redirect map approved.
- Production cutover checklist approved.

## Validation Strategy

### Count Validation
- Accounts: source vs users/profile rows.
- Categories: source vs taxonomy terms.
- Listings: source vs posts by status.
- Media: filesystem candidates vs attachments/map records.
- Redirects: public source objects vs redirect rows.

### Relationship Validation
- Listing owner exists or is explicitly placeholder/manual review.
- Listing category exists or is explicitly unresolved.
- Listing media attaches to correct listing.
- Provider profile links to user.
- Claims link to listing and claimant/current owner when applicable.
- Reservation links to listing/provider/lead/conversation.

### Data Quality Validation
- Duplicate emails.
- Duplicate phones.
- Invalid/missing phone numbers.
- Missing titles/descriptions.
- Missing categories.
- Invalid coordinates.
- Broken media paths.
- Encoding/HTML cleanup issues.

### SEO Validation
- Old URL candidates resolve to new URLs.
- New URLs return expected status.
- Slugs are stable and not duplicated unexpectedly.
- Manual-review redirects are separated from exact redirects.

### Reversibility Validation
- Every imported object has a migration map row.
- Batch rollback deletes or disables only batch-created objects.
- Claimed/modified objects are protected from destructive rollback.
- Source archives and dumps remain unchanged.

## Migration Decisions
- WordPress posts/taxonomies/users are used for public content and identities.
- Custom tables are used for stateful CRM workflows and traceability.
- Claims are seeded during import but OTP verification happens only at runtime.
- Listings without verified phones are not automatically claimable.
- Media is imported checksum-first and source paths are retained.
- Historical messages/bookings/reviews are conditional imports pending schema audit.
- Lead records are the common CRM layer for inquiries, messages, and reservations.
- Future pricing/conversion logic is supported by reservation and inquiry intensity data, but listing publishing remains free.

## Reports
Every major task must update:
- `reports/latest.md`
- `reports/latest.json`

Additional recommended reports:
- `reports/source_schema_audit.md`
- `reports/migration_dry_run_<batch>.md`
- `reports/validation_<batch>.md`
- `reports/redirect_map_summary.md`

