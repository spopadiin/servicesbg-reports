# ServicesBG Phase 2J Job Handler Contracts MVP v1

## Purpose
This document describes the platform job handler contract MVP.

Feature plugins can now register platform job handlers during bootstrap. The platform dispatcher resolves handlers through a registry and never includes or calls feature plugin internals directly.

## Scope
Changed platform files:
- `app/wp-content/plugins/servicesbg-platform/servicesbg-platform.php`
- `app/wp-content/plugins/servicesbg-platform/src/Plugin.php`
- `app/wp-content/plugins/servicesbg-platform/src/Service/JobHandlerRegistry.php`
- `app/wp-content/plugins/servicesbg-platform/src/Service/WorkflowQueue.php`
- `app/wp-content/plugins/servicesbg-platform/src/Service/WorkflowEngine.php`

Changed feature plugin files:
- `app/wp-content/plugins/servicesbg-search/src/Service/PlatformRefreshListener.php`
- `app/wp-content/plugins/servicesbg-coverage/src/Plugin.php`
- `app/wp-content/plugins/servicesbg-coverage/src/Service/PlatformJobHandler.php`
- `app/wp-content/plugins/servicesbg-ai-reviews/src/Plugin.php`
- `app/wp-content/plugins/servicesbg-ai-reviews/src/Service/PlatformJobHandler.php`

Created validation:
- `scripts/validate_job_handlers.sh`

## Platform Contract
New public helper:
- `servicesbg_platform_register_job_handler(string $handlerKey, callable $handler, array $args = []): void`

New platform service:
- `platform.job_handlers`

Handler registration metadata:
- `owner_module`
- `job_types`

The dispatcher:
- loads the queued platform job,
- resolves the handler by `handler_key`,
- validates ownership against the declared job types,
- executes the registered callable,
- preserves existing retry behavior,
- audits resolution, missing handlers, completion, and failure.

Unknown handlers do not fatal. They are audited as `platform.job_handler_missing`, then follow normal platform retry behavior.

## Registered Feature Handlers
`servicesbg-search` registers:
- `handler_key`: `search_index_refresh`
- `job_type`: `search_index_refresh`
- owner: `servicesbg-search`

`servicesbg-coverage` registers:
- `handler_key`: `coverage_refresh`
- `job_type`: `coverage_refresh`
- owner: `servicesbg-coverage`

`servicesbg-ai-reviews` registers:
- `handler_key`: `review_summary_generate`
- `job_type`: `review_summary_generate`
- owner: `servicesbg-ai-reviews`

Coverage and AI review handlers are marker-only in this phase. They acknowledge the job contract and return metadata without creating coverage zones, summaries, external calls, or notifications.

## Workflow Compatibility
Phase 2I workflow behavior is preserved:
- `reservation.completed` still creates `review_invitation_create` and publishes `review.invitation_created`.
- `claim.approved` still creates `coverage_refresh` and `search_index_refresh`.
- `review.moderated` still creates `review_summary_generate`.

The difference is that `coverage_refresh`, `search_index_refresh`, and `review_summary_generate` are now owned by feature plugin registrations instead of platform-owned marker handlers.

## Audit
Added platform audit actions:
- `platform.job_handler_registered`
- `platform.job_handler_resolved`
- `platform.job_handler_missing`

Existing platform audit actions remain active:
- `platform.job_enqueued`
- `platform.job_completed`
- `platform.job_failed`
- `platform.workflow_event_consumed`
- `platform.workflow_job_enqueued`
- `platform.workflow_job_deduplicated`
- `platform.workflow_event_published`

## Validation
Created script:
- `scripts/validate_job_handlers.sh`

Validation checks:
- PHP lint for changed platform and feature plugin files,
- shell syntax,
- staging target guard,
- staging database connection,
- activation for platform, search, coverage, and AI reviews,
- handler registration and owner metadata,
- dispatcher resolution for all three registered feature handlers,
- missing-handler safe failure,
- retry behavior after missing handler,
- job handler audit entries,
- existing Phase 2I workflow validation still passes,
- no notification/email/SMS/push/external API/payment/calendar/external LLM configuration.

Additional regression validation:
- `scripts/validate_platform_plugin.sh` passed after dispatcher changes.

## Explicitly Not Implemented
- notifications,
- email,
- SMS,
- push,
- external APIs,
- payments,
- calendar integrations,
- external LLMs,
- new feature business behavior,
- production changes.
