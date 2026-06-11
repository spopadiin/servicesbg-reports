# servicesbg Flynax to WordPress Mapping v1 Report

Detailed document: `docs/flynax_to_wp_mapping_v1.md`

## Core Mapping
- `fl_accounts` -> WordPress users and `wp_servicesbg_provider_profiles`
- `fl_account_types` -> role/capability mapping reference
- `fl_categories` -> hierarchical taxonomy `service_category`
- `fl_listings` -> CPT `service_listing`
- `fl_listing_photos` -> WordPress attachments plus `wp_servicesbg_media_map`
- `fl_listing_video` -> listing video metadata
- `fl_messages` -> conversations/messages if approved
- `fl_contacts` -> CRM leads/contact archive if approved
- booking tables -> reservation reference only; rebuild target reservation system
- `fl_data_formats` and `fl_geo_mapping` -> location lookup and coverage suggestion reference
- `fl_languages` and selected `fl_lang_keys` -> label/content reference
- `fl_pages`, category paths, listing paths -> redirect map inputs
- plan/payment tables -> restricted audit/reference if approved
- `fl_plugins` -> migration decision reference

## Implementation Gate
Review and approve `docs/source_schema_audit_v1.md` and `docs/flynax_to_wp_mapping_v1.md` before writing migration/plugin code.

