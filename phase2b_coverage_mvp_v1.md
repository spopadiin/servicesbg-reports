# ServicesBG Coverage MVP v1

## Purpose
This document describes the implemented `servicesbg-coverage` MVP.

Scope is limited to simple provider/listing coverage zones, validation, admin management, WP-CLI, and staging validation.

## Explicitly Out of Scope
- Search.
- Reservations.
- Reviews.
- AI.
- Messaging.
- CRM workflows.
- Maps.
- GIS libraries.
- Routing.
- Coverage discovery automation.

## Plugin
Path:
- `app/wp-content/plugins/servicesbg-coverage`

Main file:
- `app/wp-content/plugins/servicesbg-coverage/servicesbg-coverage.php`

The plugin includes:
- WordPress plugin header.
- Activation/deactivation hooks.
- Local autoloader.
- Schema installer.
- Coverage validation service.
- Coverage zone service.
- Admin management UI.
- WP-CLI commands.
- Health/status integration.
- Audit logging.

## Data Model

### Table: `wp_servicesbg_coverage_zones`
Created by the `servicesbg-coverage` activation hook.

Purpose:
- Assign a coverage zone to a provider, listing, or both.
- Preserve source Flynax account/listing IDs where available.
- Store simple city/region/country/radius coverage data.
- Track status and priority.

Key fields:
- `provider_user_id`
- `listing_id`
- `zone_type`
- `label`
- `country`
- `region`
- `city`
- `center_latitude`
- `center_longitude`
- `radius_km`
- `travel_fee`
- `min_order_value`
- `priority`
- `status`
- `source_account_id`
- `source_listing_id`
- `source_location_json`

`polygon_json` exists as a reserved field but maps/GIS behavior is not implemented.

## Zone Types
Allowed MVP zone types:
- `city`
- `region`
- `radius`
- `country`
- `manual`

Allowed statuses:
- `active`
- `inactive`
- `suggested`
- `needs_review`

## Validation Rules
`CoverageValidator` enforces:
- a zone must reference `provider_user_id` or `listing_id`,
- `zone_type` must be allowed,
- `status` must be allowed,
- non-radius zones need country, region, or city,
- radius zones need a positive `radius_km`.

## Admin Management UI
Admin page:
- `ServicesBG > Coverage`
- slug: `servicesbg-coverage`

Capability:
- `servicesbg_manage_coverage`

The page displays:
- coverage schema version,
- total coverage zones,
- active coverage zones,
- maps/GIS/routing disabled status.

## WP-CLI Commands
Implemented:
- `wp servicesbg coverage status`
- `wp servicesbg coverage list`
- `wp servicesbg coverage create`
- `wp servicesbg coverage create-test`

`create-test`:
- creates a temporary draft `service_listing`,
- creates a simple city coverage zone,
- stores Flynax source account/listing IDs,
- writes an audit event,
- can remove the test zone and listing with `--cleanup`.

Example:

```bash
wp servicesbg coverage create-test --cleanup --path=/opt/projects/servicesbg/wp-staging
```

## Audit Logging
Coverage writes audit records into:
- `wp_servicesbg_audit_log`

Implemented audit actions:
- `coverage.zone_created`
- `coverage.zone_status_updated`

## Runtime Validation
Dedicated validation script:
- `scripts/validate_coverage_plugin.sh`

The script validates:
- staging path guard,
- staging DB connection,
- `servicesbg-coverage` activation,
- `wp_servicesbg_coverage_zones` table,
- required columns,
- admin coverage page registration,
- `servicesbg_manage_coverage` capability,
- WP-CLI commands,
- safe `create-test --cleanup`,
- audit-log integration,
- no maps/GIS/routing options configured.

Latest staging result:
- Passed.

## Safety Guarantees
- No production URL/path is targeted by the validation script.
- No maps provider is configured.
- No GIS library is used.
- No routing logic is used.
- Search, reservations, reviews, AI, messaging, and CRM workflows are not touched.
- Flynax source IDs can be preserved through `source_account_id` and `source_listing_id`.

