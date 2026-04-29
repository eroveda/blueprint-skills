# Spec: Contact and List Management

```yaml
title: "Contact and List Management"
purpose: "Implement CRUD for contacts and lists, import/export, tagging, segmentation, and unsubscribe processing"
category: "CONSTRUCTION"

input_context: |
  From "Contact, Campaign, Automation, and Event Data Models":
  - Contact, List, contact_list_memberships, and Segment tables and model classes
  - Contact fields: id, email, first_name, last_name, tags, subscription_status, custom_fields, agency_id
  - List fields: id, name, description, agency_id
  - Segment fields: id, name, filter_rules (JSON with field/operator/value conditions)
  - Multi-tenant agency_id pattern for data isolation

  From "Authentication and Role-Based Access":
  - Authentication middleware for protecting endpoints
  - Authorization middleware: Marketer can read/write; Viewer has read-only access
  - Agency-scoped query filter restricting results to the authenticated agency
  - Authenticated user context (user_id, agency_id, role)

instructions:
  - "Implement contact CRUD endpoints: POST /contacts (create), GET /contacts (list with pagination, filtering by tag, list, subscription status), GET /contacts/:id (detail), PUT /contacts/:id (update), DELETE /contacts/:id (soft delete or archive)"
  - "Implement contact input validation: email format required, email unique within agency, first_name and last_name optional, custom_fields must be a valid key-value map"
  - "Implement list CRUD endpoints: POST /lists (create), GET /lists (list with pagination), GET /lists/:id (detail with contact count), PUT /lists/:id (update), DELETE /lists/:id (remove list and memberships)"
  - "Implement contact-to-list membership endpoints: POST /lists/:id/contacts (add contacts by ID array), DELETE /lists/:id/contacts (remove contacts by ID array), GET /lists/:id/contacts (list contacts in a list with pagination)"
  - "Implement CSV/file-based contact import: accept uploaded file, validate each row (email format, deduplication against existing contacts in the agency), create new contacts and skip or update duplicates based on a user-specified strategy (skip, update, or error), return an import summary with counts (created, updated, skipped, errors)"
  - "Implement tag management: PUT /contacts/:id/tags (set tags array), POST /contacts/:id/tags (add tags), DELETE /contacts/:id/tags (remove specific tags); tags are freeform strings stored as an array on the contact"
  - "Implement bulk operations endpoint: POST /contacts/bulk with action parameter — supports bulk tag application, bulk list assignment, and bulk deletion for a provided array of contact IDs (max 1000 per request)"
  - "Implement segment CRUD endpoints: POST /segments (create with filter_rules), GET /segments (list), GET /segments/:id (detail); filter_rules support operators: equals, not_equals, contains, greater_than, less_than, is_set, is_not_set applied to contact fields and custom_fields"
  - "Implement segment evaluation endpoint: GET /segments/:id/contacts — dynamically query contacts matching the filter_rules and return paginated results with the evaluated count"
  - "Implement unsubscribe processing: POST /contacts/:id/unsubscribe with scope parameter (list_id for per-list unsubscribe, or 'global' for global unsubscribe); per-list removes the contact from that list; global sets subscription_status to unsubscribed_global and removes from all lists"
  - "Implement contact export endpoint: GET /contacts/export — generate a CSV file of contacts matching optional filter parameters (list, tag, segment, subscription status) and return it as a downloadable file"
  - "Apply authentication and authorization middleware to all endpoints: require authenticated user, enforce Marketer role for write operations, allow Viewer role for read operations"
  - "Apply agency-scoped query filter to all database queries to enforce multi-tenant isolation"
  - "Write unit tests for contact validation, segment filter evaluation logic, and unsubscribe processing"
  - "Write integration tests for contact CRUD, list membership, CSV import (valid file, file with duplicates, file with invalid rows), bulk operations, and export"

constraints:
  - "Contacts can unsubscribe from specific lists or globally — per the business rules"
  - "Bulk operations are limited to 1000 contacts per request to prevent timeouts"
  - "CSV import must handle files with up to 100,000 rows without blocking the API response — use the background job queue for large imports and return a job ID"
  - "Contact email uniqueness is scoped per agency, not globally"
  - "Segment filter evaluation must be performed as a database query, not in-memory, to handle large contact sets"
  - "Viewer role can only read contacts and lists — all write operations must return HTTP 403 for Viewers"

acceptance_criteria:
  - "POST /contacts with valid data returns HTTP 201 with the created contact"
  - "POST /contacts with a duplicate email within the same agency returns HTTP 409"
  - "GET /contacts returns paginated results filtered to the authenticated agency only"
  - "CSV import with 100 valid rows creates 100 contacts and returns a summary with created: 100"
  - "CSV import with duplicate emails (skip strategy) skips existing contacts and reports the count"
  - "POST /contacts/:id/unsubscribe with scope 'global' sets subscription_status to unsubscribed_global"
  - "GET /segments/:id/contacts returns only contacts matching all filter_rules conditions"
  - "Viewer role can GET /contacts but receives HTTP 403 on POST /contacts"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Contact, Campaign, Automation, and Event Data Models"
    - "Authentication and Role-Based Access"
  integrates_with:
    - "Automation Workflows"
    - "Event Tracking and Analytics"

handoff: |
  Exposes for downstream specs:
  - Contact CRUD service/module for querying and updating contacts
  - List membership service for adding/removing contacts from lists
  - Tag management service for applying and removing tags
  - Segment evaluation service for resolving contacts matching filter criteria
  - Unsubscribe processing service for handling opt-outs
  - Contact query interface for recipient resolution in campaign sending

verification: "run project test suite"
```
