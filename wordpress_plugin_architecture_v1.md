# servicesbg WordPress Plugin Architecture v1

## Purpose
This document defines the planned WordPress plugin architecture for the services.bg CRM/service marketplace migration.

It is a design artifact only. No plugin code should be created until the data model and migration plan are approved.

## Plugin Strategy
Build a custom plugin named `servicesbg-core`.

The plugin owns:
- Marketplace post types and taxonomies
- Provider/customer roles and capabilities
- CRM custom tables
- Claim workflow
- Reservation/calendar workflow
- Conversation/messaging workflow
- Coverage zone management
- AI knowledge source records
- Migration WP-CLI commands
- Validation and reporting commands

The plugin does not own:
- Production services.bg
- Source archives
- Source DB dumps
- Generic WordPress core behavior
- Theme-specific presentation beyond templates/hooks required for core workflows

## Architecture Principles
- Keep public content WordPress-native where possible.
- Keep operational state in custom tables.
- Keep all imports idempotent and batch-aware.
- Preserve source IDs in object meta and migration tables.
- Make rollback operate by batch and mapping table.
- Separate migration tooling from runtime marketplace features.
- Require explicit audit records for ownership, claim, reservation, and lead state changes.

## Proposed Directory Structure

```text
servicesbg-core/
  servicesbg-core.php
  includes/
    Plugin.php
    PostTypes/
      ServiceListing.php
    Taxonomies/
      ServiceCategory.php
    Roles/
      Roles.php
      Capabilities.php
    Database/
      Schema.php
      Migrations.php
      Tables.php
    Providers/
      ProviderProfiles.php
    Claims/
      ClaimService.php
      SmsOtpService.php
    Reservations/
      AvailabilityService.php
      ReservationService.php
      CapacityService.php
    Messaging/
      ConversationService.php
      MessageService.php
    Leads/
      LeadService.php
    Coverage/
      CoverageZoneService.php
      CoverageMatcher.php
    Media/
      MediaImportService.php
    SEO/
      RedirectMapService.php
    AI/
      KnowledgeSourceService.php
      AssistantPolicy.php
    Migration/
      SourceDb.php
      Batch.php
      Mapper.php
      Importers/
        CategoryImporter.php
        AccountImporter.php
        ListingImporter.php
        MediaImporter.php
        RedirectImporter.php
        MessageImporter.php
        BookingImporter.php
      Validators/
        CountValidator.php
        RelationshipValidator.php
        MediaValidator.php
        SeoValidator.php
    Admin/
      ClaimsPage.php
      LeadsPage.php
      ReservationsPage.php
      ProviderReviewPage.php
    CLI/
      MigrationCommands.php
      ValidationCommands.php
  templates/
    single-service_listing.php
    archive-service_listing.php
    taxonomy-service_category.php
    claim-listing.php
    provider-dashboard.php
```

This structure is a planning target, not a coding instruction for this phase.

## Runtime Components

### Post Types
`ServiceListing` registers:
- CPT `service_listing`
- Public query/archive support
- REST support if needed for future dashboard/API work
- Meta registration for imported source fields and marketplace fields

### Taxonomies
`ServiceCategory` registers:
- Hierarchical taxonomy `service_category`
- Term meta for Flynax source IDs and redirect data

### Roles
`Roles` registers:
- `service_manager`
- `service_provider`
- `service_freelancer`
- `service_customer`

Capabilities should distinguish:
- Manage own listings
- Manage own coverage
- Manage own availability
- View own leads
- Reply to own conversations
- Request claims
- Review claims as staff
- Manage all marketplace records as staff

### Provider Profiles
`ProviderProfiles` manages:
- Provider/freelancer profile records
- Source account traceability
- Public profile fields
- Phone normalization and verification state
- Linkage to listings, coverage, reservations, and leads

### Claims
`ClaimService` manages:
- Claim creation
- Eligibility checks
- State transitions
- Ownership transfer after verification
- Manual review decisions
- Audit events

`SmsOtpService` manages:
- OTP creation
- OTP hashing
- Expiration
- Attempt limits
- SMS provider adapter boundary

Design decision:
- The SMS provider must be adapter-based so the initial migration is not locked to one vendor.
- No plaintext OTP is stored.

### Reservations
`AvailabilityService` manages provider availability rules.

`ReservationService` manages:
- Reservation requests
- Confirm/reject/cancel/complete states
- Lead and conversation linkage
- Customer contact snapshots

`CapacityService` manages:
- Time overlap checks
- Capacity units
- Listing-specific or provider-wide rules

### Messaging
`ConversationService` manages:
- Thread creation
- Participant resolution
- Lead/reservation linkage
- Conversation status

`MessageService` manages:
- Human messages
- System messages
- AI-generated messages
- Attachments
- Visibility rules

Design decision:
- AI-generated messages must be explicitly marked and auditable.

### Leads CRM
`LeadService` manages:
- Lead creation from inquiries, messages, reservations, and phone/click events where measurable
- Status transitions
- Provider/customer/listing linkage
- Conversion-score fields for future pricing model

### Coverage
`CoverageZoneService` manages:
- Provider/listing coverage zones
- Zone CRUD
- Travel fee and priority fields
- Imported suggestions

`CoverageMatcher` manages:
- Matching customer requested location to provider zones
- Future lead/reservation-based coverage discovery

### AI Knowledge
`KnowledgeSourceService` manages:
- Global platform sources
- Category sources
- Provider-specific sources
- Approval states
- Source provenance

`AssistantPolicy` manages:
- What the assistant may answer
- What requires provider/staff approval
- How AI output is labeled
- Boundaries for private conversation-derived learning

Design decision:
- Raw private conversations are not automatically used as knowledge. Only approved summaries or explicitly approved sources become provider knowledge.

## Migration Components

### SourceDb
Read-only database access to `services_services`.

Responsibilities:
- Connect to source DB.
- Read Flynax tables.
- Never write to source DB.
- Expose typed readers for accounts, categories, listings, and optional plugin tables.

### Batch
Responsibilities:
- Create import batch.
- Store start/end state.
- Store summary JSON.
- Mark failed/rolled back/completed.

### Mapper
Responsibilities:
- Write `wp_servicesbg_migration_map`.
- Resolve source ID to target ID.
- Detect existing imports by checksum and source ID.
- Prevent duplicate imports.

### Importers

| Importer | Responsibility |
| --- | --- |
| `CategoryImporter` | Import `fl_categories` into `service_category`. |
| `AccountImporter` | Import `fl_accounts` into users/provider profiles. |
| `ListingImporter` | Import `fl_listings` into `service_listing`. |
| `MediaImporter` | Import source filesystem media into attachments. |
| `RedirectImporter` | Generate SEO redirect records. |
| `MessageImporter` | Conditional import of Flynax messages after schema audit. |
| `BookingImporter` | Conditional import of Flynax booking data after schema audit. |

### Validators

| Validator | Responsibility |
| --- | --- |
| `CountValidator` | Source vs target counts. |
| `RelationshipValidator` | Owners, categories, listings, profiles, claims. |
| `MediaValidator` | Source paths, checksums, attachments, featured images. |
| `SeoValidator` | Redirect completeness and conflicts. |

## WP-CLI Commands

Planned command namespace: `wp servicesbg`.

Commands:
- `wp servicesbg preflight`
- `wp servicesbg schema install`
- `wp servicesbg import categories --batch=<name>`
- `wp servicesbg import accounts --batch=<name>`
- `wp servicesbg import listings --batch=<name>`
- `wp servicesbg import media --batch=<name>`
- `wp servicesbg import redirects --batch=<name>`
- `wp servicesbg import messages --batch=<name>`
- `wp servicesbg import bookings --batch=<name>`
- `wp servicesbg validate counts --batch=<id>`
- `wp servicesbg validate relationships --batch=<id>`
- `wp servicesbg validate media --batch=<id>`
- `wp servicesbg validate seo --batch=<id>`
- `wp servicesbg rollback --batch=<id> --dry-run`
- `wp servicesbg rollback --batch=<id> --confirm`
- `wp servicesbg report latest`

Command requirements:
- Default to dry-run where destructive changes are possible.
- Print batch IDs and report paths.
- Fail clearly if source counts drift unexpectedly.
- Never write to source DB.

## Custom Tables

Plugin-owned tables:
- `wp_servicesbg_import_batches`
- `wp_servicesbg_migration_map`
- `wp_servicesbg_provider_profiles`
- `wp_servicesbg_claims`
- `wp_servicesbg_availability_rules`
- `wp_servicesbg_reservations`
- `wp_servicesbg_conversations`
- `wp_servicesbg_messages`
- `wp_servicesbg_leads`
- `wp_servicesbg_coverage_zones`
- `wp_servicesbg_media_map`
- `wp_servicesbg_redirect_map`
- `wp_servicesbg_ai_knowledge_sources`
- `wp_servicesbg_ai_knowledge_chunks`
- `wp_servicesbg_ai_feedback`
- `wp_servicesbg_audit_log`

Schema rules:
- Every table has integer primary key.
- Every operational table has timestamps.
- Every migrated table has source ID columns where relevant.
- Every stateful table supports status and audit logging.
- Foreign-key-like relationships should be indexed even if MySQL foreign keys are not used because of WordPress compatibility.

## Admin Interfaces

Initial staff/admin screens:
- Claim review queue
- Imported listing review queue
- Provider profile review queue
- Lead inbox
- Reservation calendar
- Coverage zone review
- Migration batch reports
- Redirect conflicts

Provider dashboard screens:
- Profile
- Listings
- Claim requests
- Availability/calendar
- Reservations
- Messages
- Leads
- Coverage zones
- AI knowledge/FAQ sources

## Frontend Routes

Required public/user-facing routes:
- Service listing detail
- Service category archive
- Provider profile
- Claim listing
- Reservation request
- Inquiry/message form
- Customer conversation view
- Provider dashboard

Routing can use WordPress templates initially; deeper frontend work should be deferred until core data/workflows are stable.

## Security and Privacy
- Use WordPress nonces and capability checks for all dashboard/admin actions.
- Providers can access only their own listings, leads, reservations, conversations, coverage zones, and AI sources.
- Staff roles require explicit marketplace capabilities.
- OTPs are hashed and expire.
- Rate-limit claim attempts.
- Store imported contact data with source provenance.
- Do not use private conversations as AI knowledge without approval.
- Treat source data imports as staging until final privacy/business review.

## Reversibility and Idempotency
- All importers use batch IDs.
- All source-to-target relationships are recorded in `wp_servicesbg_migration_map`.
- Importers check existing map records before creating new objects.
- Rollback operates from migration map.
- Claimed, edited, or customer-touched records are protected from destructive rollback.
- Media rollback checks whether an attachment is reused by another batch/object.

## Testing Strategy
- Unit tests for normalizers, mappers, status transitions, and capacity checks.
- Integration tests for CPT/taxonomy registration and custom table schema.
- WP-CLI dry-run tests for import commands.
- Fixture-based importer tests from sampled Flynax rows.
- Validation command tests for count and relationship mismatches.
- Manual staging QA for claim, reservation, messaging, and provider dashboard flows.

## Phased Build Sequence
1. Schema, roles, CPT, taxonomy.
2. Migration batch and mapping infrastructure.
3. Category/account/listing importers.
4. Validation commands.
5. Media importer.
6. Claim seed and runtime claim workflow.
7. Provider profiles and dashboard basics.
8. Leads and messaging.
9. Reservations and availability.
10. Coverage zones.
11. AI knowledge source records and assistant boundaries.
12. SEO redirect export/handler.

## Open Decisions Before Coding
- Confirm exact source primary keys and relation columns.
- Confirm whether historical messages, bookings, and reviews should be imported.
- Confirm public URL structure and slug policy.
- Choose SMS provider adapter.
- Decide initial default availability policy for imported providers.
- Decide whether non-active Flynax listings should be private, draft, or archived custom status.
- Decide whether reviews/testimonials are in scope for first migration.

