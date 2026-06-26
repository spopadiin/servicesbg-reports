# ServicesBG Latest Report

Updated: 2026-06-26T10:40:57+03:00

## Status
Phase 2H-F CRM platform events refactor is implemented and validated.

This phase refactors `servicesbg-crm` only. No new business features were implemented. No other plugins were refactored. No production services.bg system was modified.

## Deliverables
- `app/wp-content/plugins/servicesbg-crm/src/Service/CRMService.php`
- `docs/phase2h_crm_platform_events_v1.md`
- `scripts/validate_crm_plugin.sh`
- `reports/latest.md`
- `reports/latest.json`

## Validated
- `scripts/validate_crm_plugin.sh` passed against `/opt/projects/servicesbg/wp-staging`.
- `servicesbg-platform` active case writes platform event rows.
- `servicesbg-platform` inactive fallback has no fatal error.
- Existing CRM WP-CLI commands still work.
- Existing CRM workflow validation still works.
- Existing CRM audit entries still work.
- Event payloads include lead/customer/profile/provider/listing/reservation/conversation/claim/review/status/source/timestamp fields.
- Private CRM note body text is not emitted in platform event payloads.
- Events verified:
  - `crm.lead_created`
  - `crm.customer_profile_created`
  - `crm.timeline_event_created`
  - `crm.note_created`
  - `crm.followup_created`
  - `crm.pipeline_status_changed`

## Architecture Rules Preserved
- `servicesbg-crm` remains a WordPress plugin.
- Cross-plugin communication is additive through `servicesbg-platform`.
- No direct plugin-to-plugin includes were added.
- Existing CRM plugin behavior continues working.

## Explicitly Not Done
- no coverage
- no search
- no reviews
- no notifications
- no CRM automation
- no AI automation
- no email/SMS/push delivery
- no production changes
- no external APIs
- no payments
- no calendar integrations

## Next Step
Review the CRM event payloads before approving any listener automation through `servicesbg-platform`.
