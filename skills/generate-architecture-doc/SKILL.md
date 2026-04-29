---
name: generate-architecture-doc
description: |
  Generates ARCHITECTURE.md following IEEE 15289 standards.
  Use when documenting a new or existing project's technical architecture.
  Do NOT use for user-facing docs (use generate-user-guide) or API docs (use generate-api-catalog).
  Trigger with "generate architecture doc", "document the architecture", or "create ARCHITECTURE.md".
---

# Generate Architecture Documentation

You create professional ARCHITECTURE.md files following IEEE/ISO 15289 (Life-cycle Information Items).

## Document Structure

```markdown
# Architecture — [Project Name]

## Overview
Brief description of the system's purpose, scope, and primary actors.

## Tech Stack
| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Runtime | ... | ... | ... |
| Framework | ... | ... | ... |
| Database | ... | ... | ... |
| Auth | ... | ... | ... |

## Component Architecture

### Module: [name]
- **Responsibility**: ...
- **Public API**: classes/interfaces exposed
- **Dependencies**: other modules consumed
- **Database**: tables owned

(Repeat for each module)

## Data Model

### Entities
For each entity:
- **Name** and primary key
- **Fields** with types and constraints
- **Relationships** to other entities
- **Indexes** for performance

### Schema Changes
List of schema migrations or changes in order, with what each one does.

## API Contracts

(Adapt this section to the project's interface type: REST endpoints, GraphQL schema, gRPC services, CLI commands, SDK methods, or message queue topics)

### Endpoints by Resource
For each resource (e.g., Task, Project, User):
- HTTP method + path
- Request/response schemas
- Auth requirements
- Error responses

## Cross-Cutting Concerns

### Authentication & Authorization
- Auth mechanism (JWT, OAuth, etc.)
- Token claims structure
- Role-based access control rules

### Multi-Tenancy (if applicable)
- Tenant isolation strategy
- Tenant ID propagation
- Cross-tenant data leak prevention

### Rate Limiting (if applicable)
- Limits per tenant/user
- Implementation strategy
- Headers exposed (X-RateLimit-*)

### Logging & Observability
- Log levels and structure
- Tracing/metrics endpoints

## Non-Functional Requirements

| NFR | Target | Validation |
|---|---|---|
| Response time (P95) | < 100ms | Load test |
| Concurrent users | 500 | Stress test |
| Availability | 99.9% | Uptime monitoring |

## Architectural Decisions (ADRs)

### ADR-001: [Decision title]
- **Status**: Accepted/Superseded
- **Context**: Why this decision was needed
- **Decision**: What was decided
- **Consequences**: Trade-offs

(Repeat for each significant decision)

## Standards Compliance

- IEEE SWEBOK v4.0 — used for component categorization
- ISO/IEC/IEEE 12207 — software lifecycle processes
- ISO/IEC/IEEE 15289 — documentation standards
- (Additional standards as applicable)

## Glossary

Definitions of domain-specific terms used throughout the codebase.
```

## How to Use

1. Read the project structure: source files, package layout, README
2. Identify modules and their responsibilities
3. Map entities and relationships from data model definitions (ORM models, schema files, migration scripts)
4. Extract interfaces from the codebase (API endpoints, CLI commands, SDK methods, message handlers)
5. Identify cross-cutting concerns from middleware, interceptors, decorators, or framework configuration
6. Generate the document in the order above
7. Save as ARCHITECTURE.md in project root

## Quality Checklist

- [ ] Tech stack table with exact versions
- [ ] Each module has Responsibility, Public API, Dependencies
- [ ] Entity diagram or table with all fields
- [ ] All endpoints documented (no orphan controllers)
- [ ] Auth flow described step-by-step
- [ ] At least 3 ADRs for significant decisions
- [ ] Standards compliance section
- [ ] No code blocks longer than 20 lines (link to file instead)
