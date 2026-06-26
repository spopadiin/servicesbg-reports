# ServicesBG Platform Event Catalog v1

## Purpose
Defines the planned ServicesBG platform event vocabulary for existing plugins to adopt after Phase 2H-B.

No plugin refactor or new business feature is implemented by this document. Every functional area remains a WordPress plugin. Cross-plugin communication should go through `servicesbg-platform` event and service-contract APIs rather than direct plugin-to-plugin includes.

## Naming Rules
- Event names use lowercase dot notation in documentation, for example `reservation.requested`.
- The Phase 2H event bus stores normalized names internally, for example `reservation_requested`.
- Payloads must contain IDs and state only. Do not include private raw review bodies, OTP values, secrets, API keys, or full customer message text unless the event owner explicitly documents that field.
- Every event should include `producer_module`, `object_type`, `object_id`, and `correlation_id` context when published.

## Core Audit Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `audit.recorded` | `servicesbg-core` | platform diagnostics, analytics later | `module`, `action`, `object_type`, `object_id`, `source_table`, `source_id` | Announce a durable audit record without forcing modules to query audit tables directly. |
| `audit.sensitive_action_recorded` | `servicesbg-core` | admin, security/reporting later | `module`, `action`, `actor_user_id`, `object_type`, `object_id` | Highlight claim, ownership, moderation, or privacy-sensitive operations. |

## Claims Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `claim.requested` | `servicesbg-claims` | CRM, messaging, audit, search | `claim_id`, `listing_id`, `source_listing_id`, `claimant_user_id`, `status` | A listing ownership claim was requested. |
| `claim.otp_pending` | `servicesbg-claims` | audit, admin | `claim_id`, `listing_id`, `status`, `sms_provider` | Claim is waiting for verification. No OTP value is emitted. |
| `claim.verified` | `servicesbg-claims` | CRM, reservations, messaging, search | `claim_id`, `listing_id`, `claimant_user_id`, `status` | Claim phone verification succeeded. |
| `claim.approved` | `servicesbg-claims` | listings, search, CRM, messaging | `claim_id`, `listing_id`, `target_owner_user_id`, `status` | Admin approved the claim. |
| `claim.rejected` | `servicesbg-claims` | CRM, messaging, audit | `claim_id`, `listing_id`, `reviewed_by_user_id`, `status` | Admin rejected the claim. |
| `claim.transferred` | `servicesbg-claims` | listings, search, reservations, CRM | `claim_id`, `listing_id`, `previous_owner_user_id`, `target_owner_user_id`, `status` | Ownership transfer completed. |

## Coverage Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `coverage.zone_created` | `servicesbg-coverage` | search, CRM, analytics later | `zone_id`, `provider_user_id`, `listing_id`, `status`, `country`, `region`, `city` | A coverage zone was created. |
| `coverage.zone_updated` | `servicesbg-coverage` | search, CRM | `zone_id`, `provider_user_id`, `listing_id`, `status` | A coverage zone changed. |
| `coverage.zone_disabled` | `servicesbg-coverage` | search, CRM | `zone_id`, `provider_user_id`, `listing_id`, `status` | A coverage zone was deactivated/archived. |
| `coverage.discovery_suggested` | `servicesbg-coverage` | CRM, admin, search later | `discovery_id`, `provider_user_id`, `listing_id`, `source`, `confidence` | Future inferred coverage suggestion. |

## Search Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `search.index_refresh_requested` | any owner module | `servicesbg-search` | `object_type`, `object_id`, `listing_id`, `provider_user_id`, `reason` | Request a search index refresh without directly calling search internals. |
| `search.index_updated` | `servicesbg-search` | platform, analytics later | `object_type`, `object_id`, `listing_id`, `provider_user_id`, `index_hash` | Search index row was refreshed. |
| `search.index_failed` | `servicesbg-search` | admin, audit | `object_type`, `object_id`, `error` | Search indexing failed. |
| `search.query_recorded` | `servicesbg-search` | CRM/analytics later | `query_id`, `query`, `filters_json`, `result_count` | A marketplace search query was logged. |

## Reservation Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `reservation.requested` | `servicesbg-reservations` | CRM, messaging, search, reviews later | `reservation_id`, `listing_id`, `provider_user_id`, `customer_user_id`, `lead_id`, `conversation_id`, `status` | Reservation request created. |
| `reservation.provider_accepted` | `servicesbg-reservations` | CRM, messaging, reviews later | `reservation_id`, `provider_user_id`, `status` | Provider accepted a request. |
| `reservation.provider_rejected` | `servicesbg-reservations` | CRM, messaging | `reservation_id`, `provider_user_id`, `status` | Provider rejected a request. |
| `reservation.customer_cancelled` | `servicesbg-reservations` | CRM, messaging | `reservation_id`, `customer_user_id`, `status` | Customer cancelled. |
| `reservation.completed` | `servicesbg-reservations` | reviews, AI reviews, CRM, search | `reservation_id`, `listing_id`, `provider_user_id`, `customer_user_id`, `status` | Future completion signal for reviews/trust. |
| `availability.rule_created` | `servicesbg-reservations` | search, CRM | `rule_id`, `provider_user_id`, `listing_id`, `status` | Availability changed and search may need refresh. |
| `availability.exception_created` | `servicesbg-reservations` | search, CRM | `exception_id`, `provider_user_id`, `listing_id`, `status` | One-off capacity/closure change. |

## Review Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `review.created` | `servicesbg-reviews` | AI reviews, CRM, messaging | `review_id`, `listing_id`, `provider_user_id`, `reviewer_user_id`, `reservation_id`, `lead_id`, `status`, `privacy_status` | Private raw review created. No raw body text. |
| `review.provider_response_added` | `servicesbg-reviews` | AI reviews, CRM | `review_id`, `provider_user_id`, `status` | Provider responded privately. No response body. |
| `review.moderated` | `servicesbg-reviews` | AI reviews, search, CRM | `review_id`, `listing_id`, `provider_user_id`, `status`, `privacy_status` | Admin moderation changed review state. |
| `review_case.opened` | `servicesbg-reviews` | messaging, CRM, admin | `case_id`, `review_id`, `listing_id`, `provider_user_id`, `reviewer_user_id`, `status` | Private review case opened. |
| `review_case.event_added` | `servicesbg-reviews` | CRM, admin | `event_id`, `case_id`, `review_id`, `event_type`, `visibility` | Private case timeline event. Body is not emitted. |
| `review_case.resolved` | `servicesbg-reviews` | AI reviews, CRM, search | `case_id`, `review_id`, `resolution_outcome`, `status` | Case was resolved. |

## AI Review Summary Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `review_summary.job_enqueued` | `servicesbg-ai-reviews` | platform jobs, admin | `job_id`, `scope`, `provider_user_id`, `listing_id`, `status` | Summary generation job queued. |
| `review_summary.generated` | `servicesbg-ai-reviews` | admin, search later | `summary_version_id`, `scope`, `provider_user_id`, `listing_id`, `status`, `ai_provider`, `ai_model` | Deterministic summary candidate created. |
| `review_summary.approved` | `servicesbg-ai-reviews` | search, CRM | `summary_version_id`, `approved_by_user_id`, `status` | Summary candidate approved. |
| `review_summary.published` | `servicesbg-ai-reviews` | search | `summary_version_id`, `scope`, `provider_user_id`, `listing_id`, `status` | Public summary version published; search refresh should be requested. |
| `review_summary.rejected` | `servicesbg-ai-reviews` | admin | `summary_version_id`, `rejected_by_user_id`, `status` | Summary candidate rejected. |

## Messaging Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `messaging.conversation_created` | `servicesbg-messaging` | CRM, reservations, reviews/claims where linked | `conversation_id`, `conversation_type`, `listing_id`, `provider_user_id`, `customer_user_id`, `lead_id`, `reservation_id`, `review_case_id`, `claim_id`, `status` | Conversation created. |
| `messaging.participant_added` | `servicesbg-messaging` | CRM, admin | `participant_id`, `conversation_id`, `user_id`, `role`, `status` | Participant added. |
| `messaging.message_sent` | `servicesbg-messaging` | CRM, search/analytics later | `message_id`, `conversation_id`, `sender_user_id`, `sender_type`, `visibility`, `status` | Message row created. Body is not emitted by default. |
| `messaging.conversation_read` | `servicesbg-messaging` | CRM | `conversation_id`, `user_id`, `message_id` | Participant read marker updated. |
| `messaging.conversation_status_updated` | `servicesbg-messaging` | CRM, admin | `conversation_id`, `status` | Conversation status changed. |

## CRM Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `crm.lead_created` | `servicesbg-crm` | messaging, reservations, search/analytics later | `lead_id`, `customer_profile_id`, `listing_id`, `provider_user_id`, `conversation_id`, `reservation_id`, `review_case_id`, `claim_id`, `source`, `status` | Lead/inquiry created. |
| `crm.customer_profile_created` | `servicesbg-crm` | messaging, reservations | `customer_profile_id`, `user_id`, `source_account_id` | Customer profile created. |
| `crm.timeline_event_created` | `servicesbg-crm` | admin/analytics later | `timeline_event_id`, `lead_id`, `customer_profile_id`, `event_type` | Customer timeline event created. |
| `crm.note_created` | `servicesbg-crm` | admin only | `note_id`, `lead_id`, `customer_profile_id`, `visibility` | Internal note created. Body is not emitted by default. |
| `crm.followup_created` | `servicesbg-crm` | admin, platform jobs later | `followup_id`, `lead_id`, `customer_profile_id`, `assigned_to_user_id`, `due_at`, `status` | Follow-up task created. |
| `crm.pipeline_updated` | `servicesbg-crm` | search/analytics later | `pipeline_item_id`, `lead_id`, `customer_profile_id`, `stage`, `status` | Pipeline stage/status changed. |

## Workflow/Job Events

| Event | Producer | Typical listeners | Payload keys | Purpose |
| --- | --- | --- | --- | --- |
| `platform.event_published` | `servicesbg-platform` | admin, audit, diagnostics | `event_id`, `event_name`, `producer_module`, `object_type`, `object_id`, `listener_count` | Platform event bus accepted and stored an event row. |
| `platform.job_enqueued` | `servicesbg-platform` | admin, audit | `job_id`, `job_type`, `handler_key`, `producer_module`, `status` | Platform job queued. |
| `platform.job_started` | `servicesbg-platform` | admin | `job_id`, `job_type`, `handler_key`, `attempts` | Future explicit job-start event. |
| `platform.job_completed` | `servicesbg-platform` | admin, audit | `job_id`, `job_type`, `handler_key`, `status` | Platform job completed. |
| `platform.job_failed` | `servicesbg-platform` | admin, audit | `job_id`, `job_type`, `handler_key`, `status`, `error_code` | Platform job failed. |

## Initial Job Types

| Job type | Handler owner | Payload keys | Purpose |
| --- | --- | --- | --- |
| `search.index_refresh` | `servicesbg-search` | `object_type`, `object_id`, `listing_id`, `provider_user_id`, `reason` | Refresh one search index document or related documents. |
| `review_summary.generate` | `servicesbg-ai-reviews` | `scope`, `provider_user_id`, `listing_id`, `source_event_id` | Generate deterministic review summary candidate. |
| `crm.followup_due_check` | `servicesbg-crm` | `followup_id`, `lead_id`, `customer_profile_id` | Future internal due-check primitive, not notifications. |
| `platform.validation` | `servicesbg-platform` | `validation`, `source` | Existing validation-only handler. |

## Privacy and Compatibility Notes
- Raw review text, message body text, OTP values, and private note bodies stay in owner plugin tables.
- Search consumes only approved public summaries and aggregate trust fields.
- Existing WordPress actions/filters may remain during transition, but new cross-module behavior should publish platform events in parallel first.
- Event consumers must treat payloads as immutable facts and call owner service contracts for richer read models.
