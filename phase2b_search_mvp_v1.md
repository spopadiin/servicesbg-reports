# ServicesBG Search MVP v1

## Purpose
This document describes the implemented `servicesbg-search` MVP.

Scope is limited to a ServicesBG-owned WordPress/MariaDB search index, basic marketplace filters, admin diagnostics, WP-CLI, audit logging, and runtime validation.

## Explicitly Out of Scope
- Reservations.
- Reviews.
- AI.
- Messaging.
- CRM workflows.
- Maps.
- GIS libraries.
- Routing.
- Elasticsearch.
- Vector search.
- AI ranking.

## Plugin
Path:
- `app/wp-content/plugins/servicesbg-search`

Main file:
- `app/wp-content/plugins/servicesbg-search/servicesbg-search.php`

The plugin includes:
- WordPress plugin header.
- Activation/deactivation hooks.
- Local autoloader.
- Search schema installer.
- Search indexer.
- Query service.
- Deterministic ranking service.
- Query telemetry.
- Admin diagnostics UI.
- WP-CLI commands.
- Health/status integration.
- Audit logging.

## Data Model

### Table: `wp_servicesbg_search_index`
Purpose:
- Denormalized search index for listings, providers, and categories.
- Supports filtering without making search the source of truth.

Supported indexed fields include:
- `object_type`
- `object_id`
- `listing_id`
- `provider_user_id`
- `category_term_id`
- `title`
- `summary`
- `keywords`
- `country`
- `region`
- `city`
- `coverage_status`
- `claim_status`
- `listing_status`
- `rank_score`
- `source_listing_id`
- `source_account_id`

### Table: `wp_servicesbg_search_queries`
Purpose:
- Diagnostic query logging for MVP search validation and tuning.

## Supported Filters
The MVP query service supports:
- text query,
- object type,
- category term ID,
- provider user ID,
- country,
- region,
- city,
- coverage status,
- claim status,
- claimed/unclaimed boolean filter,
- listing status.

## Coverage Filtering
Coverage filtering is simple and table-based:
- search reads indexed `coverage_status`, `country`, `region`, and `city`,
- if `servicesbg-coverage` is active, listing index rows can derive simple active/suggested coverage zones,
- search does not own coverage zones,
- search does not implement maps, GIS, or routing.

## Claimed/Unclaimed Filtering
Claim filtering uses indexed `claim_status`.

Claimed values:
- `claimed`
- `verified`
- `transferred`

Unclaimed means the row is not in one of those states.

## Provider Filtering
Provider filtering uses indexed `provider_user_id`.

For listings, `provider_user_id` is the listing author at indexing time.

## Ranking
Ranking is deterministic and local:
- object type score,
- listing status score,
- claim status score,
- text match score.

No AI ranking is implemented.

## Admin Diagnostics
Admin page:
- `ServicesBG > Search`
- slug: `servicesbg-search`

Capability:
- `servicesbg_manage_search`

Diagnostics show:
- schema version,
- indexed row count,
- logged query count,
- supported filters,
- disabled engine status for AI, Elasticsearch, vector search, maps, GIS, and routing.

## WP-CLI Commands
Implemented:
- `wp servicesbg search status`
- `wp servicesbg search rebuild-index`
- `wp servicesbg search test`
- `wp servicesbg search query`
- `wp servicesbg search diagnose`

The smoke test validates:
- listing indexing,
- category indexing,
- provider indexing,
- category filtering,
- coverage filtering,
- claimed filtering,
- provider filtering,
- query logging,
- cleanup of temporary objects.

## Audit Logging
Search writes audit records into:
- `wp_servicesbg_audit_log`

Implemented audit actions:
- `search.index_updated`
- `search.query_recorded`

## Runtime Validation
Dedicated validation script:
- `scripts/validate_search_plugin.sh`

Latest staging result:
- Passed.

The validation script checks:
- staging DB connection,
- plugin activation,
- search index and query tables,
- required table columns,
- `service_listing` CPT,
- `service_category` taxonomy,
- health integration,
- admin diagnostics page,
- admin capability,
- WP-CLI commands,
- query filters,
- audit logging,
- no AI/Elasticsearch/vector/maps/GIS/routing configuration.

## Safety Guarantees
- No production URL/path is targeted by the validation script.
- No Elasticsearch host is configured.
- No vector provider is configured.
- No AI ranking option is configured.
- No maps/GIS/routing configuration is present.
- Search remains a read-side index and is not the source of truth for listings, providers, categories, coverage, or claims.

