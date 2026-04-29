# Spec: Event Tracking and Analytics

```yaml
title: "Event Tracking and Analytics"
purpose: "Ingest engagement events, classify bounces, attribute conversions, and compute campaign and cross-campaign analytics"
category: "CONSTRUCTION"

input_context: |
  From "Contact, Campaign, Automation, and Event Data Models":
  - Event table and model class with fields: id, contact_id, campaign_id, automation_id, type (enum: open, click, bounce_hard, bounce_soft, unsubscribe, conversion), metadata (JSON), timestamp (UTC)
  - Event table indexed on campaign_id + type, contact_id, and timestamp
  - Contact model with subscription_status field
  - Campaign model with status field
  - Multi-tenant agency_id pattern

  From "Campaign Builder and Scheduling":
  - campaign_sends table tracking individual send statuses per contact per campaign
  - Campaign statistics endpoint pattern
  - Campaign send pipeline that generates trackable email sends

instructions:
  - "Implement tracking pixel endpoint: GET /track/open — accept campaign_id and contact_id as query parameters (encoded in the pixel URL embedded in sent emails), record an Event with type 'open', extract device and email client information from the request User-Agent header and store in metadata"
  - "Implement click tracking endpoint: GET /track/click — accept campaign_id, contact_id, and target_url as query parameters, record an Event with type 'click' and the target URL in metadata, redirect the user to the target_url with HTTP 302"
  - "Implement bounce processing endpoint: POST /events/bounces — accept bounce notifications (from ESP webhooks), classify each bounce as bounce_hard or bounce_soft based on the bounce code or reason, record the Event, and for hard bounces update the contact's subscription_status to 'bounced'"
  - "Implement unsubscribe event processing: when a contact unsubscribes (via the Contact and List Management unsubscribe service), record an Event with type 'unsubscribe' attributed to the originating campaign if known"
  - "Implement conversion tracking endpoint: POST /events/conversions — accept conversion events with campaign_id, contact_id, and conversion metadata (value, type), attribute the conversion to the originating campaign, record the Event"
  - "Implement event ingestion queue: all event recording must go through a background job queue to handle high-volume bursts — the tracking endpoints return HTTP 200/302 immediately and enqueue the event for async processing"
  - "Implement per-campaign analytics endpoint: GET /campaigns/:id/analytics — compute and return: total sends, unique opens, unique clicks, open rate (unique opens / total sends), click-through rate (unique clicks / unique opens), hard bounce count, soft bounce count, bounce rate, unsubscribe count, conversion count, and conversion rate"
  - "Implement aggregate analytics endpoint: GET /analytics/dashboard — return cross-campaign metrics for the authenticated agency: total campaigns sent, total emails sent, average open rate, average CTR, total conversions, and a time-series of sends per day/week/month based on a date range parameter"
  - "Implement campaign comparison endpoint: GET /analytics/compare — accept an array of campaign_ids, return side-by-side metrics for each campaign"
  - "Implement contact engagement history endpoint: GET /contacts/:id/events — return paginated events for a specific contact, ordered by timestamp descending"
  - "Implement report export endpoint: POST /analytics/export — accept campaign_id or date range, generate a CSV or PDF report of campaign analytics, enqueue as a background job, and return a download URL when complete"
  - "Apply authentication and authorization to all endpoints: Marketer and Viewer roles can read analytics; tracking pixel and click endpoints are public (no auth — they are embedded in emails)"
  - "Apply agency-scoped filtering to all analytics queries"
  - "Write unit tests for bounce classification logic, open/click deduplication (count unique opens per contact, not total pixel loads), and analytics computation formulas"
  - "Write integration tests for event ingestion (open, click, bounce, conversion), per-campaign analytics accuracy, aggregate dashboard, and report export"

constraints:
  - "Analytics events arrive with up to 5-minute delay — the system must handle near-real-time ingestion, not require real-time processing"
  - "Tracking pixel and click redirect endpoints must respond within 200ms — event recording is deferred to background processing"
  - "Open and click metrics must count unique contacts, not raw event counts (a contact opening an email 5 times counts as 1 unique open)"
  - "Hard bounces must automatically update the contact subscription_status to 'bounced' to prevent future sends"
  - "All timestamps in UTC"
  - "Analytics queries must be performant for agencies with millions of events — use pre-aggregation or materialized views where appropriate"

acceptance_criteria:
  - "GET /track/open records an open event and returns a 1x1 transparent pixel image"
  - "GET /track/click records a click event and redirects to the target URL with HTTP 302"
  - "A hard bounce event updates the contact subscription_status to 'bounced'"
  - "GET /campaigns/:id/analytics returns correct open rate calculated as unique opens divided by total sends"
  - "A contact opening the same email 3 times results in unique_opens count of 1 for that campaign"
  - "GET /analytics/dashboard returns aggregate metrics scoped to the authenticated agency"
  - "Tracking endpoints respond in under 200ms even under load — event processing is asynchronous"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Contact, Campaign, Automation, and Event Data Models"
    - "Campaign Builder and Scheduling"
  integrates_with:
    - "Contact and List Management"
    - "Notification System"

handoff: |
  Exposes for downstream specs:
  - Event ingestion interface for recording events from automations and external sources
  - Per-campaign analytics service for embedding in campaign detail views
  - Aggregate analytics service for dashboard rendering
  - Contact engagement history for contact detail views
  - Bounce and unsubscribe event processing for list hygiene in Notification System

verification: "run project test suite"
```
