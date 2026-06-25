# ServicesBG Messaging MVP v1

## Purpose
This document describes the implemented `servicesbg-messaging` MVP.

Scope is limited to internal messaging infrastructure: conversations, participants, messages, attachment metadata, read/unread tracking, linkage fields, admin diagnostics, WP-CLI commands, audit logging, and runtime validation.

## Explicitly Out of Scope
- AI chat.
- Chatbot.
- CRM workflows.
- Email delivery.
- SMS delivery.
- Push notifications.
- External APIs.
- Payments.
- Calendar integrations.

## Plugin
Path:
- `app/wp-content/plugins/servicesbg-messaging`

Main file:
- `app/wp-content/plugins/servicesbg-messaging/servicesbg-messaging.php`

The plugin includes:
- WordPress plugin header.
- Activation/deactivation hooks.
- Local autoloader.
- Schema installer/version manager.
- Conversation service.
- Participant service behavior.
- Message service behavior.
- Read/unread tracking.
- Admin diagnostics UI.
- WP-CLI commands.
- Health/status integration.
- Core audit integration.

## Data Model

### Table: `wp_servicesbg_conversations`
Purpose:
- Thread-level state for marketplace messaging.

Implemented fields include:
- `conversation_type`
- `listing_id`
- `provider_user_id`
- `customer_user_id`
- `lead_id`
- `reservation_id`
- `review_case_id`
- `claim_id`
- `channel`
- `status`
- `subject`
- `last_message_id`
- `last_message_at`
- `source_thread_id`
- `source_payload_json`
- timestamps

Supported conversation types:
- `provider_customer`
- `reservation`
- `review_case`
- `claim_case`
- `admin`

Supported statuses:
- `open`
- `pending`
- `closed`
- `archived`

### Table: `wp_servicesbg_conversation_participants`
Purpose:
- Participant tracking, roles, joined/left state, and unread counters.

Implemented fields include:
- `conversation_id`
- `participant_type`
- `user_id`
- `display_name`
- `role`
- `status`
- `unread_count`
- `last_read_message_id`
- `last_read_at`
- timestamps

### Table: `wp_servicesbg_messages`
Purpose:
- Message rows for internal conversation infrastructure.

Implemented fields include:
- `conversation_id`
- `sender_type`
- `sender_user_id`
- `body`
- `visibility`
- `status`
- `ai_generated`
- `ai_model`
- `source_message_id`
- `source_payload_json`
- `attachment_id`
- timestamps

`source_message_id` preserves `fl_messages.ID` if historical import is approved later.

### Table: `wp_servicesbg_message_attachments`
Purpose:
- Attachment metadata for message-related files.

Implemented fields include:
- `message_id`
- `conversation_id`
- `attachment_id`
- `file_name`
- `mime_type`
- `file_size`
- `source_attachment_id`
- `source_path`
- `status`
- `metadata_json`
- timestamps

The MVP stores metadata only. It does not upload, deliver, scan, or import files.

## Workflows

### Create Conversation
Creates a conversation row with optional links to:
- listing,
- lead,
- reservation,
- review case,
- claim.

### Add Participants
Adds participants with a role, type, status, and unread state.

### Send Message
Creates a message row, updates conversation `last_message_id` and `last_message_at`, increments unread count for active non-sender participants, and records audit.

### Mark Read
Sets a participant unread count to zero and stores the last read message.

## Admin Diagnostics
Admin page:
- `ServicesBG > Messaging`
- slug: `servicesbg-messaging`

Capability:
- `servicesbg_manage_messaging`

Diagnostics show:
- schema version,
- conversation count,
- participant count,
- message count,
- attachment metadata count,
- conversation status counts,
- supported conversation types,
- excluded integrations.

## WP-CLI Commands
Implemented:
- `wp servicesbg messaging status`
- `wp servicesbg messaging list`
- `wp servicesbg messaging conversation-create`
- `wp servicesbg messaging message-create`
- `wp servicesbg messaging create-test`

The smoke test validates:
- temporary listing creation,
- conversation creation,
- reservation/review-case/claim linkage fields,
- participant creation,
- message creation,
- unread tracking,
- read marker update,
- attachment metadata row,
- audit events.

## Audit Logging
Messaging writes audit records into:
- `wp_servicesbg_audit_log`

Implemented audit actions:
- `messaging.conversation_created`
- `messaging.participant_added`
- `messaging.message_sent`
- `messaging.conversation_read`
- `messaging.conversation_status_updated`

## Runtime Validation
Dedicated validation script:
- `scripts/validate_messaging_plugin.sh`

Latest staging result:
- Passed.

The validation script checks:
- staging DB connection,
- plugin activation,
- conversation/participant/message/attachment tables,
- required table columns,
- health integration,
- admin diagnostics page,
- admin capability,
- WP-CLI commands,
- conversation creation,
- participant tracking,
- message creation,
- unread tracking,
- attachment metadata,
- audit logging,
- no AI/chatbot/CRM/email/SMS/push/external API/payment/calendar configuration.

## Safety Guarantees
- No production URL/path is targeted by the validation script.
- No AI chat or chatbot option is configured.
- No email, SMS, push, or external API provider is configured.
- No CRM, payment, or calendar option is configured.
- No delivery channel is invoked.
- Flynax message/source IDs are preserved only as traceability fields; no importer is implemented in this phase.
