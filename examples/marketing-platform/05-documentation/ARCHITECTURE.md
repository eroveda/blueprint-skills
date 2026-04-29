# Architecture — Marketing Automation Platform

## Overview

The Marketing Automation Platform is a multi-tenant SaaS application that enables marketing agencies to manage contacts, design and send email campaigns, configure automation workflows, and track engagement analytics. The system serves three actor types: Agency Admins (platform and billing management), Marketers (day-to-day campaign operations), and Viewers/Clients (read-only access to reports).

The platform handles high-volume asynchronous email delivery, near-real-time event ingestion, and usage-based billing enforcement.

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Runtime | Chosen runtime | Server-side application execution |
| Framework | Chosen web framework | HTTP routing, middleware, request handling |
| Database (primary) | Chosen relational database | Persistent storage for entities and relationships |
| Database (cache) | Chosen in-memory store | Session cache, rate limiting counters, job queues |
| Background jobs | Chosen job runner | Async email sending, event processing, automation execution |
| File storage | Chosen object store | Template thumbnails, CSV imports, report exports |
| Email delivery | Chosen ESP integration | Outbound email sending via configurable provider |
| Payments | Chosen payment provider | Subscription billing and payment method management |
| Search (optional) | Chosen search engine | Full-text contact search and segment evaluation |

## Component Architecture

### Module: Authentication & Access Control

- **Responsibility**: User registration, login, JWT issuance with refresh token rotation, role-based access control (Admin, Marketer, Viewer), API key management for external integrations
- **Public API**: `AuthController`, `RoleGuard` middleware, `ApiKeyService`
- **Dependencies**: None (foundational module)
- **Database**: `users`, `refresh_tokens`, `api_keys`, `roles`

### Module: Contact & List Management

- **Responsibility**: CRUD for contacts and lists, CSV/Excel import with validation and deduplication, tag management, dynamic segment evaluation via filter rules, bulk operations, global and per-list unsubscribe processing
- **Public API**: `ContactController`, `ListController`, `SegmentController`, `ImportService`
- **Dependencies**: Authentication & Access Control
- **Database**: `contacts`, `lists`, `contact_list_memberships`, `segments`, `tags`

### Module: Template Engine

- **Responsibility**: Create and manage reusable email templates, HTML editing with merge tag substitution, template preview with sample data, thumbnail generation, versioning
- **Public API**: `TemplateController`, `MergeTagRenderer`
- **Dependencies**: None (consumed by Campaign Builder)
- **Database**: `templates`, `template_versions`

### Module: Campaign Builder & Scheduling

- **Responsibility**: Campaign lifecycle management (create, edit in draft only, preview, schedule, send), async send pipeline (resolve recipients, deduplicate, queue), status enforcement (draft -> scheduled -> sending -> sent), retry failed sends without duplicating
- **Public API**: `CampaignController`, `SendPipeline`, `CampaignScheduler`
- **Dependencies**: Contact & List Management, Template Engine, Billing & Usage
- **Database**: `campaigns`, `campaign_recipients`, `send_logs`

### Module: Automation Workflows

- **Responsibility**: Define triggers (contact added to list, tag applied, date reached), configure multi-step workflows (send email, wait, add tag, move to list), execute drip sequences with delays, pause/resume without losing state
- **Public API**: `AutomationController`, `WorkflowEngine`, `TriggerEvaluator`
- **Dependencies**: Contact & List Management, Template Engine
- **Database**: `automations`, `automation_steps`, `automation_enrollments`

### Module: Event Tracking & Analytics

- **Responsibility**: Ingest open events (tracking pixel), click events (redirect URL), classify bounces, attribute conversions, aggregate per-campaign metrics, compute cross-campaign dashboards, handle up to 5-minute event delay
- **Public API**: `EventController`, `TrackingPixelEndpoint`, `AnalyticsController`
- **Dependencies**: Campaign Builder
- **Database**: `events`, `campaign_metrics`, `dashboard_aggregates`

### Module: Billing & Usage Metering

- **Responsibility**: Track contact count and email volume per billing cycle, enforce plan limits before send (block, not overage), generate usage summaries, manage plan selection and payment method, send threshold alerts
- **Public API**: `BillingController`, `UsageEnforcer`, `PlanService`
- **Dependencies**: Authentication & Access Control
- **Database**: `billing_plans`, `usage_records`, `payment_methods`, `invoices`

### Module: Notification System

- **Responsibility**: In-app notifications for campaign completion, send failures, automation triggers; webhook dispatch to external systems; billing alerts at plan thresholds; bounce/complaint alerts for list hygiene
- **Public API**: `NotificationController`, `WebhookDispatcher`
- **Dependencies**: Campaign Builder, Automation Workflows, Billing & Usage
- **Database**: `notifications`, `webhook_configs`, `webhook_deliveries`

## Data Model

### Entities

#### Contact
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| email | String | Unique per tenant, required, valid email |
| first_name | String | Optional, max 100 chars |
| last_name | String | Optional, max 100 chars |
| tags | String[] | Array of tag labels |
| list_ids | UUID[] | Many-to-many via join table |
| subscription_status | Enum | `subscribed`, `unsubscribed_list`, `unsubscribed_global` |
| custom_fields | JSON | Flexible key-value pairs |
| created_at | Timestamp | UTC, auto-set |
| updated_at | Timestamp | UTC, auto-updated |

**Indexes**: `email` (unique per tenant), `subscription_status`, `tags` (GIN/array index)

#### List
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| name | String | Required, max 255 chars |
| description | String | Optional |
| contact_count | Integer | Denormalized counter, >= 0 |
| created_at | Timestamp | UTC |

**Relationships**: Many-to-many with Contact via `contact_list_memberships`

#### Segment
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| name | String | Required |
| filter_rules | JSON | Array of condition objects |
| evaluated_count | Integer | Computed on evaluation |
| created_at | Timestamp | UTC |

#### Campaign
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| name | String | Required, max 255 chars |
| subject | String | Required for scheduling |
| from_name | String | Required for scheduling |
| from_email | String | Required, valid email |
| template_id | UUID | FK to Template |
| status | Enum | `draft`, `scheduled`, `sending`, `sent` |
| recipient_list_ids | UUID[] | At least one required for scheduling |
| segment_id | UUID | Optional FK to Segment |
| scheduled_at | Timestamp | Required when status = scheduled |
| sent_at | Timestamp | Set when sending completes |
| created_at | Timestamp | UTC |

**Indexes**: `status`, `scheduled_at`, `tenant_id + status`

#### Template
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| name | String | Required |
| html_content | Text | Required |
| merge_tags | String[] | Extracted from html_content |
| thumbnail_url | String | Generated on save |
| created_at | Timestamp | UTC |
| updated_at | Timestamp | UTC |

#### Automation
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| name | String | Required |
| trigger_type | Enum | `contact_added`, `tag_applied`, `date_reached` |
| trigger_config | JSON | Trigger-specific parameters |
| steps | Relation | One-to-many with AutomationStep |
| status | Enum | `draft`, `active`, `paused` |
| paused_at | Timestamp | Set when paused |
| created_at | Timestamp | UTC |

#### AutomationStep
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| automation_id | UUID | FK to Automation |
| order | Integer | Sequential, >= 1 |
| action_type | Enum | `send_email`, `wait`, `add_tag`, `move_to_list` |
| action_config | JSON | Action-specific parameters |
| delay_duration | Interval | Time to wait before executing |

#### Event
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| contact_id | UUID | FK to Contact |
| campaign_id | UUID | FK to Campaign |
| type | Enum | `open`, `click`, `bounce_hard`, `bounce_soft`, `unsubscribe`, `conversion` |
| metadata | JSON | Device info, URL clicked, etc. |
| timestamp | Timestamp | UTC, may arrive with up to 5-min delay |

**Indexes**: `campaign_id + type`, `contact_id`, `timestamp`

#### BillingPlan
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| name | String | Required |
| contact_limit | Integer | Max contacts allowed |
| email_send_limit | Integer | Max emails per billing cycle |
| price | Decimal | Monthly price |
| features | JSON | Feature flags and limits |

#### UsageRecord
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| agency_id | UUID | FK to tenant |
| billing_period | DateRange | Start and end of cycle |
| contacts_used | Integer | Current contact count |
| emails_sent | Integer | Emails sent this cycle |
| created_at | Timestamp | UTC |

## API Contracts

See [API_CATALOG.md](./API_CATALOG.md) for the complete endpoint reference with request/response examples.

Summary of resources exposed:
- **Auth**: register, login, refresh, revoke
- **Contacts**: CRUD, import, bulk operations, unsubscribe
- **Lists**: CRUD, membership management
- **Segments**: CRUD, evaluate
- **Campaigns**: CRUD, schedule, send, retry
- **Templates**: CRUD, preview, render
- **Automations**: CRUD, activate, pause, resume
- **Events**: ingest (tracking pixel, redirect), query
- **Analytics**: per-campaign metrics, dashboards
- **Billing**: plans, usage, payment methods

## Cross-Cutting Concerns

### Authentication & Authorization

- **Mechanism**: JWT access tokens (short-lived) with refresh token rotation
- **Token claims**: `sub` (user ID), `tenant_id` (agency), `roles` (array of role strings)
- **RBAC enforcement**: Middleware checks role claims against endpoint requirements
- **Multi-tenancy**: Every query is scoped by `tenant_id` extracted from token; no cross-tenant data access is possible at the query layer
- **API keys**: Long-lived keys for external integrations, scoped to specific permissions

### Rate Limiting

- 100 requests per minute per tenant for standard endpoints
- 10 requests per minute for auth endpoints (login, register)
- 1,000 requests per minute for event ingestion (tracking pixel, click redirect)
- Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
- Enforced via in-memory store with sliding window counters

### Logging & Observability

- Structured JSON logging with correlation IDs (`X-Request-ID` propagated)
- Log levels: `error` (failures), `warn` (degraded), `info` (request lifecycle), `debug` (development)
- Metrics: request latency (p50/p95/p99), email send throughput, event ingestion rate, queue depth
- Health endpoint at `/health` with dependency checks (database, cache, job runner)

## Non-Functional Requirements

| NFR | Target | Validation |
|---|---|---|
| API response time (P95) | < 200ms | Load testing with realistic payloads |
| Campaign send throughput | 10,000 emails/minute | Queue-based stress test |
| Event ingestion latency | < 30 seconds processing | End-to-end tracking test |
| Concurrent users | 200 per tenant | Stress test with mixed workloads |
| Availability | 99.9% | Uptime monitoring with alerting |
| Data durability | No data loss | Database backups, WAL archiving |
| CSV import (10k rows) | < 60 seconds | Batch import benchmark |

## Architectural Decisions (ADRs)

### ADR-001: Asynchronous Email Send Pipeline

- **Status**: Accepted
- **Context**: Campaigns targeting 100k+ contacts must not block the UI. Synchronous sending would time out HTTP requests and degrade user experience.
- **Decision**: Email sending is handled by a background job queue. When a campaign is sent, the system resolves recipients, deduplicates against already-sent contacts, and enqueues individual send jobs. The campaign status transitions from `sending` to `sent` only when all jobs complete.
- **Consequences**: Adds complexity in job management, retry logic, and progress tracking. Requires a reliable job runner with at-least-once delivery. Users see send progress via polling or real-time updates rather than immediate confirmation.

### ADR-002: Block-on-Quota Billing Model

- **Status**: Accepted
- **Context**: Two approaches for handling plan limits: (a) allow overage and charge extra, or (b) block operations that exceed the quota. Overage billing introduces billing disputes and unexpected charges.
- **Decision**: The system blocks campaign sends when the email quota or contact limit would be exceeded. Users must upgrade their plan or wait for the next billing cycle. A pre-send check calculates estimated sends and compares against remaining quota.
- **Consequences**: Simpler billing logic, no surprise charges. May frustrate users who hit limits mid-campaign. Mitigated by threshold alerts at 80% and 95% usage.

### ADR-003: Near-Real-Time Event Processing

- **Status**: Accepted
- **Context**: Email engagement events (opens, clicks, bounces) arrive from external email service providers with variable latency. Guaranteeing real-time accuracy would require complex streaming infrastructure.
- **Decision**: Events are accepted with up to a 5-minute delay. The system batches incoming events and processes them in periodic sweeps. Dashboard metrics are labeled as "approximate" with a timestamp of last refresh.
- **Consequences**: Simplifies architecture by avoiding real-time streaming. Dashboards show slightly stale data. Acceptable for marketing analytics where minute-level precision is not critical.

### ADR-004: Multi-Tenant Data Isolation via Tenant-Scoped Queries

- **Status**: Accepted
- **Context**: Agencies must never see each other's data. Options include separate databases per tenant, separate schemas, or shared tables with tenant ID filtering.
- **Decision**: Shared tables with mandatory `tenant_id` column on all tenant-owned tables. A middleware layer injects `tenant_id` into every query automatically. Row-level security policies provide a secondary enforcement layer.
- **Consequences**: Simpler operations (single database), easier migrations. Requires discipline to ensure every query is scoped. Mitigated by automated tests that verify tenant isolation.

## Standards Compliance

- IEEE SWEBOK v4.0 -- used for module categorization (Construction, Design, Security, Operations)
- ISO/IEC/IEEE 12207 -- software lifecycle processes
- ISO/IEC/IEEE 15289 -- documentation standards applied to this document
- GDPR considerations -- contact data handling, unsubscribe processing, data export capabilities

## Glossary

| Term | Definition |
|---|---|
| **Tenant** | An agency account; all data is isolated per tenant |
| **Contact** | An individual email recipient managed within the platform |
| **List** | A named collection of contacts used for campaign targeting |
| **Segment** | A dynamic group of contacts defined by filter rules, evaluated at query time |
| **Campaign** | A single email send operation targeting one or more lists/segments |
| **Template** | A reusable HTML email layout with merge tag placeholders |
| **Merge tag** | A variable placeholder in a template (e.g., `{{first_name}}`) replaced with contact data at send time |
| **Automation** | A trigger-based workflow that executes a sequence of steps automatically |
| **Drip sequence** | An automation that sends a series of emails with configurable delays between them |
| **Event** | A tracked engagement action (open, click, bounce, unsubscribe, conversion) |
| **Bounce (hard)** | A permanent delivery failure (invalid address); contact should be suppressed |
| **Bounce (soft)** | A temporary delivery failure (mailbox full); delivery can be retried |
| **ESP** | Email Service Provider -- the external service used to deliver outbound emails |
| **Quota** | The maximum number of contacts or emails allowed by a billing plan |
