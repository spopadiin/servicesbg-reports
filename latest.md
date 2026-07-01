# ServicesBG Latest Status

Updated: 2026-07-02T01:10:00+03:00

Status: `documentation_reports_restructured`

Scope: repository documentation, reports, audits, QA evidence, plugin docs, ADRs, and GitHub report publishing hygiene.

Production modified: no.

Runtime behavior changed: no.

Generated/updated:
- `docs/README.md`
- `docs/adr/ADR-0001-plugin-architecture.md`
- `docs/adr/ADR-0002-platform-event-bus.md`
- `docs/adr/ADR-0003-workflow-engine.md`
- `docs/adr/ADR-0004-job-handler-contracts.md`
- `docs/adr/ADR-0005-view-model-theme-boundary.md`
- `docs/adr/ADR-0006-staging-first-migration.md`
- `docs/adr/ADR-0007-future-headless-nextjs.md`
- `reports/README.md`
- `reports/history_index.md`
- `reports/history_manifest.json`
- `audits/README.md`
- `qa/README.md`
- `qa/issues/README.md`
- `plugins/*/{README.md,ARCHITECTURE.md,API.md,WORKFLOWS.md,CHANGELOG.md}`
- `scripts/validate_documentation_structure.sh`
- `scripts/publish_reports.sh`
- `reports/latest.md`
- `reports/latest.json`

Summary:
- Added canonical documentation folders for architecture, migration, UI, integrations, reference, and ADRs.
- Added phase report folders under `reports/phase1` through `reports/phase4`.
- Copied listing/media/taxonomy/migration evidence into `audits/`.
- Copied golden listing QA materials into `qa/golden_listings/`.
- Added plugin documentation indexes for every implemented plugin.
- Updated report publishing to mirror nested report folders while protecting `.git` metadata and avoiding owner/group/permission preservation.
- Preserved historical root-level reports; no historical report was deleted.

Validation:
- `./scripts/validate_documentation_structure.sh`
- `./scripts/publish_reports.sh`

Next step:
- Keep new phase reports, audits, QA evidence, and ADRs in the new canonical folders.
