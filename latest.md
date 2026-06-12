# ServicesBG Latest Report

Updated: 2026-06-12T09:31:54+03:00

## Status
Architecture updated to include `servicesbg-reviews` with private raw reviews and AI-assisted public summary versions.

No application code was written for reviews. No production services.bg system was modified.

## Created/Updated
- `docs/reviews_ai_model_v1.md`
- `docs/servicesbg_database_schema_v1.md`
- `docs/servicesbg_plugin_table_ownership_v1.md`
- `docs/servicesbg_mvp_scope_v1.md`
- `reports/latest.md`
- `reports/latest.json`

## Review Model Decisions
- Raw reviews are private between reviewer, provider, and admin.
- Public review output is not a raw review list.
- Public review output is a versioned AI-assisted summary.
- Public summaries may use raw reviews, provider responses, resolution outcomes, reservation history, inquiry/response activity, and verified service completion signals.
- Admin must be able to audit raw reviews, cases, AI inputs, generated summaries, redaction decisions, approvals, publications, and prior versions.
- Public summaries must be regeneratable.
- AI must not generate fake reviews or invent unsupported service experiences.
- AI summaries must not expose private personal details.

## Schema Added to Phase 2 Planning
- `wp_servicesbg_reviews`
- `wp_servicesbg_review_cases`
- `wp_servicesbg_review_case_events`
- `wp_servicesbg_review_summary_versions`
- `wp_servicesbg_review_summary_audit`

## Ownership
- `servicesbg-reviews` owns private raw reviews, cases, provider responses, summary versions, and review summary audit.
- `servicesbg-ai` assists generation/redaction through contracts but does not own review records.
- `servicesbg-search` may consume only approved public summary fields and aggregate trust signals.

## Next Step
Review and approve the reviews architecture before any `servicesbg-reviews` plugin implementation.

