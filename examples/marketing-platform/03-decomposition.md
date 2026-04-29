# Marketing Automation Platform — SWEBOK Decomposition

```yaml
project:
  name: "Marketing Automation Platform"
  type: "SaaS Web Application"

actors:
  - name: "Agency Admin"
    operations:
      - "manage users and roles"
      - "configure billing plan and payment"
      - "manage platform settings (ESP, branding, domains)"
      - "view all analytics and reports"
  - name: "Marketer"
    operations:
      - "manage contacts (create, import, tag, segment)"
      - "manage lists (create, assign contacts, filter)"
      - "create and edit email templates"
      - "create, edit, preview, schedule, and send campaigns"
      - "configure automation workflows (triggers, steps, drip sequences)"
      - "pause and resume automations"
      - "view campaign analytics and dashboards"
  - name: "Viewer/Client"
    operations:
      - "view campaign performance reports"
      - "view contact lists (read-only)"
      - "export reports as CSV/PDF"

entities:
  - name: "Contact"
    fields: ["id", "email", "first_name", "last_name", "tags", "list_ids", "subscription_status", "custom_fields", "created_at", "updated_at"]
  - name: "List"
    fields: ["id", "name", "description", "contact_count", "created_at"]
  - name: "Segment"
    fields: ["id", "name", "filter_rules", "evaluated_count", "created_at"]
  - name: "Campaign"
    fields: ["id", "name", "subject", "from_name", "from_email", "template_id", "status", "recipient_list_ids", "segment_id", "scheduled_at", "sent_at", "created_at"]
  - name: "Template"
    fields: ["id", "name", "html_content", "merge_tags", "thumbnail_url", "created_at", "updated_at"]
  - name: "Automation"
    fields: ["id", "name", "trigger_type", "trigger_config", "steps", "status", "paused_at", "created_at"]
  - name: "AutomationStep"
    fields: ["id", "automation_id", "order", "action_type", "action_config", "delay_duration"]
  - name: "Event"
    fields: ["id", "contact_id", "campaign_id", "type", "metadata", "timestamp"]
  - name: "BillingPlan"
    fields: ["id", "name", "contact_limit", "email_send_limit", "price", "features"]
  - name: "UsageRecord"
    fields: ["id", "agency_id", "billing_period", "contacts_used", "emails_sent", "created_at"]

nodes:
  - id: "1"
    title: "Project Bootstrap and Environment Setup"
    category: "OPERATIONS"
    purpose: "Initialize project structure, install dependencies, configure database connections, set up background job runner, define environment variables, and establish CI/CD pipeline with staging and production environments"
    depends_on: []

  - id: "2"
    title: "Contact, Campaign, Automation, and Event Data Models"
    category: "DESIGN"
    purpose: "Define all entity schemas (Contact, List, Segment, Campaign, Template, Automation, AutomationStep, Event, BillingPlan, UsageRecord), their relationships (contact-to-list many-to-many, campaign-to-template, automation-to-steps), database indexes for query performance, and the campaign status state machine (draft → scheduled → sending → sent)"
    depends_on: ["1"]

  - id: "3"
    title: "Authentication and Role-Based Access"
    category: "SECURITY"
    purpose: "Implement JWT-based authentication with refresh token rotation, enforce role-based access control for Admin, Marketer, and Viewer roles, ensure agency-level data isolation (multi-tenancy), and provide API key management for external integrations"
    depends_on: ["1"]

  - id: "4"
    title: "Contact and List Management"
    category: "CONSTRUCTION"
    purpose: "CRUD operations for contacts and lists, CSV/Excel contact import with validation and deduplication, tag application and removal, dynamic segment evaluation using filter rules, bulk operations (assign to list, apply tag), global and per-list unsubscribe processing, and contact export"
    depends_on: ["2", "3"]

  - id: "5"
    title: "Campaign Builder and Scheduling"
    category: "CONSTRUCTION"
    purpose: "Create and edit campaigns (draft status only), select recipient lists and segments, attach templates, preview rendered email, schedule campaigns for future send, execute async send pipeline (resolve recipients, deduplicate, queue emails), enforce status transitions (draft → scheduled → sending → sent with no skipping), and retry failed sends without duplicating to already-sent contacts"
    depends_on: ["2", "3"]

  - id: "6"
    title: "Template Engine"
    category: "CONSTRUCTION"
    purpose: "Create and manage reusable email templates with HTML/MJML editing, merge tag variable substitution (contact name, custom fields), template preview with sample data, template thumbnail generation, and template versioning"
    depends_on: ["2"]

  - id: "7"
    title: "Automation Workflows"
    category: "CONSTRUCTION"
    purpose: "Define automation triggers (contact added to list, tag applied, date field reached), configure multi-step workflows (send email, wait, add tag, move to list), execute drip sequences with configurable delays, pause and resume active automations without losing execution state, and trigger auto-responses on contact actions"
    depends_on: ["2", "3", "4"]

  - id: "8"
    title: "Event Tracking and Analytics"
    category: "CONSTRUCTION"
    purpose: "Ingest open events via tracking pixel (with device and client detection), record click events via redirect URL tracking, classify bounces (hard vs soft) and update contact status, attribute conversions to originating campaigns, aggregate per-campaign metrics (open rate, CTR, bounce rate, conversion rate), compute cross-campaign dashboards, and handle up to 5-minute event arrival delay (near-real-time processing)"
    depends_on: ["2", "5"]

  - id: "9"
    title: "Billing and Usage Metering"
    category: "CONSTRUCTION"
    purpose: "Track contact count and email send volume per billing cycle, enforce plan limits before campaign send (block sends that exceed quota rather than charging overage), generate usage summaries for invoicing, manage billing plan selection and payment method via Stripe integration, and send billing threshold alerts"
    depends_on: ["2", "3"]

  - id: "10"
    title: "Notification System"
    category: "CONSTRUCTION"
    purpose: "Deliver in-app notifications for campaign completion, send failures, and automation triggers, dispatch webhook notifications to external systems on campaign events, generate billing alerts when approaching or hitting plan limits, and surface bounce/complaint alerts for list hygiene"
    depends_on: ["2", "5", "7"]

validation:
  total_nodes: 10
  construction_count: 7
  construction_ratio: "7/10 (70%)"
  expected_operations:
    - "Admin → Users → manage (covered by Node 3: role-based access)"
    - "Admin → Billing → configure (covered by Node 9: billing and usage metering)"
    - "Admin → Platform Settings → manage (covered by Node 1: bootstrap and config)"
    - "Admin → Analytics → view all (covered by Node 8: event tracking and analytics)"
    - "Marketer → Contacts → CRUD/import/tag/segment (covered by Node 4: contact and list management)"
    - "Marketer → Lists → create/assign/filter (covered by Node 4: contact and list management)"
    - "Marketer → Templates → create/edit (covered by Node 6: template engine)"
    - "Marketer → Campaigns → create/edit/preview/schedule/send (covered by Node 5: campaign builder)"
    - "Marketer → Automations → configure/pause/resume (covered by Node 7: automation workflows)"
    - "Marketer → Analytics → view dashboards (covered by Node 8: event tracking and analytics)"
    - "Viewer → Reports → view (covered by Node 8: event tracking and analytics)"
    - "Viewer → Contact Lists → view read-only (covered by Node 4: contact and list management)"
    - "Viewer → Reports → export CSV/PDF (covered by Node 8: event tracking and analytics)"
  coverage: "13/13 operations covered"
  business_rules_coverage:
    - "Campaigns editable only in draft status → Node 5 enforces draft-only editing"
    - "Campaign status flow: draft → scheduled → sending → sent → Node 5 enforces sequential transitions"
    - "Failed sends retryable without duplication → Node 5 deduplicates against already-sent contacts"
    - "Automations pausable/resumable without losing state → Node 7 preserves execution state"
    - "Contact unsubscribe (per-list or global) → Node 4 processes unsubscribe requests"
    - "Billing blocks sends over quota (no overage) → Node 9 enforces plan limits before send"
  constraints_coverage:
    - "All timestamps in UTC → Node 2 schema design enforces UTC storage"
    - "Async email sending (100k contacts non-blocking) → Node 5 uses background queue pipeline"
    - "Analytics events up to 5-minute delay → Node 8 handles near-real-time ingestion"
  orphan_check: "No orphan dependencies — all depends_on references point to valid node IDs"
```
