# Spec: Billing and Usage Metering

```yaml
title: "Billing and Usage Metering"
purpose: "Track usage against plan limits, enforce quota before campaign sends, manage billing plans, and generate usage summaries"
category: "CONSTRUCTION"

input_context: |
  From "Contact, Campaign, Automation, and Event Data Models":
  - BillingPlan table and model class with fields: id, name, contact_limit, email_send_limit, price, features
  - UsageRecord table and model class with fields: id, agency_id, billing_period, contacts_used, emails_sent
  - Contact model for counting active contacts per agency
  - Multi-tenant agency_id pattern

  From "Authentication and Role-Based Access":
  - Authentication and authorization middleware
  - Admin role required for billing management; Marketer read-only on billing info
  - Agency-scoped query filter
  - Authenticated user context

instructions:
  - "Implement billing plan CRUD endpoints (platform-level, super-admin or seeded): POST /billing-plans (create), GET /billing-plans (list all available plans), GET /billing-plans/:id (detail), PUT /billing-plans/:id (update), DELETE /billing-plans/:id (only if no agencies are subscribed)"
  - "Create an agency_subscriptions table (migration): fields id, agency_id (unique — one active plan per agency), billing_plan_id (foreign key), payment_method_token (encrypted reference to external payment provider), status (enum: active, past_due, cancelled), current_period_start (UTC), current_period_end (UTC), created_at"
  - "Implement plan subscription endpoints: POST /billing/subscribe (Admin only — select a plan for the agency), PUT /billing/subscription (Admin only — change plan), GET /billing/subscription (view current plan and usage)"
  - "Implement usage tracking service: maintain a running count of active contacts and emails sent per billing period for each agency; update contacts_used when contacts are created or deleted; update emails_sent when campaign sends are enqueued"
  - "Implement quota enforcement service: before a campaign send is executed, check whether the agency's current emails_sent plus the number of recipients would exceed the plan's email_send_limit; if it would exceed, reject the send and return an error — do not allow partial sends or overage charges"
  - "Implement contact limit enforcement: before creating or importing contacts, check whether the agency's contacts_used would exceed the plan's contact_limit; if it would exceed, reject the creation/import"
  - "Implement usage summary endpoint: GET /billing/usage — return current period contacts_used, emails_sent, contact_limit, email_send_limit, percentage used for each, and days remaining in the billing period"
  - "Implement usage history endpoint: GET /billing/usage/history — return paginated list of past billing period usage records for the agency"
  - "Implement billing threshold alerts: when usage reaches 80% and 95% of either limit, record a billing alert event that the Notification System can consume"
  - "Implement payment method management endpoints: POST /billing/payment-method (store payment reference via external payment provider integration), PUT /billing/payment-method (update), DELETE /billing/payment-method (remove)"
  - "Implement billing period rotation: a background job that runs daily, checks for agencies whose current_period_end has passed, creates a new UsageRecord for the new period, resets the running counts, and updates current_period_start and current_period_end"
  - "Seed initial billing plans: create at least three plans (e.g., Starter: 1,000 contacts / 5,000 emails; Growth: 10,000 contacts / 50,000 emails; Enterprise: 100,000 contacts / 500,000 emails) in the development seed script"
  - "Apply authentication and authorization: Admin for all write operations; Marketer can view usage and subscription"
  - "Write unit tests for quota enforcement logic (under limit, at limit, over limit), billing period rotation, and threshold alert triggers"
  - "Write integration tests for plan subscription, usage tracking across contact creation and campaign sends, quota rejection, and usage summary accuracy"

constraints:
  - "Billing enforces limits: sending beyond plan quota is blocked, not charged as overage"
  - "Contact limit enforcement must block creation, not just warn"
  - "Each agency has exactly one active billing plan at a time"
  - "Usage counts must be accurate even under concurrent operations — use atomic increments or transactions"
  - "Payment method tokens must be stored encrypted — never store raw payment credentials"
  - "All timestamps in UTC"

acceptance_criteria:
  - "A campaign send that would exceed the email_send_limit is rejected with HTTP 422 and a descriptive error"
  - "A contact import that would exceed the contact_limit is rejected with HTTP 422"
  - "GET /billing/usage returns accurate counts matching the actual number of contacts and emails sent in the current period"
  - "Billing period rotation creates a new UsageRecord and resets counts"
  - "Usage reaching 80% of email_send_limit triggers a threshold alert event"
  - "An Admin can subscribe to a plan and view usage; a Marketer can view usage but not change the plan"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Contact, Campaign, Automation, and Event Data Models"
    - "Authentication and Role-Based Access"
  integrates_with:
    - "Campaign Builder and Scheduling"
    - "Contact and List Management"
    - "Notification System"

handoff: |
  Exposes for downstream specs:
  - Quota enforcement service for campaign send pipeline and contact import to check before proceeding
  - Usage tracking service for incrementing counts from contact and campaign operations
  - Billing threshold alert events for Notification System to deliver alerts
  - Usage summary endpoint for display in admin dashboards

verification: "run project test suite"
```
