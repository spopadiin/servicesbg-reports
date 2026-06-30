# Staging QA Access Package

Updated: 2026-06-30T10:41:22+03:00

Security warning: these credentials are staging-only temporary QA credentials for `https://services.herbal.bg`. Rotate or delete these QA accounts and passwords before any production migration or cutover. Do not publish these credentials publicly. No real production credentials are included.

Login URL:
`https://services.herbal.bg/wp-login.php`

Provider dashboard URL:
`https://services.herbal.bg/dashboard/`

## Accounts

| Purpose | Username | Temporary password | Role/capabilities | Linked provider/listing references | Dashboard URL |
| --- | --- | --- | --- | --- | --- |
| Super admin QA account | `qa-super-admin` | `goLAjBMOE8NAC4eUG2YHaQhf` | `administrator` | Staging QA admin user id `297`; no migrated provider link. | `https://services.herbal.bg/wp-admin/` |
| Company provider QA account | `qa-company-provider` | `iDFNxOcjK0FHTtNg9EqJUkKH` | `service_provider` | QA wrapper user id `296`; linked migrated provider user id `163`; source Flynax account id `85`; selected listing id `2578`; source listing id `2347`. Migrated provider password was not changed. | `https://services.herbal.bg/dashboard/` |
| Freelancer provider QA account | `qa-freelancer-provider` | `sTbaz4b8Pr3YIeNWHepNWzb8` | `service_freelancer` | QA wrapper user id `299`; linked migrated provider user id `112`; source Flynax account id `19`; selected listing id `1926`; source listing id `1648`. Migrated provider password was not changed. | `https://services.herbal.bg/dashboard/` |
| Customer/test client QA account | `qa-customer` | `pgVgiDms/Yz2dZw7dI9evH/+` | `service_customer` | Staging QA customer user id `298`; no migrated provider link. | `https://services.herbal.bg/` |

Optional CSR/admin support QA account:
- Not created. Current staging roles are `administrator`, `editor`, `author`, `contributor`, `subscriber`, `service_provider`, `service_freelancer`, and `service_customer`; no dedicated CSR/support role exists.

## Golden Listing URLs

- Most complete listing: `https://services.herbal.bg/?service_listing=%d0%ba%d0%be%d0%bd%d1%82%d0%b5%d0%b9%d0%bd%d0%b5%d1%80%d0%b8-%d0%b7%d0%b0-%d1%81%d1%82%d1%80%d0%be%d0%b8%d1%82%d0%b5%d0%bb%d0%bd%d0%b8-%d0%be%d1%82%d0%bf%d0%b0%d0%b4%d1%8a%d1%86%d0%b8-%d0%bf%d0%bb`
- Best company provider listing: `https://services.herbal.bg/?service_listing=%d1%80%d0%b5%d0%bc%d0%be%d0%bd%d1%82-%d0%bd%d0%b0-%d0%b4%d0%b8%d0%b7%d0%b5%d0%bb%d0%be%d0%b2%d0%b8-%d0%ba%d0%be%d0%bc%d1%8a%d0%bd-%d1%80%d0%b5%d0%b9%d0%bb-%d0%b4%d1%8e%d0%b7%d0%b8-%d0%b8-%d0%b3%d0%bd`
- Best freelancer listing: `https://services.herbal.bg/?service_listing=%d0%b3%d0%be%d0%b1%d0%bb%d0%b5%d0%bd%d0%b8-%d0%bf%d0%be-%d0%bf%d0%be%d1%80%d1%8a%d1%87%d0%ba%d0%b0-%d0%bf%d0%bb%d0%be%d0%b2%d0%b4%d0%b8%d0%b2`
- Most media-rich listing: `https://services.herbal.bg/?service_listing=bg%d1%84%d0%b0%d1%8f%d0%bd%d1%81-%d1%82%d0%b5%d1%80%d0%b0%d0%ba%d0%be%d1%82-%d1%80%d0%b5%d0%b4%d0%b5%d0%bd%d0%b5-bg`
- Listing with video metadata: `https://services.herbal.bg/?service_listing=bg%d0%b1%d0%be%d1%8f%d0%b4%d0%b8%d1%81%d0%b2%d0%b0%d0%bd%d0%b5-%d1%81-%d0%bb%d0%b0%d1%82%d0%b5%d0%ba%d1%81-bg`
- Richest specification/theme regression listing: `https://services.herbal.bg/?service_listing=%d0%b7%d0%b0%d0%bd%d0%b8%d0%bc%d0%b0%d0%bb%d0%bd%d1%8f-%d0%be%d0%b1%d1%80%d0%b0%d0%b7%d0%be%d0%b2%d0%b0%d1%82%d0%b5%d0%bb%d0%b5%d0%bd-%d1%86%d0%b5%d0%bd%d1%82%d1%8a%d1%80`

## Recommended QA Flow

1. Public homepage: `https://services.herbal.bg/`
2. Category page: open a selected listing category from `reports/qa_golden_listings.md`.
3. Search: `https://services.herbal.bg/?s=%D1%80%D0%B5%D0%BC%D0%BE%D0%BD%D1%82`
4. Listing detail: start with the primary regression listing.
5. Provider profile: use the listing provider links, for example `https://services.herbal.bg/?author=163` and `https://services.herbal.bg/?author=112`.
6. Claim CTA: verify claim state on selected listings.
7. Reservation CTA: verify disabled/unavailable state because no active availability rules exist in staging.
8. Messaging/contact action: verify contact/messaging UI state and no external messages are sent.
9. Provider dashboard: log in as `qa-company-provider` and `qa-freelancer-provider`, then open `https://services.herbal.bg/dashboard/`.
10. Reviews/AI summary areas: verify empty/unavailable summary state; no external LLM/API calls should be made.

## Validation Checklist

- QA accounts can log in with the temporary passwords above.
- Provider QA accounts can access the provider dashboard shell.
- Customer account can access customer-facing public pages.
- Golden listing URLs resolve to published staging listings.
- Theme renders selected listing detail pages.
- No production changes were made.

