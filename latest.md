# ServicesBG Latest Report

Updated: 2026-06-27T19:25:22+03:00

## Status
Phase 3B-0 legacy UX mapping and theme blueprint is complete as documentation only.

This phase analyzed the active legacy Flynax `general_wide` frontend, the Phase 3A migration constraints, source schema audit, Flynax-to-WordPress mapping, current plugin boundaries, and the preserved archive under `/opt/projects/servicesbg/archive/extracted/servicebg`.

No WordPress theme, frontend code, CSS, JavaScript, assets, plugin behavior, migration code, or production changes were implemented.

## Deliverables
- `docs/phase3b_theme_blueprint_v1.md`
- `docs/phase3b_component_library_v1.md`
- `docs/phase3b_navigation_sitemap_v1.md`
- `reports/latest.md`
- `reports/latest.json`

## Findings
- Legacy Services.bg is search-led, with a utility/header row, primary menu, prominent add-listing CTA, horizontal homepage search, category browse pages, list/grid/map results, listing gallery/detail pages, seller/contact sidebars, mobile map/list/search toggles, and footer menu/social/RSS structure.
- The familiar information architecture should be preserved: homepage search, service categories, category archives, listing details, provider profiles, add-service/provider account flows, breadcrumbs, and footer service/provider/support/legal groupings.
- The new WordPress theme must remain presentation-only. Listings, providers, search, claims, coverage, reservations, messaging, CRM, reviews, AI review summaries, integrations, migration maps, and platform workflow state stay in plugins.
- Legacy taxonomy hierarchy and field grouping should inform presentation, but WordPress must use plugin-provided view models and migrated `service_category`/`service_listing` data.

## Blueprint Coverage
- Public sitemap
- Navigation hierarchy
- Homepage layout
- Category pages
- Listing cards
- Listing detail page
- Provider profile page
- Search/filter layout
- Mobile navigation
- Breadcrumb strategy
- Footer structure
- Existing taxonomy presentation
- Existing field grouping
- Existing forms
- Component library for header, footer, mega menu, search bar, listing card, provider card, review summary widget, reservation widget, claim widget, coverage widget, messaging indicator, dashboard cards, tables, forms, modals, alerts, timeline, and workflow status.

## Architecture Rules Preserved
- No legacy HTML/CSS/assets copied.
- No theme implementation performed.
- Business logic remains in plugins.
- Theme is defined as presentation only.
- Source ID traceability and migration behavior remain in `servicesbg-migration`.
- Search behavior remains in `servicesbg-search`.
- Claim workflow remains in `servicesbg-claims`.
- Coverage workflow remains in `servicesbg-coverage`.
- Reservation workflow remains in `servicesbg-reservations`.
- Messaging and CRM workflows remain in `servicesbg-messaging` and `servicesbg-crm`.
- Review summary public output remains in `servicesbg-reviews` and `servicesbg-ai-reviews`.

## Explicitly Not Done
- no WordPress theme files
- no CSS or JavaScript
- no component implementation
- no copied Flynax assets
- no route/rewrite changes
- no redirects activated
- no plugin logic changes
- no migration changes
- no production changes

## Next Step
Before Phase 3B implementation, define plugin view-model/template APIs for listing cards, provider cards, filters, claim status, coverage summaries, reservation affordances, review summaries, messaging/contact actions, and dashboard summaries.
