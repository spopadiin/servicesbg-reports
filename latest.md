# ServicesBG Latest Status

Updated: 2026-07-01T23:45:00+03:00

Status: `phase3c3b_listing_fidelity_corrected`

Scope: staging listing fidelity audit and correction for source listing `2780`.

Production modified: no.

Generated:
- `docs/phase3c3b_listing_fidelity_audit_v1.md`
- `reports/listing_field_comparison.md`
- `reports/listing_media_comparison.md`
- `reports/listing_taxonomy_comparison.md`
- `scripts/validate_listing_fidelity.sh`
- `reports/latest.md`
- `reports/latest.json`

Summary:
- Corrected the audited staging listing to prefer Bulgarian public labels and display values.
- Restored missing public fields: `Телефон за поръчка`, `Цената е`, `Минимално продължение`, and `Място на изпълнение`.
- Corrected taxonomy labels for `Здраве и красота` and `Масажи` while preserving source slugs/IDs.
- Imported the listing's three mapped local images as staging WP attachments, set the featured image, and preserved gallery order.
- Updated the listing detail view model and renderer to expose the corrected data without UI redesign.
- No production changes, external APIs, CSS redesign, or theme redesign.

Validation:
- `./scripts/validate_listing_fidelity.sh` validates source DB, staging storage, view model, rendered output, taxonomy, media map, attachments, featured image, and gallery order.

Next step:
- Continue fidelity audits on additional golden listings before any UI redesign.
