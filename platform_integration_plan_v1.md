# ServicesBG Platform Integration Plan v1

## Purpose
Defines how existing ServicesBG plugins will adopt `servicesbg-platform` events and contracts after Phase 2H.

This is a planning-only phase. It does not implement new business features, does not refactor existing plugins, and does not modify production.

## Architecture Rules
- Every ServicesBG functional area remains a WordPress plugin.
- Cross-plugin communication goes through `servicesbg-platform` events and service contracts.
- No direct plugin-to-plugin includes.
- No business logic in the theme.
- Existing plugins continue working during migration.
- Refactors should be incremental and reversible.

## Integration Strategy
1. Add platform publish calls next to existing audit writes, without changing current database writes.
2. Register read/service contracts through the platform service registry, one plugin at a time.
3. Add listeners that request follow-up work through platform jobs rather than directly calling another plugin's internal classes.
4. Keep existing WordPress actions/filters temporarily for backwards compatibility.
5. Move validation scripts to assert both old behavior and new platform event/job rows.
6. Remove legacy direct coupling only after all downstream listeners and validators pass.

## Shared Contract Pattern
Each owner plugin should expose a small public contract object through:
- `servicesbg_platform_register_service('module.contract', $contract)`

Contract methods should be read-oriented first:
- `getById(int $id): ?array`
- `status(int $id): string`
- `linkage(int $id): array`
- `validatePayload(array $payload): bool|\WP_Error`

Write methods should stay in the owning module's service layer and be called only by that owner module unless a future explicit command contract is approved.

## Plugin Plans

### `servicesbg-claims`

Emitted events:
- `claim.requested`
- `claim.otp_pending`
- `claim.verified`
- `claim.approved`
- `claim.rejected`
- `claim.transferred`
- `audit.sensitive_action_recorded`

Listened events:
- `messaging.conversation_created` where `claim_id` is present, to annotate claim history later.
- `crm.lead_created` where `claim_id` is present, to expose claim/customer context in admin later.
- `search.index_updated` for listing claim visibility diagnostics later.

Service contracts needed:
- `claims.claim_reader`: read claim by ID, listing ID, claimant, status.
- `claims.claim_status_reader`: expose normalized claim status for search/listing consumers.

Migration/refactor risk:
- Medium. Claim state transitions are sensitive because they can affect listing ownership. Event publication must be post-commit only and must never emit OTP values.

Implementation order:
1. Publish events after existing claim audit writes.
2. Register read-only claim contracts.
3. Add validation for emitted events.
4. Add search refresh job enqueue on approved/transferred events.

Backwards compatibility:
- Keep existing claim service methods and audit entries unchanged.
- Keep existing SMS null adapter behavior unchanged.
- Platform events are additive.

Validation plan:
- Extend claims validation to assert event rows for request/verify/review.
- Assert no OTP value appears in event payload JSON.
- Assert `search.index_refresh` job is queued only after approved/transferred states once that listener is added.

### `servicesbg-coverage`

Emitted events:
- `coverage.zone_created`
- `coverage.zone_updated`
- `coverage.zone_disabled`
- `coverage.discovery_suggested` later

Listened events:
- `claim.transferred` to refresh provider/listing coverage diagnostics if ownership changes.
- `reservation.requested` and `crm.lead_created` later as coverage discovery inputs.

Service contracts needed:
- `coverage.zone_reader`: read active/suggested zones by listing or provider.
- `coverage.coverage_summary`: return normalized country/region/city/status for search.

Migration/refactor risk:
- Low to medium. Current search reads coverage tables directly; first transition should add a contract while retaining existing SQL until search listener migration is validated.

Implementation order:
1. Publish zone create/update/disable events.
2. Register coverage summary contract.
3. Make search listen to coverage events and enqueue `search.index_refresh`.
4. Later move search coverage enrichment from direct SQL to the contract.

Backwards compatibility:
- Keep `wp_servicesbg_coverage_zones` as the source of truth.
- Keep search's current direct coverage read until contract output matches.

Validation plan:
- Extend coverage validation to assert event rows.
- Add search validation case where coverage zone creation queues an index refresh job.
- Compare contract summary against existing coverage table result.

### `servicesbg-search`

Emitted events:
- `search.index_updated`
- `search.index_failed`
- `search.query_recorded`

Listened events:
- `search.index_refresh_requested`
- `listing.created`, `listing.updated` when listing events exist.
- `claim.approved`, `claim.transferred`
- `coverage.zone_created`, `coverage.zone_updated`, `coverage.zone_disabled`
- `reservation.completed`
- `review.moderated`
- `review_summary.published`
- `crm.pipeline_updated` later for ranking signals.

Service contracts needed:
- `search.indexer`: enqueue or execute index refresh by object type/ID.
- `search.query_reader`: read query telemetry for analytics/CRM later.

Migration/refactor risk:
- Medium. Search spans many data owners and currently uses filters/actions. The first platform integration should request jobs, not synchronous reindexing.

Implementation order:
1. Add listener for `search.index_refresh_requested`.
2. Add platform job handler for `search.index_refresh`.
3. Publish `search.index_updated` after successful upsert.
4. Switch upstream plugins to request index refresh events.

Backwards compatibility:
- Keep existing WP-CLI rebuild and direct indexer methods.
- Keep existing WordPress filters while platform event listeners are introduced.

Validation plan:
- Assert refresh request creates a platform job.
- Assert `run-next` updates index and publishes `search.index_updated`.
- Assert failed index operation publishes `search.index_failed` without fatal errors.

### `servicesbg-reservations`

Emitted events:
- `availability.rule_created`
- `availability.exception_created`
- `reservation.requested`
- `reservation.provider_accepted`
- `reservation.provider_rejected`
- `reservation.customer_cancelled`
- `reservation.completed` later

Listened events:
- `crm.lead_created` later if a reservation should be initiated from an inquiry.
- `claim.transferred` to preserve provider ownership consistency for future reservations.
- `messaging.conversation_created` where `reservation_id` is present, for diagnostics only.

Service contracts needed:
- `reservations.reservation_reader`: read reservation by ID and linkage fields.
- `reservations.availability_reader`: read availability/capacity summary.

Migration/refactor risk:
- Medium. Reservation state transitions should publish events after table updates only. No calendar or payment behavior should be introduced.

Implementation order:
1. Publish reservation and availability events after existing audit writes.
2. Register reservation reader contract.
3. Make CRM listen to reservation events to append timeline events.
4. Make messaging optionally listen to reservation requested/accepted events in a later phase.

Backwards compatibility:
- Keep existing reservation service transitions and WP-CLI commands unchanged.
- Event listeners must not change reservation status.

Validation plan:
- Extend reservation validation to assert platform event rows for requested/accepted/cancelled.
- Assert CRM timeline listener is additive when introduced.
- Assert no calendar/payment options are configured.

### `servicesbg-reviews`

Emitted events:
- `review.created`
- `review.provider_response_added`
- `review.moderated`
- `review_case.opened`
- `review_case.event_added`
- `review_case.resolved`

Listened events:
- `reservation.completed` later to enable review eligibility.
- `messaging.conversation_created` where `review_case_id` is present, for case diagnostics.
- `crm.lead_created` where review linkage exists, for admin context.

Service contracts needed:
- `reviews.private_review_reader`: read metadata/status only by review ID.
- `reviews.review_case_reader`: read case metadata/events without exposing private body text.
- `reviews.public_signal_reader`: aggregate approved/disputed counts for AI/search.

Migration/refactor risk:
- High. Raw reviews are private. Event payloads must not contain raw review body, provider response body, reviewer contact info, or private case text.

Implementation order:
1. Publish metadata-only review/case events.
2. Register privacy-safe reader contracts.
3. Make AI reviews listen for moderation/case resolution events to enqueue summary jobs.
4. Make CRM listen for timeline metadata only.

Backwards compatibility:
- Keep review tables and existing audit actions unchanged.
- Existing private review workflows continue without AI/search dependency.

Validation plan:
- Assert platform event rows exist for review workflow.
- Assert payload JSON excludes `private_body`, `provider_response_body`, and case event body.
- Assert AI summary job is queued only from approved/private-safe signals when listener is added.

### `servicesbg-ai-reviews`

Emitted events:
- `review_summary.job_enqueued`
- `review_summary.generated`
- `review_summary.approved`
- `review_summary.published`
- `review_summary.rejected`

Listened events:
- `review.moderated`
- `review_case.resolved`
- `reservation.completed` later
- `search.index_refresh_requested` is not consumed directly; AI reviews should publish summary events and let search listen.

Service contracts needed:
- `ai_reviews.summary_reader`: read approved/published summary metadata.
- `ai_reviews.summary_job_reader`: read local job status.

Migration/refactor risk:
- Medium to high. AI review summaries must remain deterministic/local in current scope and must not read or emit private raw text.

Implementation order:
1. Publish existing summary job/moderation audit actions as platform events.
2. Register summary reader contract.
3. Add listener that enqueues local summary jobs from safe review events.
4. Make search listen to `review_summary.published`.

Backwards compatibility:
- Keep existing local summary job table and CLI commands.
- Platform jobs may wrap existing AI review jobs later; do not remove the module-owned job table until an explicit migration is approved.

Validation plan:
- Assert generated/approved/published events.
- Assert no external LLM/API options are configured.
- Assert raw private review text is absent from event payloads and published summary payloads.

### `servicesbg-messaging`

Emitted events:
- `messaging.conversation_created`
- `messaging.participant_added`
- `messaging.message_sent`
- `messaging.conversation_read`
- `messaging.conversation_status_updated`

Listened events:
- `crm.lead_created` later to create/link an inquiry conversation if approved.
- `reservation.requested`, `reservation.provider_accepted`, `reservation.customer_cancelled` later for conversation timeline messages if approved.
- `review_case.opened` later to create/link review-case conversations.
- `claim.requested` later to create/link claim-case conversations.

Service contracts needed:
- `messaging.conversation_reader`: read conversation metadata by ID/linkage.
- `messaging.message_writer`: command contract for owner-approved message creation later.

Migration/refactor risk:
- Medium. Message body text is private. Platform events should contain message IDs and metadata only.

Implementation order:
1. Publish metadata events after existing message/conversation writes.
2. Register conversation reader contract.
3. Let CRM listen to messaging events for timeline updates.
4. Only later approve automatic conversation creation listeners.

Backwards compatibility:
- Keep existing messaging service and tables unchanged.
- Keep no delivery channels.
- Platform events are additive.

Validation plan:
- Extend messaging validation to assert event rows.
- Assert event payload excludes message body text.
- Assert CRM timeline listener creates metadata-only timeline rows when introduced.

### `servicesbg-crm`

Emitted events:
- `crm.lead_created`
- `crm.customer_profile_created`
- `crm.timeline_event_created`
- `crm.note_created`
- `crm.followup_created`
- `crm.pipeline_updated`

Listened events:
- `claim.requested`, `claim.verified`, `claim.approved`, `claim.rejected`
- `reservation.requested`, `reservation.provider_accepted`, `reservation.customer_cancelled`, `reservation.completed`
- `messaging.conversation_created`, `messaging.message_sent`, `messaging.conversation_status_updated`
- `review.created`, `review.moderated`, `review_case.opened`, `review_case.resolved`
- `coverage.zone_created`, `search.query_recorded` later for provider/customer signals.

Service contracts needed:
- `crm.lead_reader`: read lead and linkage metadata.
- `crm.customer_profile_reader`: read profile metadata.
- `crm.timeline_writer`: owner-approved command for appending metadata-only events.

Migration/refactor risk:
- Medium. CRM is the natural aggregator, so listener scope must stay metadata-only and must not become hidden workflow automation.

Implementation order:
1. Publish CRM events after existing audit writes.
2. Register read contracts.
3. Add listeners from reservations/messaging/reviews/claims to append timeline metadata.
4. Add validation for timeline events sourced from platform events.

Backwards compatibility:
- Keep current CRM service behavior.
- CRM listeners must not create reservations, messages, payments, or notifications.

Validation plan:
- Extend CRM validation to assert emitted platform events.
- Add listener smoke tests with synthetic platform events.
- Assert no delivery, payment, calendar, or AI automation options are configured.

### `servicesbg-core`

Emitted events:
- `audit.recorded`
- `audit.sensitive_action_recorded`
- `health.component_changed` later

Listened events:
- `platform.job_failed` for future admin diagnostics.
- `platform.event_published` is already represented by platform event log/audit and does not need core consumption in MVP.

Service contracts needed:
- `core.audit_reader`: read audit rows by module/action/object.
- `core.health_reader`: read current health components.

Migration/refactor risk:
- Low to medium. Core audit is widely used. Publishing audit events should be additive and avoid recursion where platform audit writes trigger audit events again.

Implementation order:
1. Add non-recursive audit event publication in core logger later.
2. Register audit/health reader contracts.
3. Update diagnostics to show platform adoption status.

Backwards compatibility:
- Keep `wp_servicesbg_audit_log` as source of truth.
- Keep current health page filters.

Validation plan:
- Assert audit events do not recurse.
- Assert core audit rows continue for all modules.
- Assert platform audit entries remain present.

### `servicesbg-platform`

Emitted events:
- `platform.job_enqueued`
- `platform.job_started` later
- `platform.job_completed`
- `platform.job_failed`

Listened events:
- None required for MVP. Platform should stay a kernel and avoid owning business reactions.

Service contracts needed:
- `platform.events`
- `platform.jobs`
- `platform.modules`
- Existing service registry entries from Phase 2H.

Migration/refactor risk:
- Medium. Event bus and job queue are shared primitives; changes affect all modules. Keep the API stable and add behavior behind new methods rather than changing existing helper signatures.

Implementation order:
1. Keep Phase 2H APIs stable.
2. Add event catalog validation helpers later.
3. Add module contract registration checks.
4. Add adoption dashboard after plugins publish/listen.

Backwards compatibility:
- Never remove Phase 2H helper functions without a deprecation window.
- Existing plugins work without platform listeners.

Validation plan:
- Keep `scripts/validate_platform_plugin.sh`.
- Add catalog conformance checks once events are implemented.
- Assert jobs and event rows are still created in staging.

## Cross-Module Implementation Order
1. Platform conformance helpers and docs only.
2. Core audit event publication and audit reader contract.
3. Search listener for `search.index_refresh_requested` and `search.index_refresh` jobs.
4. Coverage and claims event publication, because they drive search refresh.
5. Reservations event publication and CRM timeline listener.
6. Messaging event publication and CRM metadata listener.
7. Reviews event publication and AI reviews listener.
8. AI review summary events and search listener for published summaries.
9. CRM event publication and read/timeline contracts.
10. Remove legacy direct actions/filters only after validators prove platform parity.

## Backwards Compatibility Strategy
- Platform events are additive at first.
- Existing plugin services, WP-CLI commands, admin screens, tables, and audit rows remain authoritative.
- Existing WordPress actions/filters remain until equivalent platform events/jobs are validated.
- Event payloads carry IDs and state; consumers fetch richer data through owner contracts.
- Missing `servicesbg-platform` should fail closed only for new platform integrations; existing workflows should continue until the integration phase makes platform mandatory.

## Validation Strategy
Each plugin validation script should eventually add:
- plugin remains active and current smoke tests still pass,
- expected platform event rows are created,
- expected platform job rows are queued/completed where applicable,
- emitted payloads exclude private/secret fields,
- no direct includes from another ServicesBG plugin internal path are added,
- no production paths or URLs are targeted,
- no external APIs, notifications, payments, or calendar integrations are configured unless explicitly approved in a later phase.

## Non-Goals
- No new business workflow.
- No automatic notifications.
- No payment, calendar, external API, or AI automation integration.
- No theme changes.
- No refactor of existing plugins in this phase.
