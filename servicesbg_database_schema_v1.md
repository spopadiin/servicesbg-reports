# ServicesBG Database Schema v1

## Purpose
This document defines the ServicesBG WordPress CRM marketplace database model for the staged Flynax 4.5.2 migration.

It is a planning artifact only. No application code has been written. No production services.bg system is modified by this plan.

## Non-Negotiable Rules
- Preserve Flynax source IDs on every migrated object.
- Keep imports staged, idempotent, and reversible.
- Keep business logic inside plugins/modules, not WordPress core or the active theme.
- Use WordPress native tables where they are the correct ownership layer.
- Use custom tables for CRM, marketplace workflows, reservations, messages, claims, AI, analytics, redirects, and migration traceability.
- Treat MySQL foreign keys as logical relationships first; enforce consistency in importer validation and module services.
- Never cascade-delete migrated business data. Use statuses, soft deletion, rollback batches, and audit records.

## Naming and Storage Conventions
- WordPress table prefix is assumed to be `wp_` in examples. Actual deployments must use `$wpdb->prefix`.
- ServicesBG custom tables use `{$wpdb->prefix}servicesbg_*`.
- Primary keys use `BIGINT UNSIGNED AUTO_INCREMENT`.
- WordPress object references use `BIGINT UNSIGNED` and store the target WordPress ID.
- Source Flynax IDs use `BIGINT UNSIGNED` where numeric and retain the source table name in mapping tables.
- Timestamps use UTC `DATETIME` unless WordPress APIs require local display conversion.
- Structured flexible fields use JSON text columns named `*_json`; validation belongs to the owning module.
- Phone fields store both raw and normalized values.
- Source hashes/checksums are used for idempotent re-import and drift detection.

## WordPress Native Table Usage

| WordPress table | ServicesBG usage | Owner |
| --- | --- | --- |
| `wp_users` | Providers, freelancers, customers, admins, imported Flynax accounts when approved. | WordPress core; written through ServicesBG account/provider modules. |
| `wp_usermeta` | Source account ID, provider/customer flags, phone traceability, import batch, claim status hints. | WordPress core storage; write policy owned by `servicesbg-providers`. |
| `wp_posts` | `service_listing` CPT, media attachments, optional pages imported later. | WordPress core storage; listings owned by `servicesbg-listings`. |
| `wp_postmeta` | Listing source IDs, source status, phone, pricing, location, import state, claim state, media summary. | WordPress core storage; listings/meta policy owned by `servicesbg-listings`. |
| `wp_terms` | Service category terms. | WordPress core storage; taxonomy policy owned by `servicesbg-listings`. |
| `wp_term_taxonomy` | Hierarchical `service_category`. | WordPress core storage; taxonomy policy owned by `servicesbg-listings`. |
| `wp_term_relationships` | Listing-to-category assignments. | WordPress core storage; written by listings/import modules. |
| `wp_termmeta` | Flynax category IDs, original paths, source keys, import batch, redirect hints. | WordPress core storage; taxonomy policy owned by `servicesbg-listings`. |
| `wp_options` | Plugin schema versions, feature flags, migration locks, integration settings. | WordPress core storage; module-scoped option names. |
| `wp_comments` | Not used for core reviews in MVP. Reviews use custom tables for provider/listing/reservation traceability. | WordPress core only. |

## Custom Post Types

### `service_listing`
Represents an imported or native ServicesBG service listing.

Primary WordPress fields:
- `post_title`: imported `fl_listings.service_title`.
- `post_content`: imported `fl_listings.service_about`.
- `post_author`: mapped provider/freelancer user ID when reliable; otherwise import holding user or staging owner.
- `post_status`: public and non-public state mapped from Flynax status and validation.
- `post_date`: imported listing creation date where reliable.

Recommended statuses:
- `publish`: active and validation-clean.
- `draft`: incomplete or manually staged listing.
- `private`: retained historical listing not public.
- `servicesbg_archived`: expired/trash historical listing retained for traceability.
- `servicesbg_needs_review`: active listing blocked by validation, missing owner, missing phone, duplicate category ambiguity, or media/SEO risk.

Required listing post meta:

| Meta key | Purpose |
| --- | --- |
| `_servicesbg_source_listing_id` | Flynax `fl_listings.ID`. |
| `_servicesbg_source_account_id` | Flynax `fl_listings.Account_ID`. |
| `_servicesbg_source_category_id` | Flynax `fl_listings.Category_ID`. |
| `_servicesbg_import_batch_id` | Import batch that created or last updated the listing. |
| `_servicesbg_import_status` | Imported, skipped, failed, updated, rollback-protected. |
| `_servicesbg_flynax_status` | Original Flynax listing status. |
| `_servicesbg_claim_status` | Unclaimed, claimed, manual_review, transferred, blocked. |
| `_servicesbg_phone_raw` | Source phone as stored in Flynax. |
| `_servicesbg_phone_normalized` | Normalized claim/search phone. |
| `_servicesbg_phone_verified` | Boolean or timestamp marker for verified listing phone. |
| `_servicesbg_price` | Source/general price. |
| `_servicesbg_price_type` | Imported price type. |
| `_servicesbg_service_price_hour` | Hourly service price if present. |
| `_servicesbg_service_duration` | Service duration. |
| `_servicesbg_service_conditions` | Service conditions. |
| `_servicesbg_service_certificates` | Certificate text/reference. |
| `_servicesbg_service_provision_location` | Source service provision location. |
| `_servicesbg_partner_name` | Flynax partner name. |
| `_servicesbg_partner_website` | Flynax partner URL. |
| `_servicesbg_ref_number` | Source reference number. |
| `_servicesbg_location_json` | Country/region/city/address/geocode source snapshot. |
| `_servicesbg_original_url` | Best-known original Flynax URL/path. |
| `_servicesbg_redirect_target` | WordPress canonical URL used in redirect map. |

## Taxonomies

### `service_category`
Hierarchical taxonomy for migrated Flynax service categories.

Required term meta:

| Meta key | Purpose |
| --- | --- |
| `_servicesbg_source_category_id` | Flynax `fl_categories.ID`. |
| `_servicesbg_source_parent_id` | Flynax parent category ID. |
| `_servicesbg_import_batch_id` | Batch that imported the term. |
| `_servicesbg_flynax_key` | Source category key. |
| `_servicesbg_flynax_path` | Source category path. |
| `_servicesbg_flynax_tree` | Source hierarchy trace. |
| `_servicesbg_flynax_type` | Source category type, including manual-review values such as `seo_partners`. |
| `_servicesbg_original_url` | Old category URL candidate. |
| `_servicesbg_redirect_target` | New taxonomy archive URL. |

Duplicate Flynax category keys must be disambiguated deterministically, normally by parent path and source category ID.

### Phase 2 Optional Taxonomies
- `service_tag`: editorial/search tags.
- `service_area`: optional coarse public location facet if coverage search requires taxonomy-level browsing. Coverage itself remains custom-table owned by `servicesbg-coverage`.

## User Meta Strategy

Required user meta:

| Meta key | Purpose |
| --- | --- |
| `_servicesbg_source_account_id` | Flynax `fl_accounts.ID`. |
| `_servicesbg_source_account_type` | Flynax account type. |
| `_servicesbg_import_batch_id` | Batch that imported the user. |
| `_servicesbg_account_claim_status` | Imported, unclaimed, claimed, verified, manual_review. |
| `_servicesbg_phone_raw` | Source account phone. |
| `_servicesbg_phone_normalized` | Normalized phone. |
| `_servicesbg_phone_verified` | Verified phone marker. |
| `_servicesbg_quick_account_candidate` | Whether the source account appears temporary/quick-created. |
| `_servicesbg_zero_listing_account` | Source account had no listing rows. |

Passwords from Flynax are not migrated into WordPress authentication. Account access must be reset, claimed, or reverified.

## Custom Tables by Phase

### MVP Tables

#### `wp_servicesbg_import_batches`
Owner: `servicesbg-migration`

Purpose: Tracks each staged import, dry run, rollback test, and validation run.

Columns:
- `id`
- `batch_key`
- `source_db`
- `source_dump_path`
- `source_media_root`
- `mode`
- `status`
- `started_at`
- `finished_at`
- `dry_run`
- `created_by_user_id`
- `summary_json`
- `notes`

Indexes:
- Primary key `id`.
- Unique `batch_key`.
- `status, started_at`.
- `created_by_user_id`.

Relationships:
- Referenced by migration maps, media maps, redirect maps, WordPress meta, and validation reports.

#### `wp_servicesbg_migration_map`
Owner: `servicesbg-migration`

Purpose: Canonical source-to-target traceability and rollback control.

Columns:
- `id`
- `batch_id`
- `source_system`
- `source_table`
- `source_id`
- `source_parent_id`
- `source_hash`
- `target_type`
- `target_table`
- `target_id`
- `target_owner_plugin`
- `status`
- `rollback_action`
- `rollback_protected`
- `last_imported_at`
- `notes`

Indexes:
- Primary key `id`.
- Unique `source_system, source_table, source_id, target_type`.
- `batch_id, status`.
- `target_type, target_id`.
- `source_table, source_parent_id`.
- `rollback_protected, status`.

Relationships:
- `batch_id` logically references `wp_servicesbg_import_batches.id`.
- Target IDs reference WordPress or ServicesBG custom tables depending on `target_type`.

#### `wp_servicesbg_audit_log`
Owner: `servicesbg-core`

Purpose: Cross-module audit trail for admin decisions, claims, rollbacks, imports, reservation changes, and sensitive workflow events.

Columns:
- `id`
- `actor_user_id`
- `module`
- `action`
- `object_type`
- `object_id`
- `source_table`
- `source_id`
- `ip_hash`
- `user_agent_hash`
- `payload_json`
- `created_at`

Indexes:
- Primary key `id`.
- `module, action, created_at`.
- `object_type, object_id`.
- `actor_user_id, created_at`.
- `source_table, source_id`.

#### `wp_servicesbg_provider_profiles`
Owner: `servicesbg-providers`

Purpose: Provider/freelancer profile data that should not be scattered across user meta.

Columns:
- `id`
- `user_id`
- `source_account_id`
- `profile_type`
- `display_name`
- `company_name`
- `freelancer_name`
- `status`
- `verification_status`
- `phone_raw`
- `phone_normalized`
- `phone_verified_at`
- `email_snapshot`
- `website`
- `about`
- `address`
- `country`
- `region`
- `city`
- `postal_code`
- `latitude`
- `longitude`
- `source_location_json`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- Unique `user_id`.
- Unique `source_account_id`.
- `profile_type, status`.
- `verification_status, status`.
- `phone_normalized`.
- `country, region, city`.
- `latitude, longitude`.

Relationships:
- `user_id` logically references `wp_users.ID`.
- `source_account_id` references Flynax `fl_accounts.ID`.

#### `wp_servicesbg_claims`
Owner: `servicesbg-claims`

Purpose: Listing ownership claim workflow, including SMS OTP state.

Columns:
- `id`
- `listing_id`
- `source_listing_id`
- `source_account_id`
- `claimant_user_id`
- `current_owner_user_id`
- `target_owner_user_id`
- `phone_raw`
- `phone_normalized`
- `otp_hash`
- `otp_expires_at`
- `otp_attempt_count`
- `otp_send_count`
- `sms_provider`
- `sms_message_id`
- `status`
- `verified_at`
- `transferred_at`
- `reviewed_by_user_id`
- `decision_notes`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- `listing_id, status`.
- `source_listing_id`.
- `claimant_user_id, status`.
- `phone_normalized, status`.
- `otp_expires_at, status`.
- `sms_provider, sms_message_id`.

Relationships:
- `listing_id` references `wp_posts.ID` for `service_listing`.
- `claimant_user_id`, `current_owner_user_id`, `target_owner_user_id`, `reviewed_by_user_id` reference `wp_users.ID`.
- Source IDs preserve `fl_listings.ID` and `fl_accounts.ID`.

#### `wp_servicesbg_availability_rules`
Owner: `servicesbg-reservations`

Purpose: Provider/listing recurring availability and capacity.

Columns:
- `id`
- `provider_user_id`
- `listing_id`
- `weekday`
- `start_time`
- `end_time`
- `timezone`
- `capacity`
- `effective_from`
- `effective_to`
- `status`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- `provider_user_id, weekday, status`.
- `listing_id, weekday, status`.
- `effective_from, effective_to`.

Relationships:
- `provider_user_id` references `wp_users.ID`.
- `listing_id` references `service_listing` post ID and may be null for provider-wide rules.

#### `wp_servicesbg_capacity_exceptions`
Owner: `servicesbg-reservations`

Purpose: One-off closures, overrides, holidays, and temporary capacity changes.

Columns:
- `id`
- `provider_user_id`
- `listing_id`
- `exception_date`
- `start_time`
- `end_time`
- `timezone`
- `capacity_override`
- `capacity_delta`
- `reason`
- `status`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- `provider_user_id, exception_date, status`.
- `listing_id, exception_date, status`.

#### `wp_servicesbg_reservations`
Owner: `servicesbg-reservations`

Purpose: Reservation/request records tied to listings, providers, leads, conversations, and future monetization.

Columns:
- `id`
- `listing_id`
- `provider_user_id`
- `customer_user_id`
- `lead_id`
- `conversation_id`
- `start_at`
- `end_at`
- `timezone`
- `capacity_units`
- `status`
- `source`
- `customer_name`
- `customer_phone`
- `customer_email`
- `service_location`
- `notes`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- `provider_user_id, start_at, end_at, status`.
- `listing_id, start_at, end_at, status`.
- `customer_user_id, start_at`.
- `lead_id`.
- `conversation_id`.
- `status, created_at`.

#### `wp_servicesbg_conversations`
Owner: `servicesbg-messaging`

Purpose: Thread-level messaging state for listing inquiries, provider/customer chat, and AI-assisted flows.

Columns:
- `id`
- `listing_id`
- `provider_user_id`
- `customer_user_id`
- `lead_id`
- `reservation_id`
- `channel`
- `status`
- `last_message_at`
- `source_thread_id`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- `provider_user_id, status, last_message_at`.
- `customer_user_id, status, last_message_at`.
- `listing_id, status`.
- `lead_id`.
- `reservation_id`.
- `source_thread_id`.

#### `wp_servicesbg_messages`
Owner: `servicesbg-messaging`

Purpose: Message rows. Imported Flynax messages are conditional on privacy/business approval.

Columns:
- `id`
- `conversation_id`
- `sender_type`
- `sender_user_id`
- `body`
- `visibility`
- `ai_generated`
- `ai_model`
- `source_message_id`
- `source_payload_json`
- `attachment_id`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- `conversation_id, created_at`.
- `sender_user_id, created_at`.
- `source_message_id`.
- `ai_generated, created_at`.

Relationships:
- `conversation_id` references `wp_servicesbg_conversations.id`.
- `source_message_id` preserves `fl_messages.ID` where imported.

#### `wp_servicesbg_leads`
Owner: `servicesbg-leads-crm`

Purpose: CRM lead/inquiry records from forms, messaging, reservations, contact imports, and future channels.

Columns:
- `id`
- `listing_id`
- `provider_user_id`
- `customer_user_id`
- `conversation_id`
- `reservation_id`
- `source`
- `status`
- `customer_name`
- `customer_phone`
- `customer_email`
- `service_location`
- `requested_at`
- `value_estimate`
- `conversion_score`
- `source_contact_id`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- `provider_user_id, status, requested_at`.
- `listing_id, status, requested_at`.
- `customer_user_id, requested_at`.
- `customer_phone`.
- `customer_email`.
- `conversation_id`.
- `reservation_id`.
- `source_contact_id`.

Relationships:
- `source_contact_id` preserves `fl_contacts.ID` if imported.

#### `wp_servicesbg_coverage_zones`
Owner: `servicesbg-coverage`

Purpose: Provider/listing service area definitions and imported location-derived suggestions.

Columns:
- `id`
- `provider_user_id`
- `listing_id`
- `zone_type`
- `label`
- `country`
- `region`
- `city`
- `center_latitude`
- `center_longitude`
- `radius_km`
- `polygon_json`
- `travel_fee`
- `min_order_value`
- `priority`
- `status`
- `source_account_id`
- `source_listing_id`
- `source_location_json`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- `provider_user_id, status, priority`.
- `listing_id, status, priority`.
- `country, region, city`.
- `center_latitude, center_longitude`.
- `source_account_id`.
- `source_listing_id`.

Future optional index:
- Spatial index on generated geometry columns if the production MySQL version and hosting support it.

#### `wp_servicesbg_media_map`
Owner: `servicesbg-migration`; attachments owned by WordPress media.

Purpose: Media source-to-attachment traceability and failed media import tracking.

Columns:
- `id`
- `import_batch_id`
- `attachment_id`
- `owner_type`
- `owner_id`
- `source_path`
- `source_checksum`
- `source_photo_id`
- `source_listing_id`
- `source_account_id`
- `media_type`
- `status`
- `failure_reason`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- Unique `source_path`.
- `attachment_id`.
- `owner_type, owner_id`.
- `source_photo_id`.
- `source_listing_id`.
- `import_batch_id, status`.

Relationships:
- `source_photo_id` preserves `fl_listing_photos.ID`.

#### `wp_servicesbg_redirect_map`
Owner: `servicesbg-seo-migration`

Purpose: Old Flynax URL to new WordPress URL mapping, including confidence and validation state.

Columns:
- `id`
- `import_batch_id`
- `source_type`
- `source_table`
- `source_id`
- `old_path`
- `old_url_hash`
- `new_url`
- `http_status`
- `confidence`
- `status`
- `hit_count`
- `last_hit_at`
- `created_at`
- `updated_at`

Indexes:
- Primary key `id`.
- Unique `old_url_hash`.
- `source_type, source_id`.
- `status, confidence`.
- `import_batch_id, status`.

#### `wp_servicesbg_search_index`
Owner: `servicesbg-search`

Purpose: Dedicated denormalized search records for service listings, providers, categories, coverage, claim state, and ranking signals. Search is not hidden inside listings or coverage.

Columns:
- `id`
- `object_type`
- `object_id`
- `listing_id`
- `provider_user_id`
- `category_term_id`
- `title`
- `summary`
- `keywords`
- `phone_normalized`
- `country`
- `region`
- `city`
- `latitude`
- `longitude`
- `coverage_status`
- `claim_status`
- `listing_status`
- `rank_score`
- `source_listing_id`
- `source_account_id`
- `indexed_at`
- `index_hash`

Indexes:
- Primary key `id`.
- Unique `object_type, object_id`.
- `listing_id`.
- `provider_user_id`.
- `category_term_id`.
- `country, region, city`.
- `listing_status, claim_status`.
- `rank_score`.
- `source_listing_id`.
- `source_account_id`.
- Fulltext `title, summary, keywords` where supported.

Relationships:
- References WordPress listings, users, and taxonomy terms logically.
- Rebuilt from source-of-truth modules. It is not the authority for listing or coverage data.

#### `wp_servicesbg_search_queries`
Owner: `servicesbg-search`

Purpose: Search analytics and quality feedback for MVP tuning without depending on generic analytics.

Columns:
- `id`
- `query_text`
- `normalized_query`
- `filters_json`
- `result_count`
- `selected_object_type`
- `selected_object_id`
- `user_id`
- `session_hash`
- `created_at`

Indexes:
- Primary key `id`.
- `normalized_query, created_at`.
- `result_count`.
- `selected_object_type, selected_object_id`.
- `user_id, created_at`.

### Phase 2 Tables

#### `wp_servicesbg_reviews`
Owner: `servicesbg-reviews`

Purpose: Listing/provider reviews with reservation and lead traceability.

Key columns:
- `id`, `listing_id`, `provider_user_id`, `customer_user_id`, `reservation_id`, `lead_id`, `rating`, `title`, `body`, `status`, `source_review_id`, `created_at`, `updated_at`.

Indexes:
- `provider_user_id, status, created_at`.
- `listing_id, status, created_at`.
- `reservation_id`.
- `customer_user_id, created_at`.

#### `wp_servicesbg_lead_events`
Owner: `servicesbg-leads-crm`

Purpose: Timeline events for lead lifecycle, assignment, follow-up, conversion, and future pricing signals.

Key columns:
- `id`, `lead_id`, `event_type`, `actor_user_id`, `payload_json`, `created_at`.

Indexes:
- `lead_id, created_at`.
- `event_type, created_at`.
- `actor_user_id, created_at`.

#### `wp_servicesbg_provider_verifications`
Owner: `servicesbg-providers`

Purpose: Provider verification workflow beyond basic SMS claims.

Key columns:
- `id`, `provider_user_id`, `verification_type`, `status`, `evidence_attachment_id`, `reviewed_by_user_id`, `reviewed_at`, `notes`, `created_at`, `updated_at`.

Indexes:
- `provider_user_id, status`.
- `verification_type, status`.

#### `wp_servicesbg_ai_knowledge_sources`
Owner: `servicesbg-ai`

Purpose: Approved AI knowledge sources by global, category, provider, or listing scope.

Key columns:
- `id`, `scope_type`, `scope_id`, `owner_user_id`, `source_type`, `source_url`, `source_attachment_id`, `title`, `approval_status`, `source_hash`, `created_at`, `updated_at`.

Indexes:
- `scope_type, scope_id, approval_status`.
- `owner_user_id`.
- `source_hash`.

#### `wp_servicesbg_ai_knowledge_chunks`
Owner: `servicesbg-ai`

Purpose: Chunked searchable/retrievable AI knowledge. Embeddings may live in an external vector store; this table owns traceability.

Key columns:
- `id`, `source_id`, `chunk_index`, `chunk_text`, `token_count`, `embedding_ref`, `status`, `created_at`, `updated_at`.

Indexes:
- Unique `source_id, chunk_index`.
- `embedding_ref`.
- `status`.

#### `wp_servicesbg_ai_training_events`
Owner: `servicesbg-ai`

Purpose: Explicit learning/training events for provider, category, and platform layers. Imported private messages must not be used here unless separately approved and anonymized.

Key columns:
- `id`, `scope_type`, `scope_id`, `event_type`, `source_object_type`, `source_object_id`, `actor_user_id`, `approved_by_user_id`, `training_payload_json`, `status`, `created_at`.

Indexes:
- `scope_type, scope_id, created_at`.
- `source_object_type, source_object_id`.
- `event_type, status`.
- `approved_by_user_id, created_at`.

#### `wp_servicesbg_ai_feedback`
Owner: `servicesbg-ai`

Purpose: Provider/customer/admin feedback on AI answers and retrieval quality.

Key columns:
- `id`, `conversation_id`, `message_id`, `provider_user_id`, `rating`, `feedback`, `created_at`.

Indexes:
- `conversation_id`.
- `message_id`.
- `provider_user_id, created_at`.

#### `wp_servicesbg_notifications`
Owner: `servicesbg-integrations`

Purpose: Outbound notification queue/audit for SMS, email, and future channels.

Key columns:
- `id`, `user_id`, `channel`, `event_type`, `provider`, `external_message_id`, `payload_json`, `status`, `sent_at`, `created_at`, `updated_at`.

Indexes:
- `user_id, created_at`.
- `channel, status`.
- `provider, external_message_id`.

### Phase 3 Tables

#### `wp_servicesbg_analytics_events`
Owner: `servicesbg-analytics`

Purpose: Marketplace analytics events for funnels, reservations, inquiry intensity, and monetization planning.

Key columns:
- `id`, `event_type`, `actor_user_id`, `listing_id`, `provider_user_id`, `lead_id`, `reservation_id`, `source`, `payload_json`, `occurred_at`.

Indexes:
- `event_type, occurred_at`.
- `provider_user_id, occurred_at`.
- `listing_id, occurred_at`.
- `lead_id`.
- `reservation_id`.

#### `wp_servicesbg_pricing_signals`
Owner: `servicesbg-analytics`

Purpose: Aggregated signals for future dynamic conversion/pricing model.

Key columns:
- `id`, `provider_user_id`, `listing_id`, `period_start`, `period_end`, `inquiry_count`, `reservation_count`, `response_time_avg`, `conversion_rate`, `score`, `created_at`.

Indexes:
- `provider_user_id, period_start, period_end`.
- `listing_id, period_start, period_end`.

#### `wp_servicesbg_coverage_discoveries`
Owner: `servicesbg-coverage`

Purpose: Suggested coverage improvements from leads, reservations, searches, and imported geodata.

Key columns:
- `id`, `provider_user_id`, `listing_id`, `suggested_zone_json`, `evidence_json`, `status`, `created_at`, `updated_at`.

Indexes:
- `provider_user_id, status`.
- `listing_id, status`.

#### `wp_servicesbg_external_integration_events`
Owner: `servicesbg-integrations`

Purpose: Durable audit of external API events, webhooks, SMS delivery receipts, map/geocode calls, email events, and LLM provider calls.

Key columns:
- `id`, `integration`, `event_type`, `external_id`, `object_type`, `object_id`, `request_hash`, `response_status`, `payload_json`, `created_at`.

Indexes:
- `integration, event_type, created_at`.
- `external_id`.
- `object_type, object_id`.
- `response_status, created_at`.

#### `wp_servicesbg_payment_audit`
Owner: `servicesbg-monetization`

Purpose: Historical Flynax plan/payment references and future marketplace monetization audit. Not part of MVP billing.

Key columns:
- `id`, `source_table`, `source_id`, `source_account_id`, `provider_user_id`, `listing_id`, `amount`, `currency`, `gateway`, `status`, `paid_at`, `payload_json`, `import_batch_id`, `created_at`.

Indexes:
- `source_table, source_id`.
- `provider_user_id, paid_at`.
- `listing_id, paid_at`.
- `status, paid_at`.

## Foreign-Key-Like Relationship Strategy
Physical MySQL foreign keys are optional and should not be required in MVP because WordPress hosting environments, plugin activation order, and rollback behavior are easier to manage with logical constraints.

Validation must enforce:
- Every custom-table `listing_id` points to a `service_listing` post unless it is explicitly nullable.
- Every `provider_user_id`, `customer_user_id`, and actor user reference points to `wp_users.ID` unless anonymous/contact snapshot is allowed.
- Every migrated object has a `wp_servicesbg_migration_map` row.
- Every migrated listing retains source listing, account, and category IDs.
- Every redirect row references a known source object or is marked `manual_review`.
- Every media map row either has an attachment ID or a failure status/reason.

Delete policy:
- Do not hard-delete imported data during normal operations.
- Use statuses for archival, trash, blocked, superseded, and rollback-protected states.
- Rollback tools may delete rows only for an import batch that has no protected/manual modifications.

## Batch Rollback Strategy
Each import batch must write:
- A row in `wp_servicesbg_import_batches`.
- One or more `wp_servicesbg_migration_map` rows per source object.
- Source IDs in native WordPress meta or custom-table source fields.
- Audit events for destructive rollback or manual override actions.

Rollback rules:
- Roll back in reverse dependency order: search indexes, redirects, media maps/attachments, CRM records, messages, reservations, claims, listings, provider profiles/users, categories.
- Never delete a WordPress user, listing, or profile marked `rollback_protected`.
- Mark claimed, manually edited, reserved, reviewed, or AI-trained objects as protected by default.
- For skipped source objects, keep a migration map row with `status=skipped` and reason.
- For failed source objects, keep a migration map row with `status=failed` and validation details.
- Re-running a batch must compare `source_hash` and update only idempotent targets.

## Flynax Source ID Preservation
Required preservation points:
- `fl_accounts.ID`: user meta, provider profile `source_account_id`, migration map.
- `fl_categories.ID`: term meta, migration map, listing source category meta.
- `fl_listings.ID`: post meta, custom domain source fields, migration map.
- `fl_listing_photos.ID`: media map `source_photo_id`.
- `fl_messages.ID`: messages `source_message_id` if imported.
- `fl_contacts.ID`: leads `source_contact_id` if imported.
- Booking/payment/plugin table IDs: payment audit or migration map if imported as reference.

## MVP/Phase Summary

| Phase | Schema included |
| --- | --- |
| MVP | Native WP users/posts/terms/media; `service_listing`; `service_category`; import batches; migration map; audit log; provider profiles; claims/SMS OTP; availability; capacity exceptions; reservations; conversations; messages; leads; coverage zones; media map; redirect map; dedicated search index/query tables. |
| Phase 2 | Reviews; lead events; provider verification; AI knowledge sources/chunks/training events/feedback; notifications. |
| Phase 3 | Analytics events; pricing signals; coverage discoveries; external integration events; payment audit/future monetization. |

