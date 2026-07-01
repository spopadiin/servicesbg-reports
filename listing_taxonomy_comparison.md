# Listing Taxonomy Comparison

Listing: source `fl_listings.ID=2780`, staging `service_listing` post `3004`.

| Legacy taxonomy item | Source DB value | Migration destination | View model | Theme output | Status |
| --- | --- | --- | --- | --- | --- |
| Parent category | `fl_categories.ID=2226`, `Key=services_health_services`, `Path=zdravie-i-krasota-health-beauty-services` | `service_category` term `69`, source meta `_servicesbg_source_category_id=2226` | breadcrumb ancestor | `Здраве и красота` | OK after label correction |
| Listing category | `fl_categories.ID=2239`, `Key=services_services_health_services_massages`, `Path=zdravie-i-krasota-health-beauty-services/masazhi-massages` | `service_category` term `286`, source meta `_servicesbg_source_category_id=2239` | `category.label=Масажи` | `Масажи` | OK after label correction |
| Category hierarchy | `Parent_ID=2226`, `Level=1` | term `286` parent term `69` | breadcrumb/category hierarchy | parent then child | OK |
| Public Bulgarian parent label | `fl_lang_keys: categories+name+services_health_services = Здраве и красота` | term name | visible label | visible label | OK |
| Public Bulgarian child label | `fl_lang_keys: categories+name+services_services_health_services_massages = Масажи` | term name | visible label | visible label | OK |
| Legacy Latin path/slug | `zdravie-i-krasota-health-beauty-services/masazhi-massages` | term slug remains Latin compatibility slug with source ID suffix | URL/term link | URL only | OK |

## Finding
The staging term labels had been normalized from source keys instead of Bulgarian language labels. This was a taxonomy migration/data normalization error for the existing staging terms, not a theme redesign issue.

## Correction
Staging term labels for source category IDs `2226` and `2239` were updated to Bulgarian while preserving source IDs, slugs, paths, and hierarchy.
