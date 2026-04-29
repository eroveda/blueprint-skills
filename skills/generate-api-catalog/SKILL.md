---
name: generate-api-catalog
description: |
  Creates API_CATALOG.md or INTERFACE_CATALOG.md documenting all system interfaces with examples.
  Use when documenting APIs, CLIs, SDKs, or message interfaces for consumers (frontend devs, integrators, partners).
  Do NOT use for architecture overview (use generate-architecture-doc) or user-facing docs (use generate-user-guide).
  Trigger with "generate API catalog", "document the API", or "create interface catalog".
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

### Authentication Flows

Document the complete auth lifecycle:

#### Registration
1. [How a new user/client registers]
2. [What credentials are issued]
3. [Example request/response]

#### Login
1. [How to obtain an access token]
2. [Token format and expiration]
3. [Example request/response]

#### Token Refresh
1. [How to refresh an expired token]
2. [Refresh token rotation policy]
3. [Example request/response]

#### Revocation
1. [How to revoke a token/session]
2. [What happens to active sessions]
3. [Example request/response]

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

## Interface Types

Adapt the catalog structure to the project's interface type:

### REST / GraphQL APIs
Use the endpoint-by-resource structure above with HTTP method, path, request/response schemas, and cURL examples.

### CLI Tools
For each command:
- Command syntax with arguments and flags
- Description of what it does
- Example invocation with expected output
- Exit codes and their meanings

### SDK / Library
For each public function/method:
- Signature with parameter types and return type
- Description and usage example
- Error/exception types

### Message Queues / Event Streams
For each topic/queue:
- Message schema (publish and consume)
- Routing/partitioning strategy
- Example payload
- Consumer group expectations

### gRPC Services
For each service/method:
- Proto definition summary
- Request/response message schemas
- Streaming type (unary, server, client, bidirectional)
- Example using grpcurl or client code

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

1. Identify the project's interface type (REST, GraphQL, CLI, SDK, message queue, gRPC)
2. Scan the codebase for interface definitions (controllers, route handlers, command definitions, proto files, public functions)
3. For each interface, extract:
   - Method/command/topic signature
   - Input schema (request body, arguments, message format)
   - Output schema (response, return value, published events)
   - Validation rules
   - Auth/permission requirements
4. Generate usage examples for each interface (cURL, CLI invocation, code snippet, etc.)
5. Group by resource or domain area
6. Save as API_CATALOG.md or INTERFACE_CATALOG.md

## Quality Checklist

- [ ] Every endpoint in the codebase is documented
- [ ] Each endpoint has a working cURL example
- [ ] Request/response examples are valid JSON
- [ ] Permissions are explicit per endpoint
- [ ] Common errors section is complete
- [ ] Pagination is documented if used
- [ ] Rate limiting headers are documented
