# ServicesBG Phase 1 Runtime Validation Plan v1

## Purpose
This document defines how to validate the Phase 1 ServicesBG WordPress plugin foundation in a staging runtime.

It covers only:
- `servicesbg-core`
- `servicesbg-listings`
- `servicesbg-providers`
- `servicesbg-migration`

It explicitly does not implement or validate claims, reservations, AI, messaging, coverage, CRM, search, or migration importers.

## Safety Rules
- Do not run these scripts against production services.bg.
- Do not use the Flynax source database `services_services` as the WordPress staging database.
- Use `/opt/projects/servicesbg/wp-staging` or another path under `/opt/projects/servicesbg/wp-staging*`.
- Use a staging URL such as `http://servicesbg-staging.local`, not `https://services.bg`.
- Use a dev/staging WordPress database such as `servicesbg_wp_staging`.
- The scripts are designed to refuse obvious production paths, production URLs, and the Flynax source database.

## Created Scripts

### `scripts/setup_staging_wp.sh`
Installs and prepares a staging WordPress runtime.

What it does:
- Downloads WordPress into `/opt/projects/servicesbg/wp-staging` by default.
- Creates `wp-config.php` using staging DB settings.
- Sets `WP_ENVIRONMENT_TYPE` to `staging`.
- Installs WordPress if not already installed.
- Symlinks the four Phase 1 plugins from `repo/app/wp-content/plugins`.
- Activates the four Phase 1 plugins.
- Flushes rewrite rules.

Default environment:

| Variable | Default |
| --- | --- |
| `PROJECT_ROOT` | `/opt/projects/servicesbg` |
| `REPO_ROOT` | `/opt/projects/servicesbg/repo` |
| `WP_STAGING_ROOT` | `/opt/projects/servicesbg/wp-staging` |
| `WP_STAGING_URL` | `http://servicesbg-staging.local` |
| `WP_STAGING_DB_NAME` | `servicesbg_wp_staging` |
| `WP_STAGING_DB_USER` | `servicesbg` |
| `WP_STAGING_DB_HOST` | `localhost` |
| `PLUGIN_LINK_MODE` | `symlink` |

Required environment:
- `WP_STAGING_DB_PASSWORD`
- `WP_STAGING_ADMIN_PASSWORD`, unless local WP-CLI can generate one during setup.

Example:

```bash
export WP_STAGING_DB_PASSWORD='replace-me'
export WP_STAGING_ADMIN_PASSWORD='replace-me'
./scripts/setup_staging_wp.sh
```

Use copied plugins instead of symlinks:

```bash
export PLUGIN_LINK_MODE=copy
./scripts/setup_staging_wp.sh
```

### `scripts/validate_phase1_plugins.sh`
Validates the Phase 1 runtime after setup.

Checks:
- All four plugins are active.
- Base custom tables exist:
  - `wp_servicesbg_audit_log`
  - `wp_servicesbg_import_batches`
  - `wp_servicesbg_migration_map`
- `service_listing` CPT is registered.
- `service_category` taxonomy is registered.
- Provider roles exist:
  - `service_provider`
  - `service_freelancer`
  - `service_customer`
- Admin health/status page is registered as `servicesbg-health`.
- WP-CLI placeholders work:
  - `wp servicesbg status`
  - `wp servicesbg migration preflight`

Run:

```bash
./scripts/validate_phase1_plugins.sh
```

## Expected Result
The validation script should print a sequence of `PASS` lines and finish with:

```text
Phase 1 staging runtime validation passed.
```

## Execution Result: 2026-06-12
The staging validation flow was executed from `/opt/projects/servicesbg/repo`.

Observed setup result:
- `/opt/projects/servicesbg/wp-staging` was targeted.
- WordPress was downloaded into the staging path.
- `wp-config.php` was generated.
- The separate staging database `servicesbg_wp_staging` was created.
- The four Phase 1 plugins were symlinked into staging and activated.

Observed validation result:
- `scripts/validate_phase1_plugins.sh` was executed.
- Validation passed.
- One Phase 1 bug was found and fixed during validation: the root `wp servicesbg` command placeholder originally prevented nested WP-CLI subcommands. The root command was changed to a namespace-style command and validation was expanded to check all required placeholders.

Passed checks:
- WordPress staging database connection.
- Plugin activation for `servicesbg-core`, `servicesbg-listings`, `servicesbg-providers`, and `servicesbg-migration`.
- `wp_servicesbg_audit_log`.
- `wp_servicesbg_import_batches`.
- `wp_servicesbg_migration_map`.
- `service_listing` CPT.
- `service_category` taxonomy.
- `service_provider`, `service_freelancer`, and `service_customer` roles.
- Admin health page registration: `servicesbg-health`.
- WP-CLI placeholders:
  - `wp servicesbg`
  - `wp servicesbg status`
  - `wp servicesbg migration`
  - `wp servicesbg migration preflight`

## What Success Means
Success confirms:
- The plugin skeletons can activate in WordPress.
- Activation hooks create the intended Phase 1 tables.
- The listing CPT and category taxonomy are registered.
- Provider role placeholders are installed.
- The admin health page is available to WordPress admin users.
- The WP-CLI command namespace is registered.

## What Success Does Not Mean
Success does not validate:
- Flynax data imports.
- Migration rollback.
- Media migration.
- Redirect generation.
- Claims by SMS OTP.
- Reservations/calendar.
- AI.
- Messaging.
- Coverage zones.
- Lead CRM.
- Search.

Those areas remain intentionally deferred.

## Troubleshooting

### WordPress download fails
Check network access and WP-CLI installation:

```bash
wp --info
```

### Database creation fails
Create the staging database manually, then rerun setup:

```sql
CREATE DATABASE servicesbg_wp_staging CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Plugin activation fails
Run a PHP syntax check from the repo:

```bash
find app/wp-content/plugins/servicesbg-core app/wp-content/plugins/servicesbg-listings app/wp-content/plugins/servicesbg-providers app/wp-content/plugins/servicesbg-migration -name '*.php' -print0 | xargs -0 -n1 php -l
```

### Admin health page not found
Confirm `servicesbg-core` is active:

```bash
wp plugin is-active servicesbg-core --path=/opt/projects/servicesbg/wp-staging
```

## Next Step After Validation
If Phase 1 validation passes in staging, the next implementation step is migration tooling preflight design and dry-run-only import scaffolding. Production services.bg remains out of scope.
