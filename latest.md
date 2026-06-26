# ServicesBG Latest Report

Updated: 2026-06-26T12:27:45+03:00

## Status
Phase 2I workflow engine MVP is implemented and validated.

This phase implements cross-plugin workflow orchestration inside `servicesbg-platform`. It uses only the platform event bus and job queue. No new business features were implemented. No production services.bg system was modified.

## Deliverables
- `app/wp-content/plugins/servicesbg-platform/src/Plugin.php`
- `app/wp-content/plugins/servicesbg-platform/src/Service/WorkflowEngine.php`
- `docs/phase2i_workflow_engine_v1.md`
- `scripts/validate_workflow_engine.sh`
- `reports/latest.md`
- `reports/latest.json`

## Validated
- `scripts/validate_workflow_engine.sh` passed against `/opt/projects/servicesbg/wp-staging`.
- `scripts/validate_platform_plugin.sh` passed after the workflow engine change.
- Workflow registration works.
- Event consumption works.
- Platform job enqueue works.
- Idempotency prevents duplicate workflow jobs for the same source event.
- Retry behavior requeues after first failure and fails after max attempts.
- Workflow audit trail works.
- Workflows verified:
  - `reservation.completed` -> `review_invitation_create` job -> `review.invitation_created`
  - `claim.approved` -> `coverage_refresh` job + `search_index_refresh` job
  - `review.moderated` -> `review_summary_generate` deterministic marker job
- No notification/email/SMS/push/external API/payment/calendar/external LLM configuration is present.

## Events And Jobs
- Listened events:
  - `reservation.completed`
  - `claim.approved`
  - `review.moderated`
- Published workflow event:
  - `review.invitation_created`
- Enqueued job types:
  - `review_invitation_create`
  - `coverage_refresh`
  - `search_index_refresh`
  - `review_summary_generate`

## Architecture Rules Preserved
- `servicesbg-platform` remains the cross-plugin kernel.
- Cross-plugin communication uses platform events and platform jobs only.
- No direct plugin-to-plugin includes were added.
- No feature plugin internals are called from workflows.
- Existing platform behavior continues working.

## Explicitly Not Done
- no notifications
- no email
- no SMS
- no push
- no external APIs
- no external LLMs
- no production changes
- no payments
- no calendar integrations

## Next Step
Review workflow job contracts before allowing owner plugins to execute feature-specific side effects from these queued platform jobs.
