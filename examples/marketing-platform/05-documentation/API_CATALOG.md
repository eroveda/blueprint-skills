# API Catalog — Marketing Automation Platform

## Base URL

- Production: `https://api.your-platform.com/v1`
- Staging: `https://api.staging.your-platform.com/v1`
- Local: `http://localhost:8080/v1`

## Authentication

### How to Authenticate

1. Obtain an access token via the login endpoint.
2. Include in every request: `Authorization: Bearer {access_token}`
3. Token format: JWT with claims `sub` (user ID), `tenant_id` (agency), `roles` (array).
4. Access tokens expire after 15 minutes. Use the refresh token to obtain a new one.

### Authentication Flows

#### Register — `POST /auth/register`

```bash
curl -X POST https://api.your-platform.com/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@agency.com","password":"secureP@ss123","agency_name":"Bright Agency","first_name":"Maria","last_name":"Lopez"}'
```

**Response 201**:
```json
{"user_id":"usr_a1b2c3d4","tenant_id":"ten_x9y8z7w6","email":"admin@agency.com","role":"admin","access_token":"eyJhbGciOi...","refresh_token":"rt_f5e6d7c8...","expires_in":900}
```

The first user who registers creates the agency and receives the Admin role.

#### Login — `POST /auth/login`

**Request**: `{"email":"admin@agency.com","password":"secureP@ss123"}`

**Response 200**:
```json
{"access_token":"eyJhbGciOi...","refresh_token":"rt_f5e6d7c8...","expires_in":900,"user":{"id":"usr_a1b2c3d4","email":"admin@agency.com","role":"admin","tenant_id":"ten_x9y8z7w6"}}
```

#### Refresh — `POST /auth/refresh`

**Request**: `{"refresh_token":"rt_f5e6d7c8..."}`

**Response 200**: `{"access_token":"eyJhbGciOi...","refresh_token":"rt_new_token...","expires_in":900}`

Refresh tokens are single-use (rotation policy). Using an already-consumed token revokes all tokens for that user.

#### Revoke — `POST /auth/revoke`

**Request**: `{"refresh_token":"rt_f5e6d7c8..."}`

**Response 204**: No content. Pass `"all": true` to revoke all sessions.

## Common Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {access_token}` |
| `Content-Type` | Yes (POST/PUT/PATCH) | `application/json` |
| `X-Request-ID` | Optional | UUID for request tracing |

## Common Errors

| Status | Meaning | Example Body |
|---|---|---|
| 400 | Validation error | `{"error": "email is required"}` |
| 401 | Missing/invalid token | `{"error": "Unauthorized"}` |
| 403 | Insufficient permissions | `{"error": "Admin role required"}` |
| 404 | Resource not found | `{"error": "Contact not found"}` |
| 409 | Conflict (duplicate) | `{"error": "Contact with this email already exists"}` |
| 422 | Business rule violation | `{"error": "Campaign can only be edited in draft status"}` |
| 429 | Rate limit exceeded | `{"error": "Too many requests", "retry_after": 30}` |

## Endpoints by Resource

---

### Contacts

#### List Contacts — `GET /contacts?page=1&size=20&list_id={id}&tag={tag}&q={search}`

```bash
curl https://api.your-platform.com/v1/contacts?page=1&size=20 -H "Authorization: Bearer eyJ..."
```

**Response 200**:
```json
{"data":[{"id":"con_1a2b3c","email":"jane@example.com","first_name":"Jane","last_name":"Doe","tags":["vip","newsletter"],"subscription_status":"subscribed","created_at":"2026-03-15T10:00:00Z"}],"pagination":{"page":1,"size":20,"total_items":1432,"total_pages":72}}
```

**Permissions**: Admin, Marketer (full), Viewer (read-only)

#### Create Contact — `POST /contacts`

```bash
curl -X POST https://api.your-platform.com/v1/contacts \
  -H "Authorization: Bearer eyJ..." -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","first_name":"John","last_name":"Smith","tags":["lead"],"list_ids":["lst_abc123"],"custom_fields":{"company":"Acme Inc"}}'
```

**Response 201**:
```json
{"id":"con_4d5e6f","email":"john@example.com","first_name":"John","last_name":"Smith","tags":["lead"],"list_ids":["lst_abc123"],"subscription_status":"subscribed","custom_fields":{"company":"Acme Inc"},"created_at":"2026-04-29T14:30:00Z","updated_at":"2026-04-29T14:30:00Z"}
```

**Validation**: `email` required, valid format, unique per tenant. `first_name`/`last_name` max 100 chars.

**Permissions**: Admin, Marketer

#### Import Contacts — `POST /contacts/import`

**Request**: `multipart/form-data` with `file` field (CSV or Excel).

**Response 202**: `{"import_id":"imp_7g8h9i","status":"processing","message":"Import started. Check status at /contacts/import/imp_7g8h9i"}`

**Permissions**: Admin, Marketer

#### Unsubscribe Contact — `POST /contacts/{id}/unsubscribe`

**Request**: `{"scope":"list","list_id":"lst_abc123"}` or `{"scope":"global"}` for global unsubscribe.

**Response 200**: `{"id":"con_4d5e6f","subscription_status":"unsubscribed_list","unsubscribed_from":["lst_abc123"]}`

**Permissions**: Admin, Marketer

---

### Lists

#### List All Lists — `GET /lists?page=1&size=20`

**Response 200**:
```json
{"data":[{"id":"lst_abc123","name":"Newsletter Subscribers","description":"Weekly newsletter recipients","contact_count":4520,"created_at":"2026-02-01T08:00:00Z"}],"pagination":{"page":1,"size":20,"total_items":5,"total_pages":1}}
```

**Permissions**: Admin, Marketer (full), Viewer (read-only)

#### Create List — `POST /lists`

```bash
curl -X POST https://api.your-platform.com/v1/lists \
  -H "Authorization: Bearer eyJ..." -H "Content-Type: application/json" \
  -d '{"name":"Webinar Attendees","description":"Contacts who attended the April webinar"}'
```

**Response 201**: `{"id":"lst_def456","name":"Webinar Attendees","description":"Contacts who attended the April webinar","contact_count":0,"created_at":"2026-04-29T14:30:00Z"}`

**Permissions**: Admin, Marketer

---

### Campaigns

#### List Campaigns — `GET /campaigns?page=1&size=20&status={status}`

**Response 200**:
```json
{"data":[{"id":"cmp_1a2b3c","name":"April Newsletter","subject":"Your April Updates","status":"sent","sent_at":"2026-04-15T09:00:00Z","created_at":"2026-04-10T11:00:00Z"}],"pagination":{"page":1,"size":20,"total_items":23,"total_pages":2}}
```

**Permissions**: Admin, Marketer, Viewer

#### Create Campaign — `POST /campaigns`

```bash
curl -X POST https://api.your-platform.com/v1/campaigns \
  -H "Authorization: Bearer eyJ..." -H "Content-Type: application/json" \
  -d '{"name":"May Promo","subject":"Exclusive May Deals Inside","from_name":"Bright Agency","from_email":"hello@brightagency.com","template_id":"tpl_abc123","recipient_list_ids":["lst_abc123"]}'
```

**Response 201**:
```json
{"id":"cmp_4d5e6f","name":"May Promo","subject":"Exclusive May Deals Inside","from_name":"Bright Agency","from_email":"hello@brightagency.com","template_id":"tpl_abc123","status":"draft","recipient_list_ids":["lst_abc123","lst_def456"],"segment_id":null,"scheduled_at":null,"sent_at":null,"created_at":"2026-04-29T14:30:00Z"}
```

**Validation**: `name` and `subject` required (max 255 chars). `from_email` required, valid format.

**Permissions**: Admin, Marketer

#### Schedule Campaign — `POST /campaigns/{id}/schedule`

**Request**: `{"scheduled_at":"2026-05-01T09:00:00Z"}`

**Response 200**: `{"id":"cmp_4d5e6f","status":"scheduled","scheduled_at":"2026-05-01T09:00:00Z"}`

**Validation**: Must be in `draft` status. `scheduled_at` must be in the future. At least one recipient list required.

**Error 422**: `{"error":"Campaign can only be scheduled from draft status"}`

#### Send Campaign Now — `POST /campaigns/{id}/send`

**Response 202**: `{"id":"cmp_4d5e6f","status":"sending","estimated_recipients":4520,"message":"Campaign send started. Delivery is in progress."}`

**Validation**: Must be in `draft` or `scheduled` status. Email quota must not be exceeded.

**Error 422**: `{"error":"Email quota exceeded. Your plan allows 10000 emails and you have 800 remaining. This campaign targets 4520 contacts."}`

#### Retry Failed Sends — `POST /campaigns/{id}/retry`

**Response 202**: `{"id":"cmp_4d5e6f","status":"sending","retrying_count":42,"message":"Retrying 42 failed sends. Already-delivered contacts will not receive duplicates."}`

**Permissions** (all campaign actions): Admin, Marketer

---

### Templates

#### List Templates — `GET /templates?page=1&size=20`

**Response 200**:
```json
{"data":[{"id":"tpl_abc123","name":"Standard Newsletter","merge_tags":["{{first_name}}","{{company}}"],"thumbnail_url":"https://cdn.your-platform.com/thumbs/tpl_abc123.png","created_at":"2026-03-01T10:00:00Z","updated_at":"2026-04-20T15:30:00Z"}],"pagination":{"page":1,"size":20,"total_items":8,"total_pages":1}}
```

**Permissions**: Admin, Marketer

#### Create Template — `POST /templates`

```bash
curl -X POST https://api.your-platform.com/v1/templates \
  -H "Authorization: Bearer eyJ..." -H "Content-Type: application/json" \
  -d '{"name":"Welcome Email","html_content":"<html><body><h1>Welcome, {{first_name}}!</h1></body></html>"}'
```

**Response 201**:
```json
{"id":"tpl_def456","name":"Welcome Email","html_content":"<html><body><h1>Welcome, {{first_name}}!</h1><p>Thanks for joining.</p></body></html>","merge_tags":["{{first_name}}"],"thumbnail_url":"https://cdn.your-platform.com/thumbs/tpl_def456.png","created_at":"2026-04-29T14:30:00Z","updated_at":"2026-04-29T14:30:00Z"}
```

#### Preview Template — `POST /templates/{id}/preview`

**Request**: `{"sample_data":{"first_name":"Jane","company":"Acme Inc"}}`

**Response 200**: `{"rendered_html":"<html><body><h1>Welcome, Jane!</h1><p>Thanks for joining.</p></body></html>"}`

---

### Automations

#### List Automations — `GET /automations?page=1&size=20&status={status}`

**Response 200**:
```json
{"data":[{"id":"aut_1a2b3c","name":"Welcome Sequence","trigger_type":"contact_added","status":"active","created_at":"2026-03-10T08:00:00Z"}],"pagination":{"page":1,"size":20,"total_items":3,"total_pages":1}}
```

**Permissions**: Admin, Marketer

#### Create Automation — `POST /automations`

**Request**:
```json
{
  "name": "Welcome Sequence",
  "trigger_type": "contact_added",
  "trigger_config": {"list_id": "lst_abc123"},
  "steps": [
    {"order": 1, "action_type": "send_email", "action_config": {"template_id": "tpl_def456"}, "delay_duration": "PT0S"},
    {"order": 2, "action_type": "wait", "action_config": {}, "delay_duration": "P3D"},
    {"order": 3, "action_type": "send_email", "action_config": {"template_id": "tpl_ghi789"}, "delay_duration": "PT0S"},
    {"order": 4, "action_type": "add_tag", "action_config": {"tag": "onboarded"}, "delay_duration": "PT0S"}
  ]
}
```

**Response 201**:
```json
{"id":"aut_4d5e6f","name":"Welcome Sequence","trigger_type":"contact_added","trigger_config":{"list_id":"lst_abc123"},"status":"draft","steps":[{"id":"stp_01","order":1,"action_type":"send_email","delay_duration":"PT0S"},{"id":"stp_02","order":2,"action_type":"wait","delay_duration":"P3D"},{"id":"stp_03","order":3,"action_type":"send_email","delay_duration":"PT0S"},{"id":"stp_04","order":4,"action_type":"add_tag","delay_duration":"PT0S"}],"created_at":"2026-04-29T14:30:00Z"}
```

#### Activate / Pause / Resume

- `POST /automations/{id}/activate` -- draft to active
- `POST /automations/{id}/pause` -- active to paused
- `POST /automations/{id}/resume` -- paused to active

```bash
curl -X POST https://api.your-platform.com/v1/automations/aut_4d5e6f/activate \
  -H "Authorization: Bearer eyJ..."
```

**Response 200**: `{"id":"aut_4d5e6f","status":"active","message":"Automation is now active."}`

**Error 422**: `{"error":"Automation must be in draft status to activate"}`

---

### Analytics

#### Campaign Metrics — `GET /analytics/campaigns/{campaign_id}`

```bash
curl https://api.your-platform.com/v1/analytics/campaigns/cmp_1a2b3c -H "Authorization: Bearer eyJ..."
```

**Response 200**:
```json
{
  "campaign_id": "cmp_1a2b3c",
  "campaign_name": "April Newsletter",
  "metrics": {"total_sent":4520,"delivered":4480,"opens":1850,"unique_opens":1620,"clicks":430,"unique_clicks":380,"bounces_hard":15,"bounces_soft":25,"unsubscribes":12,"conversions":45},
  "rates": {"open_rate":0.362,"click_through_rate":0.085,"bounce_rate":0.009,"unsubscribe_rate":0.003,"conversion_rate":0.010},
  "last_updated": "2026-04-15T14:25:00Z"
}
```

**Note**: Metrics may reflect up to a 5-minute delay from actual engagement.

**Permissions**: Admin, Marketer, Viewer

#### Dashboard Aggregate — `GET /analytics/dashboard?from=2026-04-01&to=2026-04-30`

**Response 200**:
```json
{"period":{"from":"2026-04-01","to":"2026-04-30"},"total_campaigns_sent":12,"total_emails_sent":54200,"average_open_rate":0.34,"average_click_rate":0.08,"average_bounce_rate":0.007,"total_conversions":320,"top_campaign":{"id":"cmp_1a2b3c","name":"April Newsletter","open_rate":0.362}}
```

#### Export Report — `GET /analytics/campaigns/{campaign_id}/export?format=csv`

Supported formats: `csv`, `pdf`. Returns file download with appropriate `Content-Type`.

**Permissions** (all analytics): Admin, Marketer, Viewer

---

### Billing

#### Get Current Plan and Usage — `GET /billing`

```bash
curl https://api.your-platform.com/v1/billing -H "Authorization: Bearer eyJ..."
```

**Response 200**:
```json
{
  "plan": {"id":"plan_starter","name":"Starter","contact_limit":5000,"email_send_limit":25000,"price":49.00},
  "usage": {"billing_period":{"start":"2026-04-01","end":"2026-04-30"},"contacts_used":3200,"emails_sent":18500,"contacts_remaining":1800,"emails_remaining":6500}
}
```

**Permissions**: Admin

#### Change Plan — `PUT /billing/plan`

**Request**: `{"plan_id":"plan_professional"}`

**Response 200**: `{"plan":{"id":"plan_professional","name":"Professional","contact_limit":25000,"email_send_limit":100000,"price":149.00},"message":"Plan changed successfully. New limits are effective immediately."}`

**Error 422**: `{"error":"Cannot downgrade: current usage (3200 contacts) exceeds the target plan limit (2000 contacts)."}`

#### Update Payment Method — `PUT /billing/payment-method`

**Request**: `{"payment_token":"tok_visa_4242..."}`

**Response 200**: `{"card_last_four":"4242","card_brand":"Visa","exp_month":12,"exp_year":2028,"message":"Payment method updated."}`

**Permissions** (all billing): Admin

---

## Pagination

All list endpoints support pagination:

| Parameter | Default | Description |
|---|---|---|
| `page` | 1 | Page number (1-based) |
| `size` | 20 | Items per page (max 100) |

Response includes: `{"pagination":{"page":1,"size":20,"total_items":1432,"total_pages":72}}`

## Rate Limiting

| Endpoint Group | Limit |
|---|---|
| Authentication (login, register) | 10 requests/minute |
| Standard API endpoints | 100 requests/minute per tenant |
| Event ingestion (tracking pixel, clicks) | 1,000 requests/minute |

**Response headers**: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (unix timestamp).

Exceeding the limit returns `429 Too Many Requests` with `retry_after` in the body.

## Webhooks

### Available Events

| Event | Trigger |
|---|---|
| `campaign.sent` | Campaign finishes sending |
| `campaign.failed` | Campaign send encounters errors |
| `contact.unsubscribed` | Contact unsubscribes (list or global) |
| `automation.completed` | Contact completes all automation steps |
| `billing.threshold` | Usage reaches 80% or 95% of plan limit |

### Webhook Payload

```json
{"event":"campaign.sent","timestamp":"2026-04-29T14:30:00Z","tenant_id":"ten_x9y8z7w6","data":{"campaign_id":"cmp_4d5e6f","campaign_name":"May Promo","total_sent":4520,"failed":3}}
```

Webhooks are delivered via `POST` to the configured URL. Failed deliveries are retried up to 3 times with exponential backoff.

## Versioning

- **Current version**: v1 (all endpoints prefixed with `/v1/`)
- **Deprecation policy**: 6 months of support after successor release, communicated via `Sunset` response header.
