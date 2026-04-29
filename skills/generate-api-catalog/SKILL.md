---
name: generate-api-catalog
description: |
  Creates API_CATALOG.md mapping all endpoints with examples.
  Use when documenting the API for consumers (frontend devs, integrators, partners).
  Trigger with "generate API catalog" or "document the API".
---

# Generate API Catalog

You create API_CATALOG.md that documents every endpoint with full request/response examples.

## Document Structure

```markdown
# API Catalog — [Project Name]

## Base URL
- Production: `https://...`
- Staging: `https://...`
- Local: `http://localhost:8080`

## Authentication

### How to Authenticate
1. Obtain a token from [auth provider]
2. Include in every request: `Authorization: Bearer {token}`
3. Token format: JWT with claims: `sub`, `tenant_id`, `roles`

### Required Claims
| Claim | Type | Example |
|---|---|---|
| sub | UUID | `user_abc123` |
| tenant_id | UUID | `tenant_xyz789` |
| roles | string[] | `["admin"]` |

## Common Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | Bearer token |
| `Content-Type` | Yes (POST/PATCH) | `application/json` |
| `X-Request-ID` | Optional | UUID for request tracing |

## Common Errors

| Status | Meaning | Example Body |
|---|---|---|
| 400 | Validation error | `{"error": "title is required"}` |
| 401 | Missing/invalid token | `{"error": "Unauthorized"}` |
| 403 | Insufficient permissions | `{"error": "Admin role required"}` |
| 404 | Resource not found | `{"error": "Task not found"}` |
| 422 | Business rule violation | `{"error": "Cannot transition from TODO to DONE"}` |
| 429 | Rate limit exceeded | `{"error": "Too many requests"}` |

## Endpoints by Resource

### [Resource Name] (e.g., Tasks)

#### Create [Resource]
`POST /api/.../[resource]`

**Request**:
```json
{
  "field1": "value",
  "field2": 123
}
```

**Response 201**:
```json
{
  "id": "uuid",
  "field1": "value",
  "field2": 123,
  "created_at": "2026-04-29T10:30:00Z"
}
```

**Validation**:
- `field1`: required, max 255 chars
- `field2`: required, positive integer

**Permissions**: Admin or Member role

**Example cURL**:
```bash
curl -X POST https://.../api/.../tasks \
  -H "Authorization: Bearer eyJ..." \
  -H "Content-Type: application/json" \
  -d '{"field1": "value", "field2": 123}'
```

(Repeat structure for GET list, GET single, PATCH, DELETE)

## Pagination

Endpoints returning lists support pagination:
- `?page=1&size=20`
- Response includes: `X-Total-Count`, `X-Page-Count` headers

## Rate Limiting

- 100 requests per minute per tenant
- Headers returned:
  - `X-RateLimit-Limit: 100`
  - `X-RateLimit-Remaining: 87`
  - `X-RateLimit-Reset: 1714389600`

## Webhooks (if applicable)

### Available Events
- `task.created`
- `task.status_changed`
- `task.deleted`

### Webhook Payload
Standard structure for all events.

## Versioning

- Current: v1
- Deprecation policy: 6 months notice via response headers
```

## How to Generate

1. Scan all REST controllers/resources in the codebase
2. For each endpoint, extract:
   - HTTP method + path
   - Request DTO (parse Java/TS class)
   - Response DTO
   - Validation annotations
   - Auth annotations (@RolesAllowed, etc.)
3. Generate cURL example for each endpoint
4. Group by resource (Task, Project, User, etc.)
5. Save as API_CATALOG.md

## Quality Checklist

- [ ] Every endpoint in the codebase is documented
- [ ] Each endpoint has a working cURL example
- [ ] Request/response examples are valid JSON
- [ ] Permissions are explicit per endpoint
- [ ] Common errors section is complete
- [ ] Pagination is documented if used
- [ ] Rate limiting headers are documented
