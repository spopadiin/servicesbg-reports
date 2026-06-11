# servicesbg Data Model v1

## Purpose
This document defines the WordPress CRM/service marketplace data model for migrating services.bg from Flynax 4.5.2.

It is a planning artifact only. No application code has been written in this phase.

## Source Baseline
- Source platform: Flynax 4.5.2
- Source database: `services_services`
- Source prefix: `fl_`
- Active template: `general_wide`
- Accounts: 189
- Categories: 277
- Listings: 2735 total / 2640 active
- Media root: `/opt/projects/servicesbg/archive/extracted/servicebg/files`
- Listing media pattern: `files/YYYY-MM/adXXXX/`
- Account media pattern: `files/account-media/`

## Target Principles
- Use WordPress-native objects where they improve editorial/admin compatibility.
- Use custom tables for CRM workflows, state machines, audit logs, reservations, conversations, claims, coverage zones, and migration traceability.
- Preserve all Flynax source IDs and migration batch IDs.
- Keep imports staged and reversible.
- Do not blindly clone Flynax. Preserve valuable data, relationships, URLs, and media while rebuilding marketplace workflows.
- Do not modify production services.bg during migration.

## WordPress Object Overview

| Domain object | WordPress storage | Notes |
| --- | --- | --- |
| Provider company | `wp_users` + provider profile table | Role `service_provider`; company profile fields in custom table. |
| Freelancer | `wp_users` + provider profile table | Role `service_freelancer`; same profile structure with profile type. |
| Customer / visitor | `wp_users` | Role `service_customer` for registered users; anonymous inquiries allowed with CRM lead record. |
| Service listing | CPT `service_listing` | Main public marketplace item. |
| Service category | Taxonomy `service_category` | Hierarchical taxonomy migrated from `fl_categories`. |
| Location | listing meta + coverage zone tables | Imported source location preserved; normalized coverage modeled separately. |
| Reservation | Custom table | Time, capacity, state, source, and CRM linkage. |
| Conversation | Custom tables | Thread, participants, messages, attachments, AI visibility. |
| Claim | Custom table | SMS OTP, verification, ownership transfer, audit trail. |
| Coverage zone | Custom table | Radius, city/region, polygon, travel fee, priority. |
| Lead / inquiry | Custom table | CRM record tied to listing/provider/conversation/reservation. |
| Media | WordPress Media Library + mapping table | Imported files attached to listings/providers with source paths retained. |
| Review | Custom table or later CPT/table | Planned object; source review data has not yet been confirmed in inventory. |

## Roles and Capabilities

| Role | Purpose | Initial capabilities |
| --- | --- | --- |
| `administrator` | Platform owner | Full WordPress and servicesbg management. |
| `service_manager` | Marketplace staff | Moderate listings, claims, reservations, leads, and conversations. |
| `service_provider` | Company provider | Manage own profile, listings, coverage, calendar, messages, leads, knowledge base. |
| `service_freelancer` | Individual provider | Same as provider with freelancer profile type. |
| `service_customer` | Registered customer | Manage own profile, reservations, inquiries, conversations. |

Imported Flynax account types map as:

| Flynax account type | Target role | Profile type |
| --- | --- | --- |
| `provider_company` | `service_provider` | `company` |
| `account_freelancer` | `service_freelancer` | `freelancer` |
| `visitor_registered` | `service_customer` | `customer` |

## Service Listing CPT

### CPT
- Post type: `service_listing`
- Public: yes
- Has archive: yes
- Supports: title, editor, excerpt, thumbnail, revisions
- Author: provider/freelancer WordPress user, or an importer-owned placeholder user until claimed
- Status mapping:
  - active Flynax listing: `publish`
  - expired/trash/incomplete Flynax listing: import as non-public status for audit unless business approval says otherwise

### Core Fields

| Field | Storage | Source |
| --- | --- | --- |
| title | `post_title` | `fl_listings.service_title` |
| description | `post_content` | `fl_listings.service_about` |
| short summary | `post_excerpt` | derived from description |
| status | `post_status` + meta | Flynax listing status |
| provider owner | `post_author` | `fl_listings.Account_ID` through account mapping |
| category | `service_category` term relation | Flynax category relationship |
| featured image | `_thumbnail_id` | first imported listing image |

### Listing Meta

| Meta key | Purpose |
| --- | --- |
| `_servicesbg_source_listing_id` | Original `fl_listings.ID` or equivalent listing key. |
| `_servicesbg_source_account_id` | Original owner account ID. |
| `_servicesbg_import_batch_id` | Migration batch identifier. |
| `_servicesbg_import_status` | Imported, skipped, needs_review, failed. |
| `_servicesbg_flynax_status` | Raw source status. |
| `_servicesbg_claim_status` | `unclaimed_imported`, `claim_requested`, `phone_verified`, `ownership_transferred`, `rejected`, `manual_review`. |
| `_servicesbg_phone_raw` | Source phone as imported. |
| `_servicesbg_phone_normalized` | E.164 or normalized local canonical phone. |
| `_servicesbg_phone_verified` | Boolean verification flag. |
| `_servicesbg_price` | Imported price. |
| `_servicesbg_price_hour` | Imported `service_price_hour`. |
| `_servicesbg_price_type` | Derived pricing mode if available. |
| `_servicesbg_duration` | Imported `service_duration`. |
| `_servicesbg_conditions` | Imported `service_conditions`. |
| `_servicesbg_certificates` | Imported `service_certificates`. |
| `_servicesbg_provision_location` | Imported `service_provision_location`. |
| `_servicesbg_partner_name` | Imported `partner_name`. |
| `_servicesbg_partner_website` | Imported `partner_website`. |
| `_servicesbg_country` | Imported country. |
| `_servicesbg_country_level1` | Imported region. |
| `_servicesbg_country_level2` | Imported city/locality. |
| `_servicesbg_latitude` | Imported `Loc_latitude`. |
| `_servicesbg_longitude` | Imported `Loc_longitude`. |
| `_servicesbg_original_url` | Source URL if reconstructable. |
| `_servicesbg_redirect_target` | Final WordPress permalink used for redirect mapping. |

## Service Category Taxonomy

### Taxonomy
- Name: `service_category`
- Object type: `service_listing`
- Hierarchical: yes
- Public: yes

### Term Meta

| Meta key | Purpose |
| --- | --- |
| `_servicesbg_source_category_id` | Original `fl_categories.ID`. |
| `_servicesbg_source_parent_id` | Original parent category ID. |
| `_servicesbg_import_batch_id` | Migration batch identifier. |
| `_servicesbg_flynax_key` | Original category key/path if available. |
| `_servicesbg_original_url` | Source category URL if reconstructable. |
| `_servicesbg_redirect_target` | WordPress category archive URL. |

The full 277 category hierarchy must be imported before listings so listing category relations can be resolved deterministically.

## Provider Profile Model

### WordPress Users
Every provider, freelancer, and customer is a WordPress user. Imported users receive generated credentials and must not be treated as production-login-ready until the identity verification and password reset flow is confirmed.

User meta:

| Meta key | Purpose |
| --- | --- |
| `_servicesbg_source_account_id` | Original `fl_accounts.ID`. |
| `_servicesbg_source_account_type` | Flynax account type. |
| `_servicesbg_import_batch_id` | Migration batch identifier. |
| `_servicesbg_account_claim_status` | Imported, claimed, merged, disabled, needs_review. |
| `_servicesbg_phone_raw` | Source phone if present. |
| `_servicesbg_phone_normalized` | Normalized phone. |
| `_servicesbg_phone_verified` | Verification flag. |

### Provider Profile Table
Custom table: `wp_servicesbg_provider_profiles`

| Column | Purpose |
| --- | --- |
| `id` | Internal profile ID. |
| `user_id` | WordPress user ID. |
| `source_account_id` | Original Flynax account ID. |
| `profile_type` | `company`, `freelancer`, `customer`. |
| `display_name` | Public provider name. |
| `company_name` | Company name if applicable. |
| `phone_raw` | Original phone. |
| `phone_normalized` | Normalized phone. |
| `phone_verified_at` | Verification timestamp. |
| `email` | Contact email if available. |
| `website` | Provider website. |
| `address_raw` | Imported address/location text. |
| `country` | Country. |
| `region` | Region. |
| `city` | City/locality. |
| `latitude` | Latitude. |
| `longitude` | Longitude. |
| `profile_status` | `imported`, `active`, `disabled`, `needs_review`. |
| `created_at` / `updated_at` | Audit timestamps. |

## Claim by SMS OTP

Custom table: `wp_servicesbg_claims`

| Column | Purpose |
| --- | --- |
| `id` | Claim ID. |
| `listing_id` | WordPress service listing post ID. |
| `source_listing_id` | Flynax listing ID. |
| `claimant_user_id` | User requesting claim. |
| `current_owner_user_id` | Owner before transfer. |
| `phone_raw` | Phone used for claim. |
| `phone_normalized` | Normalized phone used for SMS. |
| `otp_hash` | Hashed one-time code. |
| `otp_expires_at` | Expiration timestamp. |
| `attempt_count` | Verification attempts. |
| `status` | Claim state. |
| `verification_provider` | SMS gateway identifier. |
| `verified_at` | Phone verification timestamp. |
| `transferred_at` | Ownership transfer timestamp. |
| `reviewed_by_user_id` | Staff reviewer for manual decisions. |
| `decision_notes` | Review notes. |
| `created_at` / `updated_at` | Audit timestamps. |

Decision:
- Automatic transfer is allowed only when an OTP is sent to and verified against the listing phone.
- Listings without a usable phone enter enrichment/manual review. Enriched phone candidates are not verified and cannot trigger automatic transfer until SMS verification succeeds.
- Claim records are immutable in meaning: status changes are also written to the audit log.

## Reservation and Calendar Model

### Availability Rules
Custom table: `wp_servicesbg_availability_rules`

| Column | Purpose |
| --- | --- |
| `id` | Rule ID. |
| `provider_user_id` | Provider/freelancer user. |
| `listing_id` | Optional listing-specific rule. |
| `weekday` | Day of week. |
| `start_time` / `end_time` | Local available time window. |
| `capacity` | Parallel capacity. |
| `timezone` | Provider timezone. |
| `effective_from` / `effective_to` | Rule validity. |
| `status` | active/inactive. |

### Reservations
Custom table: `wp_servicesbg_reservations`

| Column | Purpose |
| --- | --- |
| `id` | Reservation ID. |
| `listing_id` | Service listing. |
| `provider_user_id` | Provider. |
| `customer_user_id` | Registered customer if available. |
| `lead_id` | CRM lead/inquiry linkage. |
| `conversation_id` | Conversation linkage. |
| `start_at` / `end_at` | Reserved interval. |
| `timezone` | Reservation timezone. |
| `capacity_units` | Capacity consumed. |
| `status` | `requested`, `confirmed`, `rejected`, `cancelled`, `completed`, `no_show`. |
| `source` | marketplace, admin, imported, provider. |
| `customer_name` / `customer_phone` / `customer_email` | Snapshot contact data. |
| `notes` | Internal notes. |
| `created_at` / `updated_at` | Audit timestamps. |

Decision:
- Reservations are custom-table CRM records, not posts, because they need time-range queries, status transitions, capacity checks, and auditability.
- Every reservation should create or link to a lead/inquiry so future conversion metrics are measurable.

## Conversations and Messaging

### Conversations
Custom table: `wp_servicesbg_conversations`

| Column | Purpose |
| --- | --- |
| `id` | Conversation ID. |
| `listing_id` | Related listing. |
| `provider_user_id` | Provider participant. |
| `customer_user_id` | Customer if registered. |
| `lead_id` | CRM linkage. |
| `reservation_id` | Optional reservation linkage. |
| `status` | open, pending_provider, pending_customer, closed, archived. |
| `channel` | web, email, imported, ai_assisted. |
| `last_message_at` | Sort timestamp. |
| `created_at` / `updated_at` | Audit timestamps. |

### Messages
Custom table: `wp_servicesbg_messages`

| Column | Purpose |
| --- | --- |
| `id` | Message ID. |
| `conversation_id` | Conversation. |
| `sender_type` | customer, provider, staff, ai, system. |
| `sender_user_id` | User ID if applicable. |
| `body` | Message content. |
| `visibility` | participant, staff, internal_ai_note. |
| `ai_generated` | Boolean. |
| `ai_model` | Model identifier if generated. |
| `source_message_id` | Original Flynax message ID if imported. |
| `created_at` | Message timestamp. |

Decision:
- Flynax messages should be imported if retained in the source DB and mapped to conversations with source IDs preserved.
- AI messages must be marked and auditable; provider/customer visible messages must never be silently replaced by generated content.

## AI Chatbot Learning Layers

The AI assistant has three knowledge layers:

| Layer | Scope | Sources |
| --- | --- | --- |
| Global platform | Whole marketplace | Site policies, booking rules, common support answers, conversion patterns. |
| Category | Service category | Category descriptions, common Q&A, typical service constraints. |
| Provider-specific | Provider/listing | Provider profile, listings, coverage, calendar, FAQs, uploaded docs, website extracts, approved conversation learnings. |

Custom tables:
- `wp_servicesbg_ai_knowledge_sources`
- `wp_servicesbg_ai_knowledge_chunks`
- `wp_servicesbg_ai_feedback`

Required fields:
- source type, owner scope, source URL/path, approval status, extracted text hash, embedding/vector reference if used later, timestamps, and source Flynax IDs where applicable.

Decision:
- Provider-specific learning requires provider approval or staff-approved ingestion for external documents/websites.
- Conversations can inform future learning only through explicit approved summaries, not automatic raw-message training.

## Coverage Zones

Custom table: `wp_servicesbg_coverage_zones`

| Column | Purpose |
| --- | --- |
| `id` | Zone ID. |
| `provider_user_id` | Provider. |
| `listing_id` | Optional listing-specific zone. |
| `zone_type` | radius, city, region, polygon, excluded_area. |
| `label` | Provider-facing zone name. |
| `country` / `region` / `city` | Administrative coverage values. |
| `center_lat` / `center_lng` | Radius center. |
| `radius_km` | Radius. |
| `polygon_json` | Polygon geometry if used. |
| `travel_fee` | Optional surcharge. |
| `min_order_value` | Optional commercial rule. |
| `priority` | Match precedence. |
| `status` | active/inactive. |
| `source_listing_id` | Source trace if derived from listing. |
| `created_at` / `updated_at` | Audit timestamps. |

Decision:
- Imported location fields create initial coverage suggestions, not final authoritative zones unless the source clearly represents service coverage.
- Coverage discovery should later use inquiry/reservation data to suggest new zones, exclusions, and travel-fee boundaries.

## Leads / Inquiries CRM

Custom table: `wp_servicesbg_leads`

| Column | Purpose |
| --- | --- |
| `id` | Lead ID. |
| `listing_id` | Related listing. |
| `provider_user_id` | Provider. |
| `customer_user_id` | Registered customer if known. |
| `conversation_id` | Conversation linkage. |
| `reservation_id` | Reservation linkage. |
| `source` | listing_form, phone_click, message, reservation, admin, imported. |
| `status` | new, contacted, quoted, reserved, won, lost, spam, archived. |
| `customer_name` / `customer_phone` / `customer_email` | Snapshot contact details. |
| `service_location` | Customer requested location. |
| `requested_at` | Inquiry/request timestamp. |
| `value_estimate` | Optional value estimate. |
| `conversion_score` | Future scoring field. |
| `created_at` / `updated_at` | Audit timestamps. |

Decision:
- Leads are the unifying CRM object for messages, reservations, and future conversion pricing.
- Click/call events can later be stored as lead events or analytics events if measurable.

## Media Model

WordPress Media Library stores imported images/files.

Custom mapping table: `wp_servicesbg_media_map`

| Column | Purpose |
| --- | --- |
| `id` | Mapping ID. |
| `attachment_id` | WordPress attachment ID. |
| `owner_type` | listing, provider, message, knowledge_source. |
| `owner_id` | WordPress owner ID. |
| `source_path` | Original file path relative to source media root. |
| `source_checksum` | File checksum. |
| `source_listing_id` | Source listing if applicable. |
| `source_account_id` | Source account if applicable. |
| `import_batch_id` | Migration batch. |
| `status` | imported, missing, duplicate, failed. |
| `created_at` | Import timestamp. |

Decision:
- Source archives and imported DB dumps are never overwritten.
- Media import must be checksum-based and idempotent.
- The first valid listing image becomes featured image; all valid images remain attached in source order.

## SEO and Redirect Mapping

Custom table: `wp_servicesbg_redirect_map`

| Column | Purpose |
| --- | --- |
| `id` | Redirect map ID. |
| `source_type` | listing, category, provider, static_page. |
| `source_id` | Flynax ID if applicable. |
| `old_path` | Original path. |
| `new_url` | WordPress target URL. |
| `http_status` | Usually 301. |
| `confidence` | exact, reconstructed, manual_review. |
| `status` | active, disabled, conflict, needs_review. |
| `created_at` / `updated_at` | Audit timestamps. |

Decision:
- Preserve exact URLs when practical. Where Flynax URL generation cannot be fully reconstructed, create a confidence-marked redirect candidate and validate with crawl/source data before cutover.

## Migration Traceability

Custom table: `wp_servicesbg_migration_map`

| Column | Purpose |
| --- | --- |
| `id` | Mapping ID. |
| `batch_id` | Migration batch ID. |
| `source_table` | Flynax table. |
| `source_id` | Flynax row ID. |
| `target_type` | user, post, term, attachment, custom_table. |
| `target_table` | WordPress table or custom table. |
| `target_id` | Target row/object ID. |
| `checksum` | Source row checksum for idempotency. |
| `status` | imported, skipped, failed, deleted_by_rollback. |
| `notes` | Decision or error summary. |
| `created_at` / `updated_at` | Audit timestamps. |

Custom table: `wp_servicesbg_import_batches`

| Column | Purpose |
| --- | --- |
| `id` | Batch ID. |
| `name` | Human-readable batch name. |
| `source_db` | Source database. |
| `source_media_root` | Source media root. |
| `started_at` / `finished_at` | Timing. |
| `status` | running, completed, failed, rolled_back. |
| `summary_json` | Counts and validation summary. |

Decision:
- Every imported Flynax account, category, listing, media file, and optional message gets a mapping record.
- Rollback uses mapping records and batch IDs instead of broad destructive commands.

## Flynax Table Mapping

| Flynax table | Target object | Required action |
| --- | --- | --- |
| `fl_accounts` | WordPress users + provider profiles | Import identities, roles, contact fields, source IDs. |
| `fl_categories` | `service_category` taxonomy | Import hierarchy and term meta before listings. |
| `fl_listings` | `service_listing` CPT | Import title, description, owner, category, pricing, phone, partner, location, geodata. |
| `fl_config` | Migration reference only | Review for URL/media/site behavior; do not import blindly. |
| Flynax message tables, if present | conversations/messages | Confirm table names during source analysis; import if useful and legally/operationally appropriate. |
| Flynax booking tables/plugin data, if present | reservations/availability | Confirm schema; migrate only reliable historical or future reservation records. |
| Flynax media files | attachments + media map | Import from filesystem with checksum and owner mapping. |

## Open Data Questions Before Coding
- Confirm actual primary key names for `fl_accounts`, `fl_categories`, and `fl_listings`.
- Confirm all Flynax listing category relation fields.
- Confirm Flynax message table names and whether messages should be migrated.
- Confirm whether Flynax booking plugin contains meaningful reservation data.
- Confirm URL generation rules for listing/category/provider pages.
- Confirm source character encoding and Bulgarian slug handling.
- Confirm whether reviews/testimonials exist and should map to `Review`.

