# servicesbg Plugin Strategy v1 Report

Detailed document: `docs/plugin_strategy_v1.md`

## Status
Completed. No application code or production changes were made.

## Key Decisions
- All ServicesBG functionality must be plugin/module-based.
- Core marketplace logic must be custom.
- Existing plugins may support commodity concerns such as SEO, security, anti-spam, backups, caching, and optional text search.
- External APIs must be accessed through custom adapters.
- Generic directory, booking, CRM, form, and chatbot plugins should not own core ServicesBG workflow data.

## MVP Decision
Build custom ServicesBG marketplace and migration modules. Evaluate existing plugins only where they do not own Flynax traceability, claims, reservations, conversations, leads, coverage zones, or AI knowledge provenance.

