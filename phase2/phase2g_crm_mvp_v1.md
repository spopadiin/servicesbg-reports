# ServicesBG CRM MVP v1

## Purpose
This document describes the implemented `servicesbg-crm` MVP.

Scope is limited to CRM infrastructure: leads/inquiries, customer profiles, customer timelines, CRM notes, follow-up tasks, pipeline/status tracking, cross-module linkage fields, admin diagnostics, WP-CLI commands, health/status integration, audit logging, and runtime validation.

## Explicitly Out of Scope
- AI automation.
- Email delivery.
- SMS delivery.
- Push notifications.
- External APIs.
- Payments.
- Calendar integrations.

## Plugin
Path:
- `app/wp-content/plugins/servicesbg-crm`

Main file:
- `app/wp-content/plugins/servicesbg-crm/servicesbg-crm.php`

The plugin includes:
- WordPress plugin header.
- Activation/deactivation hooks.
- Local autoloader.
- Schema installer/version manager.
- CRM write service.
- Lead/inquiry creation.
- Customer profile creation/reuse.
- Customer timeline events.
- Provider customer history through linked customer/profile/timeline rows.
- CRM notes.
- Follow-up tasks.
- Pipeline/status tracking.
- Admin diagnostics UI.
- WP-CLI commands.
- Health/status integration.
- Core audit integration.

## Data Model

### Table: `wp_servicesbg_leads`
Purpose:
- CRM source of truth for marketplace leads and inquiries.

Implemented fields include:
- `listing_id`
- `provider_user_id`
- `customer_user_id`
- `customer_profile_id`
- `conversation_id`
- `reservation_id`
- `review_id`
- `review_case_id`
- `claim_id`
- `source`
- `channel`
- `status`
- `priority`
- customer snapshot fields
- service request fields
- `value_estimate`
- `conversion_score`
- `source_contact_id`
- `source_payload_json`
- timestamps

`source_contact_id` preserves `fl_contacts.ID` if contact import is approved later.

### Table: `wp_servicesbg_customer_profiles`
Purpose:
- Customer identity/contact snapshot for lead history.

Implemented fields include:
- `user_id`
- `source_account_id`
- display/contact fields
- `status`
- `first_lead_id`
- `last_lead_id`
- `last_provider_user_id`
- `last_contact_at`
- `source_payload_json`
- timestamps

### Table: `wp_servicesbg_customer_timeline`
Purpose:
- Customer and lead history for provider/customer interactions.

Implemented linkage fields include:
- lead,
- provider,
- listing,
- reservation,
- review,
- review case,
- conversation,
- claim.

### Table: `wp_servicesbg_crm_notes`
Purpose:
- Internal CRM notes linked to leads, customers, providers, listings, reservations, reviews, conversations, and claims.

### Table: `wp_servicesbg_followup_tasks`
Purpose:
- Follow-up task tracking with assignee, due date, status, priority, and cross-module linkage fields.

### Table: `wp_servicesbg_pipeline_items`
Purpose:
- Pipeline/stage/status tracking for leads and linked customer/provider/listing context.

Supported lead statuses:
- `new`
- `open`
- `contacted`
- `qualified`
- `converted`
- `lost`
- `archived`

Supported pipeline stages:
- `new`
- `contacted`
- `qualified`
- `proposal`
- `booked`
- `won`
- `lost`

Supported pipeline statuses:
- `open`
- `won`
- `lost`
- `archived`

## Workflows

### Lead Creation
Creates a lead row, creates or reuses a customer profile, records a customer timeline event, creates an initial pipeline item, and writes audit entries.

### Customer Profile Timeline
Timeline events are created for lead creation, CRM notes, follow-up tasks, and pipeline updates.

### Provider Customer History
Provider history is queryable through `provider_user_id` on leads, timeline events, notes, follow-ups, and pipeline items.

### CRM Notes
Creates internal notes and linked timeline/audit records.

### Follow-up Tasks
Creates task rows with due dates, assignee, status, priority, and linked timeline/audit records.

### Pipeline Updates
Creates or updates a pipeline item for a lead and records timeline/audit entries.

## Admin Diagnostics
Admin page:
- `ServicesBG > CRM`
- slug: `servicesbg-crm`

Capability:
- `servicesbg_manage_crm`

Diagnostics show:
- schema version,
- lead count,
- customer profile count,
- customer timeline count,
- CRM note count,
- follow-up task count,
- pipeline item count,
- lead status counts,
- pipeline status counts,
- excluded integrations.

## WP-CLI Commands
Implemented:
- `wp servicesbg crm status`
- `wp servicesbg crm leads`
- `wp servicesbg crm lead-create`
- `wp servicesbg crm customer-profile`
- `wp servicesbg crm followup-create`
- `wp servicesbg crm create-test`

The smoke test validates:
- temporary listing creation,
- lead creation,
- customer profile creation,
- timeline event creation,
- CRM note creation,
- follow-up task creation,
- pipeline status update,
- listing/provider/reservation/review/conversation/claim linkage fields,
- core audit entries.

## Audit Logging
CRM writes audit records into:
- `wp_servicesbg_audit_log`

Implemented audit actions:
- `crm.lead_created`
- `crm.customer_profile_created`
- `crm.timeline_event_created`
- `crm.note_created`
- `crm.followup_created`
- `crm.pipeline_updated`

## Runtime Validation
Dedicated validation script:
- `scripts/validate_crm_plugin.sh`

The validation script checks:
- PHP lint,
- shell syntax,
- staging DB connection,
- staging plugin activation,
- CRM tables,
- required table columns,
- health integration,
- admin diagnostics page,
- admin capability,
- WP-CLI commands,
- lead creation,
- customer profile creation,
- timeline event creation,
- follow-up creation,
- pipeline status update,
- audit logging,
- no AI automation/email/SMS/push/external API/payment/calendar configuration.

## Safety Guarantees
- No production URL/path is targeted by the validation script.
- No AI automation option is configured.
- No email, SMS, push, or external API provider is configured.
- No payment or calendar option is configured.
- No delivery channel is invoked.
- Flynax contact/account source IDs are preserved only as traceability fields; no importer is implemented in this phase.
