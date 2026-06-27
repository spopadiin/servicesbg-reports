# ServicesBG Latest Report

Updated: 2026-06-27T20:42:22+03:00

## Status
Phase 3B-1 plugin view-model and template API contracts are implemented and validated.

This phase added read-only PHP contracts and sanitized template-safe view-model providers for future ServicesBG theme and `servicesbg-ui` consumers. No theme, CSS, JavaScript, route changes, legacy Flynax assets, external APIs, production changes, or business workflow writes were implemented.

## Deliverables
- `app/wp-content/plugins/servicesbg-ui`
- `docs/phase3b_view_model_api_contracts_v1.md`
- `docs/phase3b_template_api_matrix_v1.md`
- `scripts/validate_view_model_contracts.sh`
- `reports/latest.md`
- `reports/latest.json`

## Implemented View Models
- `listing_card`
- `listing_detail_header`
- `listing_detail_body`
- `provider_card`
- `provider_profile_summary`
- `search_filters`
- `search_results`
- `claim_status`
- `coverage_summary`
- `reservation_cta`
- `private_review_summary_state`
- `public_ai_review_summary`
- `messaging_contact_action`
- `crm_provider_dashboard_summary`
- `workflow_status_summary`
- `breadcrumbs`
- `category_navigation`

## Owner Services
- `listings.view_models`
- `providers.view_models`
- `search.view_models`
- `claims.view_models`
- `coverage.view_models`
- `reservations.view_models`
- `reviews.view_models`
- `ai_reviews.view_models`
- `messaging.view_models`
- `crm.view_models`
- `platform.workflow_view_models`
- `ui.view_models`

## Validation
`./scripts/validate_view_model_contracts.sh` passed against `/opt/projects/servicesbg/wp-staging`.

Validated:
- PHP syntax for contract and provider files.
- `servicesbg-ui` WP-CLI diagnostics.
- all required view models return normalized arrays.
- all view models are marked read-only.
- platform view-model services are registered.
- claim view models do not expose OTP/SMS secrets.
- private review view models do not expose private review/provider response body fields.

## Architecture Rules Preserved
- Theme remains presentation-only.
- Business logic remains inside owner plugins.
- View models are read-only.
- No direct theme access to plugin DB tables is required.
- Permission and privacy checks happen in owner providers.
- Fallback empty states are normalized.
- No production changes.
- No external APIs.
- No legacy Flynax HTML/CSS/assets copied.

## Explicitly Not Done
- no WordPress theme implementation
- no CSS
- no JavaScript
- no frontend components
- no REST endpoints
- no shortcodes or blocks
- no route/rewrite changes
- no production redirects
- no external API calls
- no business workflow writes

## Next Step
Phase 3B can implement theme/template rendering against `servicesbg_ui_view_model()` after confirming final public permalink patterns and any required REST/block exposure strategy.
