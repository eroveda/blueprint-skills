# Spec: Campaign Builder and Scheduling

```yaml
title: "Campaign Builder and Scheduling"
purpose: "Implement campaign creation, editing, preview, scheduling, async send pipeline, and retry logic"
category: "CONSTRUCTION"

input_context: |
  From "Contact, Campaign, Automation, and Event Data Models":
  - Campaign table and model class with fields: id, name, subject, from_name, from_email, template_id, status (enum: draft, scheduled, sending, sent), recipient_list_ids, segment_id, scheduled_at, sent_at, created_by, agency_id
  - Campaign status state machine module with allowed transitions: draft to scheduled, scheduled to sending, sending to sent
  - Template table and model class
  - Contact and List tables for recipient resolution
  - Multi-tenant agency_id pattern

  From "Authentication and Role-Based Access":
  - Authentication and authorization middleware
  - Marketer role required for campaign write operations; Viewer role for read-only
  - Agency-scoped query filter
  - Authenticated user context (user_id, agency_id, role)

instructions:
  - "Implement campaign CRUD endpoints: POST /campaigns (create with status 'draft'), GET /campaigns (list with pagination, filterable by status, date range, created_by), GET /campaigns/:id (detail), PUT /campaigns/:id (update — only when status is 'draft'), DELETE /campaigns/:id (only when status is 'draft')"
  - "Implement campaign input validation: name required (max 200 characters), subject required (max 200 characters), from_name required, from_email required and valid email format, template_id must reference an existing template, recipient_list_ids must be a non-empty array of valid list IDs"
  - "Enforce the status state machine on all status changes: use the state machine module from the Data Models spec; reject any transition not in the allowed map and return HTTP 422 with a descriptive error"
  - "Implement campaign preview endpoint: GET /campaigns/:id/preview — render the associated template with merge tags replaced using sample contact data, return the rendered HTML"
  - "Implement campaign schedule endpoint: POST /campaigns/:id/schedule with scheduled_at parameter — validate scheduled_at is in the future, transition status from draft to scheduled, persist scheduled_at"
  - "Implement a scheduled campaign dispatcher: a background job that runs periodically, queries campaigns with status 'scheduled' and scheduled_at in the past, and transitions each to 'sending' then enqueues the send pipeline"
  - "Implement the async send pipeline as a background job: resolve all recipient contacts from recipient_list_ids and optional segment_id, deduplicate contacts across multiple lists, exclude contacts with subscription_status of unsubscribed_global or unsubscribed_list (for the relevant lists), exclude contacts already sent to in this campaign (for retry scenarios), queue individual email send jobs for each remaining recipient"
  - "Create a campaign_sends tracking table (migration): fields id, campaign_id, contact_id, status (enum: queued, sent, failed), error_message (nullable), sent_at (nullable), created_at — with a unique constraint on (campaign_id, contact_id) to prevent duplicate sends"
  - "Implement individual email send job: for each recipient, record in campaign_sends, invoke the email sending interface (abstracted — actual ESP integration is outside this spec), update campaign_sends status to sent or failed"
  - "Implement send completion check: after all individual sends complete, transition the campaign status from sending to sent and record sent_at"
  - "Implement retry endpoint: POST /campaigns/:id/retry — for campaigns with status 'sending' or 'sent' that have failed sends, re-enqueue only the failed sends by checking campaign_sends for status 'failed', without duplicating to contacts with status 'sent'"
  - "Implement campaign statistics summary endpoint: GET /campaigns/:id/stats — return total recipients, sent count, failed count, pending count from campaign_sends"
  - "Apply authentication, authorization, and agency-scoped filtering to all endpoints"
  - "Write unit tests for status transition enforcement, scheduled_at validation, and recipient deduplication logic"
  - "Write integration tests for campaign CRUD, scheduling, the send pipeline (mock email sending), retry logic, and the statistics endpoint"

constraints:
  - "Campaigns can only be edited while in 'draft' status — updating a non-draft campaign returns HTTP 422"
  - "Campaign status transitions must follow: draft to scheduled to sending to sent — no skipping allowed"
  - "Failed sends must be retryable without duplicating to already-sent contacts"
  - "Scheduled campaigns must have scheduled_at set to a future timestamp — past dates are rejected"
  - "Email sending is asynchronous — campaigns with 100,000 contacts must not block the API response"
  - "All timestamps in UTC"

acceptance_criteria:
  - "POST /campaigns creates a campaign with status 'draft' and returns HTTP 201"
  - "PUT /campaigns/:id on a 'scheduled' campaign returns HTTP 422 with error 'Campaign can only be edited in draft status'"
  - "POST /campaigns/:id/schedule with a past date returns HTTP 422"
  - "POST /campaigns/:id/schedule with a future date transitions status to 'scheduled'"
  - "The send pipeline resolves recipients from multiple lists, deduplicates, and excludes unsubscribed contacts"
  - "POST /campaigns/:id/retry re-enqueues only failed sends — contacts with status 'sent' in campaign_sends are not re-sent"
  - "GET /campaigns/:id/stats returns accurate counts matching campaign_sends records"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Contact, Campaign, Automation, and Event Data Models"
    - "Authentication and Role-Based Access"
  integrates_with:
    - "Template Engine"
    - "Contact and List Management"
    - "Billing and Usage Metering"
    - "Notification System"

handoff: |
  Exposes for downstream specs:
  - Campaign service/module with status management for Automation Workflows
  - Campaign send pipeline interface for triggering sends from automations
  - campaign_sends table for Event Tracking to attribute events to campaigns
  - Campaign statistics endpoint for Analytics dashboards
  - Campaign completion events for Notification System

verification: "run project test suite"
```
