# Spec: Notification System

```yaml
title: "Notification System"
purpose: "Deliver in-app notifications, webhook dispatches, billing alerts, and bounce/complaint alerts"
category: "CONSTRUCTION"

input_context: |
  From "Contact, Campaign, Automation, and Event Data Models":
  - Event model for subscribing to event types (open, click, bounce, unsubscribe, conversion)
  - Campaign model with status transitions for detecting completion and failures
  - Automation model with status and enrollment tracking
  - Multi-tenant agency_id pattern

  From "Campaign Builder and Scheduling":
  - Campaign completion events (status transitions to 'sent')
  - Campaign send failure events (individual sends with status 'failed')
  - campaign_sends table for querying failure counts

  From "Automation Workflows":
  - Automation enrollment events and step execution events
  - Automation pause/resume events
  - automation_enrollments table for tracking execution status

instructions:
  - "Create a notifications table (migration): fields id, agency_id, user_id (nullable — null means all users in the agency), type (enum: campaign_completed, campaign_failed, automation_triggered, billing_threshold, billing_limit_reached, bounce_alert, complaint_alert), title, body (text), metadata (JSON — contextual data like campaign_id, automation_id, usage percentage), is_read (boolean, default false), created_at (UTC)"
  - "Create a webhook_endpoints table (migration): fields id, agency_id, url, secret (for signing payloads), event_types (array/JSON — list of event types to subscribe to), is_active (boolean), created_by, created_at, updated_at"
  - "Create a webhook_deliveries table (migration): fields id, webhook_endpoint_id, event_type, payload (JSON), response_status (integer, nullable), response_body (text, nullable), status (enum: pending, delivered, failed), attempts (integer, default 0), next_retry_at (nullable, UTC), created_at"
  - "Implement in-app notification endpoints: GET /notifications (list for authenticated user with pagination, filterable by type and is_read), PUT /notifications/:id/read (mark as read), POST /notifications/read-all (mark all as read), GET /notifications/unread-count (return count of unread notifications)"
  - "Implement notification creation service: a central function that creates notification records and is called by other services when notable events occur — campaign completion, send failures, automation triggers, billing thresholds"
  - "Implement campaign completion notification: when a campaign transitions to 'sent', create a notification for the campaign creator with the campaign name, total sent count, and failure count"
  - "Implement campaign failure notification: when a campaign has more than 5% failed sends after completion, create a high-priority notification for the campaign creator and all Admin users in the agency"
  - "Implement automation trigger notification: when an automation enrollment begins, create a notification for the automation creator with the contact name and automation name"
  - "Implement billing threshold notifications: consume billing threshold alert events from the Billing and Usage Metering spec — create notifications when usage reaches 80% (warning) and 95% (critical) of contact or email limits, addressed to all Admin users in the agency"
  - "Implement bounce/complaint alert: when hard bounce count for a campaign exceeds 2% of total sends, or when multiple complaints are received, create a notification recommending list hygiene review"
  - "Implement webhook management endpoints (Admin only): POST /webhooks (register endpoint with URL, secret, and event_types), GET /webhooks (list registered endpoints), PUT /webhooks/:id (update), DELETE /webhooks/:id (delete), POST /webhooks/:id/test (send a test payload)"
  - "Implement webhook dispatch service: when a subscribed event occurs, serialize the event payload, sign it with the endpoint's secret using HMAC, enqueue delivery as a background job"
  - "Implement webhook delivery with retry: attempt delivery via HTTP POST to the registered URL, record response_status and response_body; on failure (non-2xx response or timeout), retry up to 5 times with exponential backoff (1 min, 5 min, 30 min, 2 hours, 12 hours); update status to 'delivered' or 'failed'"
  - "Apply authentication, authorization, and agency-scoped filtering: Admin for webhook management; all authenticated users receive notifications scoped to their user_id or agency"
  - "Write unit tests for notification creation logic, webhook payload signing, and retry backoff calculation"
  - "Write integration tests for notification CRUD, mark-as-read, webhook registration, webhook delivery (mock HTTP), and retry behavior"

constraints:
  - "Webhook payloads must be signed with HMAC using the endpoint's secret so recipients can verify authenticity"
  - "Webhook delivery must not block the triggering operation — all deliveries are enqueued as background jobs"
  - "Webhook retry uses exponential backoff with a maximum of 5 attempts"
  - "In-app notifications are scoped to the user or agency — users must only see their own notifications"
  - "Notification creation must be idempotent for the same event — do not create duplicate notifications for the same campaign completion or billing threshold crossing"
  - "All timestamps in UTC"

acceptance_criteria:
  - "When a campaign transitions to 'sent', a notification is created for the campaign creator"
  - "When campaign failure rate exceeds 5%, a high-priority notification is created for Admins"
  - "GET /notifications returns only notifications for the authenticated user, ordered by created_at descending"
  - "PUT /notifications/:id/read marks the notification as read and returns HTTP 200"
  - "POST /webhooks registers an endpoint and POST /webhooks/:id/test delivers a test payload"
  - "Webhook delivery records the response status and retries on failure with exponential backoff"
  - "Billing threshold notifications are created when usage reaches 80% and 95%"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Contact, Campaign, Automation, and Event Data Models"
    - "Campaign Builder and Scheduling"
    - "Automation Workflows"
  integrates_with:
    - "Event Tracking and Analytics"
    - "Billing and Usage Metering"

handoff: |
  Exposes for downstream specs:
  - Notification creation service for any future module to trigger in-app notifications
  - Webhook dispatch service for external system integrations
  - Notification and webhook endpoints for UI rendering

verification: "run project test suite"
```
