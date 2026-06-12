# ServicesBG Phase 2A Claims Implementation Report

Updated: 2026-06-12T09:31:54+03:00

## Status
`servicesbg-claims` has been implemented and validated in the staging WordPress runtime.

No production services.bg system was modified.

## Implemented
- `servicesbg-claims` plugin skeleton and bootstrap.
- Activation/deactivation hooks.
- Autoloader structure.
- Claims schema/version manager.
- `wp_servicesbg_claims` table.
- Claim request workflow foundation.
- OTP claim model using hashed OTP values.
- Claim status and review status tracking.
- Migration support fields for:
  - Flynax listing ID
  - Flynax account ID
  - temporary source account flag
- Phone normalization and claimability abstraction.
- SMS provider adapter interface.
- Null SMS adapter for no-provider mode.
- Admin claim review screen.
- Admin capability: `servicesbg_manage_claims`.
- Audit logging into `wp_servicesbg_audit_log`.
- WP-CLI namespace:
  - `wp servicesbg claims`
  - `wp servicesbg claims status`
  - `wp servicesbg claims request`
- Core health page integration.

## Explicitly Not Implemented
- Actual SMS provider integration.
- Reservations.
- AI.
- Messaging.
- CRM.
- Coverage.
- Search.
- Importers.
- Ownership transfer business logic beyond status/review foundations.

## Staging Validation
Validated in `/opt/projects/servicesbg/wp-staging`.

Passed:
- `servicesbg-claims` plugin activation.
- `wp_servicesbg_claims` table exists.
- Core health page still registers.
- Existing Phase 1 plugins remain active.
- Existing Phase 1 tables remain present.
- `service_listing` CPT remains registered.
- `service_category` taxonomy remains registered.
- Provider roles remain present.
- WP-CLI placeholders work:
  - `wp servicesbg`
  - `wp servicesbg status`
  - `wp servicesbg migration`
  - `wp servicesbg migration preflight`
  - `wp servicesbg claims`
  - `wp servicesbg claims status`

## Validation Commands
- `find app/wp-content/plugins/servicesbg-claims -name '*.php' -print0 | xargs -0 -n1 php -l`
- `php -l app/wp-content/plugins/servicesbg-core/src/Admin/HealthPage.php`
- `bash -n scripts/setup_staging_wp.sh`
- `bash -n scripts/validate_phase1_plugins.sh`
- `scripts/setup_staging_wp.sh`
- `scripts/validate_phase1_plugins.sh`

## Notes
- `scripts/validate_phase1_plugins.sh` now validates the Phase 1 foundation plus Phase 2A claims module.
- The null SMS adapter intentionally records a failed SMS-send state because no SMS provider has been configured yet.
- Claim request records can be created through service code or WP-CLI, but real phone delivery remains deferred.

