# Spec: Contact, Campaign, Automation, and Event Data Models

```yaml
title: "Contact, Campaign, Automation, and Event Data Models"
purpose: "Define all entity schemas, relationships, indexes, and the campaign status state machine"
category: "DESIGN"

input_context: |
  From "Project Bootstrap and Environment Setup":
  - Project directory structure and module organization conventions
  - Database connection and migration tooling
  - Test runner configuration and conventions

instructions:
  - "Create a migration for the Contact entity with fields: id (primary key), email (unique per agency), first_name, last_name, tags (array/JSON), subscription_status (enum: subscribed, unsubscribed_list, unsubscribed_global, bounced), custom_fields (JSON), agency_id (foreign key for multi-tenancy), created_at (UTC), updated_at (UTC)"
  - "Create a migration for the List entity with fields: id, name, description, agency_id, created_at; and a join table contact_list_memberships for the many-to-many relationship between Contact and List, with a unique constraint on (contact_id, list_id)"
  - "Create a migration for the Segment entity with fields: id, name, filter_rules (JSON — stores conditions like field, operator, value arrays), agency_id, created_at"
  - "Create a migration for the Template entity with fields: id, name, html_content (text), merge_tags (array/JSON — list of variable placeholders), thumbnail_url, agency_id, created_at, updated_at"
  - "Create a migration for the Campaign entity with fields: id, name, subject, from_name, from_email, template_id (foreign key to Template), status (enum: draft, scheduled, sending, sent), recipient_list_ids (array/JSON), segment_id (nullable foreign key to Segment), scheduled_at (nullable, UTC), sent_at (nullable, UTC), created_by (foreign key to User), agency_id, created_at"
  - "Create a migration for the Automation entity with fields: id, name, trigger_type (enum: contact_added_to_list, tag_applied, date_field_reached, manual), trigger_config (JSON), status (enum: active, paused, draft), paused_at (nullable, UTC), agency_id, created_at"
  - "Create a migration for the AutomationStep entity with fields: id, automation_id (foreign key to Automation), order (integer), action_type (enum: send_email, wait, add_tag, remove_tag, move_to_list), action_config (JSON), delay_duration (interval/integer in seconds); with a unique constraint on (automation_id, order)"
  - "Create a migration for the Event entity with fields: id, contact_id (foreign key), campaign_id (nullable foreign key), automation_id (nullable foreign key), type (enum: open, click, bounce_hard, bounce_soft, unsubscribe, conversion), metadata (JSON — stores URL, device, client info), timestamp (UTC); partitioned or indexed by timestamp for query performance"
  - "Create a migration for the BillingPlan entity with fields: id, name, contact_limit (integer), email_send_limit (integer), price (decimal), features (JSON), created_at"
  - "Create a migration for the UsageRecord entity with fields: id, agency_id (foreign key), billing_period (date range or month identifier), contacts_used (integer), emails_sent (integer), created_at"
  - "Add database indexes: Contact.email + agency_id (unique), Contact.agency_id, Campaign.status + agency_id, Campaign.agency_id, Event.campaign_id + type, Event.contact_id, Event.timestamp, Automation.agency_id + status, UsageRecord.agency_id + billing_period"
  - "Implement the campaign status state machine as a validation module: define allowed transitions (draft to scheduled, scheduled to sending, sending to sent), reject any transition not in the allowed map, and expose a transition function that validates and updates status"
  - "Create model/entity classes or modules for each table with basic validation rules (email format on Contact, required fields, enum constraints)"
  - "Write unit tests for the campaign status state machine covering all valid transitions and all invalid transition attempts"
  - "Write migration tests that verify all tables can be created and rolled back cleanly"

constraints:
  - "All timestamps stored in UTC with timezone-aware column types"
  - "Campaign status transitions must follow the exact sequence: draft to scheduled to sending to sent — no skipping allowed"
  - "Every entity must include agency_id for multi-tenant data isolation"
  - "Email uniqueness is scoped per agency, not globally"
  - "Event table must be optimized for high-volume inserts and time-range queries"

acceptance_criteria:
  - "All migrations run successfully and create the expected tables with correct column types"
  - "All migrations can be rolled back without errors"
  - "Campaign status state machine allows draft to scheduled transition"
  - "Campaign status state machine rejects draft to sending transition (must go through scheduled)"
  - "Campaign status state machine rejects sent to draft transition (no reverse)"
  - "Contact email uniqueness is enforced per agency — same email in different agencies is allowed"
  - "Event table has indexes on campaign_id + type and on timestamp"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Project Bootstrap and Environment Setup"
  integrates_with: []

handoff: |
  Exposes for downstream specs:
  - Contact, List, and contact_list_memberships tables and model classes
  - Segment table and model class with filter_rules JSON structure
  - Campaign table, model class, and status state machine module
  - Template table and model class with merge_tags structure
  - Automation and AutomationStep tables and model classes
  - Event table and model class optimized for high-volume writes
  - BillingPlan and UsageRecord tables and model classes
  - All entity validation rules and enum definitions
  - Multi-tenant agency_id pattern for data isolation

verification: "run project test suite"
```
