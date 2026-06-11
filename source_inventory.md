# servicesbg Source Inventory

## Source Platform
- Platform: Flynax
- Version: 4.5.2
- Database: services_services
- Table prefix: fl_
- Active template: general_wide

## Core Data Counts
- Accounts: 189
- Categories: 277
- Listings total: 2735
- Listings active: 2640
- Listings expired: 59
- Listings trash: 35
- Listings incomplete: 1

## Core Tables
- fl_accounts
- fl_categories
- fl_listings
- fl_config

## Important Listing Fields
- service_title
- service_about
- service_price_hour
- service_duration
- service_conditions
- service_certificates
- service_provision_location
- phone
- partner_name
- partner_website
- price
- country
- country_level1
- country_level2
- Loc_latitude
- Loc_longitude

## Account Types
- provider_company
- account_freelancer
- visitor_registered

## Active Template
- general_wide

## Media
- Root: files/
- Listing media pattern: files/YYYY-MM/adXXXX/
- Account media: files/account-media/

## Installed / Present Plugins
- autoRegPrevent
- badWordsFilter
- bankWireTransfer
- booking
- bookmarks
- categories_tree
- ccbill
- cookiesPolicy
- dataEntriesImport
- FAQs
- fieldBoundBoxes
- importExportCategories
- ipgeo
- listingPreview
- listings_box
- listings_carousel
- locationFinder
- multiField
- online
- qrCode
- ref
- reportBrokenListing
- similarListings
- sitemap
- socialMetaData
- stripe
- tag_cloud
- testimonials
- underConstructions
- verificationCode

## Target Migration Concept
Flynax service marketplace should migrate into WordPress as:
- Custom post type: service
- Hierarchical service categories
- Provider/company profiles
- Freelancer profiles
- Registered visitors/customers
- Listing media imported into WordPress media library
- Location/geodata preserved
- CRM layer for leads/enquiries
