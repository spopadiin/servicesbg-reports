# Listing Field Comparison

Listing: source `fl_listings.ID=2780`, staging `service_listing` post `3004`.

| Legacy field | Source DB value | Migration destination | View model | Theme output | Status |
| --- | --- | --- | --- | --- | --- |
| Title | `service_title=Студио за професионални масажи Варна` | `wp_posts.post_title` | `listing_detail_header.data.title` | H1 | OK |
| Description | `service_about` Bulgarian text | `wp_posts.post_content` | `listing_detail_body` field `service_about` | Description section | OK |
| Category | `Category_ID=2239` | `service_category` term, source meta `_servicesbg_source_category_id=2239` | `category.label=Масажи` | Header/meta and breadcrumbs | OK after taxonomy correction |
| Parent category | source category `2226` | parent `service_category` term | breadcrumb/category hierarchy | Breadcrumb/category navigation | OK after taxonomy correction |
| Price | `price=30|currency_lev` | `_servicesbg_price=30|currency_lev`, `_servicesbg_price_display=30 лв.` | `price=30 лв.` | Header and detail field | OK after normalization correction |
| Цената е | `price_type=4`, label `средна цена` | `_servicesbg_price_type=4`, `_servicesbg_price_type_display=средна цена` | detail field `price_type` | `Цената е: средна цена` | OK after view-model correction |
| Минимално продължение | `service_duration=30` | `_servicesbg_service_duration=30`, `_servicesbg_service_duration_display=30 мин.` | detail field `service_duration` | `Минимално продължение...: 30 мин.` | OK after view-model correction |
| Място на изпълнение | `service_provision_location=1`, label `На адрес на доставчика` | `_servicesbg_service_provision_location=1`, `_servicesbg_service_provision_location_display=На адрес на доставчика` | detail field `service_provision_location` | `Място на изпълнение: На адрес на доставчика` | OK after view-model correction |
| Телефон за поръчка | `phone=a:0882|n:360084` | `_servicesbg_phone_raw`, `_servicesbg_phone_normalized=0882360084`, claim seed phone | detail field `phone=0882 360084` | `Телефон за поръчка: 0882 360084` | OK after view-model correction |
| Reference number | `ref_number=RF192059` | `_servicesbg_ref_number=RF192059` | detail field `reference` | `Референтен номер: RF192059` | OK |
| Coordinates | `Loc_latitude=42.733883`, `Loc_longitude=25.48583` | `_servicesbg_location_json` | `location.latitude`, `location.longitude` | not currently visible as coordinates | OK, intentionally not rendered as raw coordinates |
| Address text | embedded in `service_about`: `гр. Варна, ул. Ген. Киселов 27` | `wp_posts.post_content` | description field | visible in description | OK |
| Conditions | empty | `_servicesbg_service_conditions` empty | omitted because empty | omitted | OK |
| Certificates | empty | `_servicesbg_service_certificates` empty | omitted because empty | omitted | OK |
| Partner name | empty | `_servicesbg_partner_name` empty | not shown | not shown | OK |
| Partner website | empty | `_servicesbg_partner_website` empty | not shown | not shown | OK |

No intentionally deprecated visible legacy field was identified for this listing.
