# Blueprint Skills v0.1 Rewrite Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make all 7 skills stack-agnostic and replace the example with two new cases (marketing platform + fraud detection pipeline).

**Architecture:** Each skill is a standalone SKILL.md file that instructs AI agents. Examples are structured directories with generated outputs. No code compilation — all deliverables are markdown.

**Tech Stack:** Markdown, YAML (for structured outputs within skills)

---

## File Structure

### Skills (modify existing)
- `skills/swebok-decompose/SKILL.md` — remove Java bias, new agnostic example, multi-layer guidance
- `skills/swebok-generate-spec/SKILL.md` — remove Java/Quarkus, agnostic example, add spec sequencing
- `skills/universal-framework/SKILL.md` — add output format, deeper project types, signal detection
- `skills/generate-architecture-doc/SKILL.md` — remove Flyway/JPA, generic placeholders
- `skills/generate-functional-flows/SKILL.md` — minor review
- `skills/generate-user-guide/SKILL.md` — minor review
- `skills/generate-api-catalog/SKILL.md` — add non-REST interfaces, auth flows

### Examples (delete + create)
- Delete: `examples/taskflow-api/`
- Create: `examples/marketing-platform/` (all files below)
- Create: `examples/fraud-detection-pipeline/` (all files below)

---

### Task 1: Rewrite `universal-framework` SKILL.md

This must go first because other skills reference it.

**Files:**
- Modify: `skills/universal-framework/SKILL.md`

- [ ] **Step 1: Rewrite the frontmatter**

Replace the `description` to clarify when to use and when NOT to use:

```yaml
---
name: universal-framework
description: |
  Applies the INPUTS → PROCESOS → OUTPUTS → SOPORTE framework to ANY software type.
  Use as the FIRST step when decomposing any project, especially non-standard types (IoT, ML, blockchain, simulation, etc.).
  Do NOT use for projects already decomposed or when the user only needs documentation.
  Trigger with "apply universal framework", "decompose using INPUTS-PROCESOS-OUTPUTS", or "what are the dimensions of this project".
---
```

- [ ] **Step 2: Expand each project type with more depth**

Replace the current 4-bullet lists per type with 6-8 bullets that include concrete sub-items. For example, REST API currently has:
```
- INPUTS: HTTP requests, payloads, JWT tokens
```
Expand to:
```
- INPUTS: HTTP requests (REST/GraphQL), request payloads (JSON/XML/form-data), authentication tokens (JWT/OAuth/API keys), webhook callbacks from external services, file uploads
```

Do this for all 8 project types: REST API, Data Pipeline/ETL, Mobile App, IoT, Blockchain, Machine Learning, Simulation, Research Project.

- [ ] **Step 3: Add "Signals to detect project type" section**

Add after "Applied to Different Project Types":

```markdown
## Signals to Detect Project Type

When the user doesn't explicitly state the project type, look for these keywords:

| Signal Keywords | Likely Type |
|---|---|
| endpoint, route, controller, API, REST, GraphQL, CRUD | REST API / Web Service |
| extract, transform, load, pipeline, warehouse, batch, ingest | Data Pipeline / ETL |
| screen, navigation, offline, push notification, app store | Mobile App |
| sensor, actuator, MQTT, firmware, OTA, edge, device | IoT |
| contract, wallet, token, mint, stake, chain, gas | Blockchain |
| train, model, predict, feature, dataset, inference, epoch | Machine Learning |
| simulate, iterate, parameter sweep, agent-based, Monte Carlo | Simulation |
| hypothesis, experiment, p-value, dataset, methodology | Research Project |

If keywords from multiple types appear, the project is likely **multi-layer** — apply the framework to each layer independently.
```

- [ ] **Step 4: Add structured output format**

Replace the current "How to Apply" section (which ends with "generate the work breakdown using swebok-decompose") with a section that produces concrete output:

```markdown
## Output Format

Produce a structured analysis before passing to `swebok-decompose`:

```yaml
project:
  name: "..."
  type: "REST API | Mobile | ETL | IoT | Blockchain | ML | Simulation | Research"
  layers:
    - name: "..."  # only if multi-layer
      type: "..."

dimensions:
  inputs:
    - category: "..."
      items:
        - "specific input 1"
        - "specific input 2"
  processes:
    - category: "..."
      items:
        - "specific process 1"
        - "specific process 2"
  outputs:
    - category: "..."
      items:
        - "specific output 1"
        - "specific output 2"
  support:
    - category: "..."
      items:
        - "specific support element 1"
        - "specific support element 2"

mapping_to_swebok:
  design_candidates: ["items from inputs/outputs that need shared structure"]
  construction_candidates: ["items from processes — these become work nodes"]
  security_candidates: ["items from support related to auth/access"]
  operations_candidates: ["items from support related to deploy/config/monitoring"]

next_step: "Pass this analysis to swebok-decompose to generate the work breakdown"
```
```

- [ ] **Step 5: Verify the complete file is coherent**

Read the full file, check that: framework definition, project types, signals, output format, multi-layer section, and anti-patterns all flow logically. No dangling references.

- [ ] **Step 6: Commit**

```bash
git add skills/universal-framework/SKILL.md
git commit -m "rewrite: universal-framework — add output format, signals, deeper project types"
```

---

### Task 2: Rewrite `swebok-decompose` SKILL.md

**Files:**
- Modify: `skills/swebok-decompose/SKILL.md`

- [ ] **Step 1: Rewrite the frontmatter**

```yaml
---
name: swebok-decompose
description: |
  Breaks down a project description into IEEE SWEBOK-categorized work nodes.
  Use after universal-framework (or directly for straightforward projects) to create the work breakdown.
  Do NOT use for documentation generation or spec creation — those have their own skills.
  Trigger with "decompose this project", "break down using SWEBOK", or "create work breakdown".
---
```

- [ ] **Step 2: Add multi-layer guidance**

Add after "Critical Rules":

```markdown
## Multi-Layer Projects

For projects with multiple layers (identified by `universal-framework`):

1. Generate a **separate node list per layer**
2. Each layer has its own Bootstrap node
3. Cross-layer dependencies are marked in `depends_on` with the format `"layer:node_id"`
4. Each layer must independently satisfy the CONSTRUCTION ratio ≥ 40%
```

- [ ] **Step 3: Replace the hardcoded TaskFlow example**

Remove the current example (lines 88-123) and replace with a stack-agnostic example:

```markdown
## Example

Input: "A marketing automation platform where agencies manage contacts, design email campaigns, set up automations, and track performance"

Output:
```yaml
project:
  name: "Marketing Automation Platform"
  type: "SaaS Web Application"

actors:
  - name: "Agency Admin"
    operations: ["manage users", "configure billing", "view all analytics"]
  - name: "Marketer"
    operations: ["manage contacts", "create campaigns", "set up automations", "view campaign analytics"]
  - name: "Viewer"
    operations: ["view campaign reports", "view contact lists"]

entities:
  - name: "Contact"
    fields: ["id", "email", "name", "tags", "list_id", "created_at"]
  - name: "Campaign"
    fields: ["id", "name", "subject", "template_id", "status", "scheduled_at", "sent_at"]
  - name: "Automation"
    fields: ["id", "name", "trigger_type", "steps", "status"]
  - name: "Event"
    fields: ["id", "contact_id", "campaign_id", "type", "timestamp"]

nodes:
  - id: "1"
    title: "Project Bootstrap and Environment Setup"
    category: "OPERATIONS"
    purpose: "Initialize project structure, dependencies, database, and configuration"
    depends_on: []
  - id: "2"
    title: "Contact, Campaign, Automation, and Event Data Models"
    category: "DESIGN"
    purpose: "Define all entities, relationships, and database schema"
    depends_on: ["1"]
  - id: "3"
    title: "Authentication and Role-Based Access"
    category: "SECURITY"
    purpose: "Implement auth flow and role enforcement for Admin, Marketer, Viewer"
    depends_on: ["1"]
  - id: "4"
    title: "Contact and List Management"
    category: "CONSTRUCTION"
    purpose: "CRUD for contacts and lists with import/export, tagging, segmentation"
    depends_on: ["2", "3"]
  - id: "5"
    title: "Campaign Builder and Scheduling"
    category: "CONSTRUCTION"
    purpose: "Create, edit, preview, schedule, and send email campaigns"
    depends_on: ["2", "3"]
  - id: "6"
    title: "Template Engine"
    category: "CONSTRUCTION"
    purpose: "Create and manage reusable email templates with variable substitution"
    depends_on: ["2"]
  - id: "7"
    title: "Automation Workflows"
    category: "CONSTRUCTION"
    purpose: "Define triggers, conditions, and actions for drip sequences and auto-responses"
    depends_on: ["2", "3", "4"]
  - id: "8"
    title: "Event Tracking and Analytics"
    category: "CONSTRUCTION"
    purpose: "Track opens, clicks, bounces, conversions and aggregate into dashboards"
    depends_on: ["2", "5"]
  - id: "9"
    title: "Billing and Usage Metering"
    category: "CONSTRUCTION"
    purpose: "Track email sends, contact limits, plan enforcement"
    depends_on: ["2", "3"]
  - id: "10"
    title: "Notification System"
    category: "CONSTRUCTION"
    purpose: "In-app and webhook notifications for campaign events, automation triggers, billing alerts"
    depends_on: ["2", "5", "7"]

validation:
  total_nodes: 10
  construction_count: 7
  construction_ratio: "7/10 (70%)"
  expected_operations:
    - "Admin → Users → manage"
    - "Marketer → Contacts → CRUD"
    - "Marketer → Campaigns → create/schedule/send"
    - "Marketer → Automations → create/configure"
    - "Marketer → Analytics → view"
    - "Viewer → Reports → view"
  coverage: "6/6 operations covered"
```
```

- [ ] **Step 4: Verify the complete file is coherent**

Read the full file. Check that: Universal Framework reference, SWEBOK categories, critical rules, multi-layer guidance, output format, anti-patterns, and new example all flow logically.

- [ ] **Step 5: Commit**

```bash
git add skills/swebok-decompose/SKILL.md
git commit -m "rewrite: swebok-decompose — agnostic example, multi-layer guidance, better frontmatter"
```

---

### Task 3: Rewrite `swebok-generate-spec` SKILL.md

**Files:**
- Modify: `skills/swebok-generate-spec/SKILL.md`

- [ ] **Step 1: Rewrite the frontmatter**

```yaml
---
name: swebok-generate-spec
description: |
  Converts a SWEBOK node into a complete executable specification.
  Use after swebok-decompose to generate detailed specs ready for code generation by any AI agent.
  Do NOT use for documentation — use the generate-* skills for that.
  Trigger with "generate spec for this node", "create executable specification", or "spec this node".
---
```

- [ ] **Step 2: Make the spec structure template agnostic**

Replace all Java-specific references in the template:

- `input_context`: change "Classes already created (with FQN paths)" → "Modules/classes/files already created (with paths)"
- `input_context`: change "Database tables already migrated" → "Database tables/collections already created"
- `input_context`: change "Specific class/file/package names from previous specs" → "Specific module/file/function names from previous specs"
- `instructions`: change "Reference exact class names, packages, file paths" → "Reference exact module names, file paths, function signatures"
- `handoff`: change "Class names exposed for injection" → "Modules/services exposed for consumption"
- `verification`: change `"./mvnw test"` → `"run project test suite (e.g., pytest, npm test, go test, ./mvnw test, mix test)"`

- [ ] **Step 3: Add spec ordering/sequencing section**

Add after "Mandatory Rules per Spec":

```markdown
## Spec Execution Order

When generating specs for all nodes in a decomposition:

1. **Build the dependency graph** from `depends_on` fields
2. **Topological sort** — specs with no dependencies first
3. **Group parallelizable specs** — specs with the same dependencies can run concurrently
4. **Output the execution order** as part of the spec set:

```yaml
execution_order:
  - phase: 1
    specs: ["Project Bootstrap and Environment Setup"]
    reason: "No dependencies — must run first"
  - phase: 2
    specs: ["Data Models", "Authentication"]
    reason: "Both depend only on Bootstrap — can run in parallel"
  - phase: 3
    specs: ["Contact Management", "Campaign Builder", "Template Engine"]
    reason: "Depend on Design + Security — can run in parallel"
  - phase: 4
    specs: ["Automation Workflows", "Event Tracking"]
    reason: "Depend on Construction specs from phase 3"
```
```

- [ ] **Step 4: Replace the Java example with an agnostic one**

Remove the current example (lines 73-128) and replace:

```markdown
## Example

For node "Campaign Builder and Scheduling":

```yaml
title: "Campaign Builder and Scheduling"
purpose: "Implement campaign creation, editing, preview, scheduling, and send functionality"
category: "CONSTRUCTION"

input_context: |
  Project initialized with chosen framework and language.
  Campaign entity exists with fields: id, name, subject, body, template_id,
  status (draft/scheduled/sending/sent/failed), scheduled_at, sent_at, created_by.
  Contact and List modules available for recipient selection.
  Auth module provides current user context and role verification.

instructions:
  - "Create campaign input validation (name required, subject max 200 chars, body required)"
  - "Implement create campaign — accepts name, subject, body, template_id, recipient_list_ids"
  - "Implement update campaign — only allowed when status is 'draft'"
  - "Implement campaign preview — render template with sample contact data"
  - "Implement schedule campaign — set scheduled_at, transition status to 'scheduled'"
  - "Implement send campaign — resolve recipient list, queue individual sends, transition to 'sending'"
  - "Implement campaign status query — filter by status, date range, created_by"
  - "Add unit tests for status transition rules"
  - "Add integration tests for campaign creation and scheduling"
  - "Update ARCHITECTURE.md with campaign module description"
  - "Update FUNCTIONAL_FLOWS.md with campaign lifecycle for Marketer"

constraints:
  - "Campaign can only be edited in 'draft' status"
  - "Status transitions: draft → scheduled → sending → sent (no skipping)"
  - "Failed sends must be retryable without duplicating to already-sent contacts"
  - "Scheduled campaigns must validate scheduled_at is in the future"
  - "All timestamps in UTC, ISO 8601 format"

acceptance_criteria:
  - "Creating a campaign returns the campaign with status 'draft'"
  - "Editing a 'scheduled' campaign is rejected"
  - "Scheduling with a past date is rejected"
  - "Sending resolves all contacts from recipient lists"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Contact, Campaign, Automation, and Event Data Models"
    - "Authentication and Role-Based Access"
  integrates_with:
    - "Template Engine"
    - "Notification System"

handoff: |
  Exposes:
  - Campaign service/module for consumption by Automation Workflows
  - Campaign status query for Analytics dashboard
  - Send queue interface for Event Tracking

verification: "run project test suite"
```
```

- [ ] **Step 5: Update mandatory rules to be agnostic**

Change rule 2 from "All code uses the base package defined in the Bootstrap spec" to "All code follows the project structure defined in the Bootstrap spec".

Change rule 5 from "Reference exact class names from dependency specs" to "Reference exact module/file/function names from dependency specs".

- [ ] **Step 6: Verify the complete file is coherent**

Read the full file. Check that no Java/Quarkus/Maven references remain anywhere.

- [ ] **Step 7: Commit**

```bash
git add skills/swebok-generate-spec/SKILL.md
git commit -m "rewrite: swebok-generate-spec — agnostic template and example, add spec sequencing"
```

---

### Task 4: Rewrite `generate-architecture-doc` SKILL.md

**Files:**
- Modify: `skills/generate-architecture-doc/SKILL.md`

- [ ] **Step 1: Rewrite the frontmatter**

```yaml
---
name: generate-architecture-doc
description: |
  Generates ARCHITECTURE.md following IEEE 15289 standards.
  Use when documenting a new or existing project's technical architecture.
  Do NOT use for user-facing docs (use generate-user-guide) or API docs (use generate-api-catalog).
  Trigger with "generate architecture doc", "document the architecture", or "create ARCHITECTURE.md".
---
```

- [ ] **Step 2: Make the document template agnostic**

In the Document Structure section:
- Change "### Migrations / List of Flyway/Liquibase migrations" → "### Schema Changes / List of schema migrations or changes in order, with what each one does"
- In "API Contracts" section, add: "(Adapt this section to the project's interface type: REST endpoints, GraphQL schema, gRPC services, CLI commands, SDK methods, or message queue topics)"
- In "Cross-Cutting Concerns", change the Multi-Tenancy subsection to be conditional: "### Multi-Tenancy (if applicable)"
- Same for Rate Limiting: "### Rate Limiting (if applicable)"

- [ ] **Step 3: Make "How to Use" agnostic**

Replace:
```
3. Map entities and relationships from JPA/database files
4. Extract API endpoints from controllers/resources
5. Identify cross-cutting concerns from filters/interceptors
```

With:
```
3. Map entities and relationships from data model definitions (ORM models, schema files, migration scripts)
4. Extract interfaces from the codebase (API endpoints, CLI commands, SDK methods, message handlers)
5. Identify cross-cutting concerns from middleware, interceptors, decorators, or framework configuration
```

- [ ] **Step 4: Verify and commit**

```bash
git add skills/generate-architecture-doc/SKILL.md
git commit -m "rewrite: generate-architecture-doc — remove Java-specific references, generic placeholders"
```

---

### Task 5: Rewrite `generate-api-catalog` SKILL.md

**Files:**
- Modify: `skills/generate-api-catalog/SKILL.md`

- [ ] **Step 1: Rewrite the frontmatter**

```yaml
---
name: generate-api-catalog
description: |
  Creates API_CATALOG.md or INTERFACE_CATALOG.md documenting all system interfaces with examples.
  Use when documenting APIs, CLIs, SDKs, or message interfaces for consumers (frontend devs, integrators, partners).
  Do NOT use for architecture overview (use generate-architecture-doc) or user-facing docs (use generate-user-guide).
  Trigger with "generate API catalog", "document the API", or "create interface catalog".
---
```

- [ ] **Step 2: Rename and restructure for non-REST support**

Change the title from "Generate API Catalog" to "Generate Interface Catalog".

After the "Endpoints by Resource" section in the document structure, add:

```markdown
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
```

- [ ] **Step 3: Add authentication flows section**

Add after the Authentication section in the document structure:

```markdown
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
```

- [ ] **Step 4: Make "How to Generate" agnostic**

Replace:
```
1. Scan all REST controllers/resources in the codebase
2. For each endpoint, extract:
   - HTTP method + path
   - Request DTO (parse Java/TS class)
   - Response DTO
   - Validation annotations
   - Auth annotations (@RolesAllowed, etc.)
```

With:
```
1. Identify the project's interface type (REST, GraphQL, CLI, SDK, message queue, gRPC)
2. Scan the codebase for interface definitions (controllers, route handlers, command definitions, proto files, public functions)
3. For each interface, extract:
   - Method/command/topic signature
   - Input schema (request body, arguments, message format)
   - Output schema (response, return value, published events)
   - Validation rules
   - Auth/permission requirements
```

- [ ] **Step 5: Verify and commit**

```bash
git add skills/generate-api-catalog/SKILL.md
git commit -m "rewrite: generate-api-catalog — support non-REST interfaces, add auth flows"
```

---

### Task 6: Minor review of `generate-functional-flows` and `generate-user-guide`

**Files:**
- Modify: `skills/generate-functional-flows/SKILL.md`
- Modify: `skills/generate-user-guide/SKILL.md`

- [ ] **Step 1: Update `generate-functional-flows` frontmatter**

```yaml
---
name: generate-functional-flows
description: |
  Creates FUNCTIONAL_FLOWS.md describing user-facing flows in plain language.
  Use when documenting for non-technical stakeholders (managers, PMs, clients).
  Do NOT use for technical architecture (use generate-architecture-doc) or API docs (use generate-api-catalog).
  Trigger with "generate functional flows", "document user flows", or "create FUNCTIONAL_FLOWS.md".
---
```

- [ ] **Step 2: Review `generate-functional-flows` content**

Scan for any stack-specific references. The current content is already agnostic. Only change needed: in Quality Checklist, change "Every endpoint has a corresponding flow" → "Every system capability has a corresponding flow".

- [ ] **Step 3: Update `generate-user-guide` frontmatter**

```yaml
---
name: generate-user-guide
description: |
  Creates USER_GUIDE.md for end-users (not developers).
  Use when preparing onboarding material or end-user documentation.
  Do NOT use for API docs (use generate-api-catalog) or technical architecture (use generate-architecture-doc).
  Trigger with "generate user guide", "document for end users", or "create USER_GUIDE.md".
---
```

- [ ] **Step 4: Review `generate-user-guide` content**

Scan for stack-specific references. Already agnostic. No changes needed beyond frontmatter.

- [ ] **Step 5: Commit both**

```bash
git add skills/generate-functional-flows/SKILL.md skills/generate-user-guide/SKILL.md
git commit -m "review: functional-flows and user-guide — update frontmatter, minor fixes"
```

---

### Task 7: Delete `examples/taskflow-api/` and create example structure

**Files:**
- Delete: `examples/taskflow-api/README.md`
- Create: `examples/marketing-platform/01-input.md`
- Create: `examples/fraud-detection-pipeline/01-input.md`

- [ ] **Step 1: Delete taskflow-api**

```bash
rm -rf examples/taskflow-api
```

- [ ] **Step 2: Create marketing-platform input**

Create `examples/marketing-platform/01-input.md`:

```markdown
# Marketing Automation Platform — Input

## The Idea

A marketing automation platform where agencies manage contacts, design email campaigns, set up automations (drip sequences), and track performance (opens, clicks, conversions).

## Actors

- **Agency Admin** — manages users, billing, and platform configuration
- **Marketer** — manages contacts, creates campaigns, configures automations, reviews analytics
- **Viewer/Client** — views campaign reports and contact lists (read-only)

## Key Features

- Contact management with lists, tags, and segmentation
- Email campaign builder with templates, scheduling, and sending
- Automation workflows: trigger-based drip sequences and auto-responses
- Event tracking: opens, clicks, bounces, unsubscribes, conversions
- Analytics dashboards per campaign and aggregate
- Billing based on contact count and email volume

## Business Rules

- Campaigns can only be edited while in "draft" status
- Campaign status flow: draft → scheduled → sending → sent (no skipping)
- Failed sends are retryable without duplicating to already-sent contacts
- Automations can be paused and resumed without losing state
- Contacts can unsubscribe from specific lists or globally
- Billing enforces limits: sending beyond plan quota is blocked, not charged overage

## Constraints

- All timestamps in UTC
- Email sending is async — campaigns with 100k contacts don't block the UI
- Analytics events arrive with up to 5-minute delay (near-real-time, not real-time)
```

- [ ] **Step 3: Create fraud-detection-pipeline input**

Create `examples/fraud-detection-pipeline/01-input.md`:

```markdown
# Fraud Detection Pipeline — Input

## The Idea

A real-time fraud detection system that ingests financial transactions, extracts features, trains ML models, scores transactions in real-time, and triggers alerts for suspicious activity.

## Actors

- **Data Engineer** — manages data pipelines, monitors ingestion health, configures feature stores
- **Data Scientist** — trains and evaluates models, experiments with features, deploys model versions
- **Fraud Analyst** — reviews flagged transactions, confirms/dismisses fraud cases, provides labeled data
- **Admin** — manages users, configures alert thresholds, views system health

## Key Features

- Transaction ingestion from multiple sources (payment processors, bank feeds) via streaming
- Feature engineering: real-time features (velocity, amount deviation) and batch features (historical patterns)
- Model training pipeline: data splitting, training, evaluation, model registry
- Real-time scoring: sub-100ms latency per transaction
- Alert management: flagged transactions queue, analyst review workflow, feedback loop
- Model monitoring: drift detection, performance degradation alerts, A/B testing

## Business Rules

- Transactions scoring above threshold are held for review (not auto-rejected)
- Analyst decisions feed back into training data for next model version
- Model deployment requires approval from at least one Data Scientist
- A/B testing runs for minimum 7 days before a model can be promoted
- Feature store maintains point-in-time correctness (no data leakage)

## Constraints

- Scoring latency must be < 100ms at P99
- Transaction volume: 10,000+ per second at peak
- Model retraining runs daily on previous day's labeled data
- All predictions must be explainable (feature importance per decision)
- Audit trail required for every analyst decision
```

- [ ] **Step 4: Commit**

```bash
git add -A examples/
git commit -m "replace examples: remove taskflow-api, add marketing-platform and fraud-detection inputs"
```

---

### Task 8: Generate `marketing-platform` example — framework + decomposition

**Files:**
- Create: `examples/marketing-platform/02-framework.md`
- Create: `examples/marketing-platform/03-decomposition.md`

- [ ] **Step 1: Apply universal-framework to the marketing platform input**

Use the rewritten `universal-framework` skill mentally against the input in `01-input.md`. Write the output to `examples/marketing-platform/02-framework.md` following the new structured output format defined in Task 1. The output must include: project type identification, all four dimensions (INPUTS, PROCESOS, OUTPUTS, SOPORTE) with categorized items, and the mapping_to_swebok section.

- [ ] **Step 2: Apply swebok-decompose to produce the work breakdown**

Use the rewritten `swebok-decompose` skill mentally against the framework output. Write to `examples/marketing-platform/03-decomposition.md`. Must include: project info, actors, entities, full node list with depends_on, and validation section. Ensure CONSTRUCTION ratio ≥ 40%.

- [ ] **Step 3: Verify consistency**

Check that: every actor operation from `01-input.md` is covered by a node, every business rule maps to a constraint somewhere, CONSTRUCTION ratio passes, no orphan dependencies.

- [ ] **Step 4: Commit**

```bash
git add examples/marketing-platform/
git commit -m "example: marketing-platform — framework analysis and decomposition"
```

---

### Task 9: Generate `marketing-platform` example — specs

**Files:**
- Create: `examples/marketing-platform/04-specs/01-bootstrap.md`
- Create: `examples/marketing-platform/04-specs/02-data-models.md`
- Create: `examples/marketing-platform/04-specs/03-authentication.md`
- Create: `examples/marketing-platform/04-specs/04-contact-management.md`
- Create: `examples/marketing-platform/04-specs/05-campaign-builder.md`
- Create: `examples/marketing-platform/04-specs/06-template-engine.md`
- Create: `examples/marketing-platform/04-specs/07-automation-workflows.md`
- Create: `examples/marketing-platform/04-specs/08-event-tracking.md`
- Create: `examples/marketing-platform/04-specs/09-billing.md`
- Create: `examples/marketing-platform/04-specs/10-notifications.md`

- [ ] **Step 1: Generate spec for each node**

Apply `swebok-generate-spec` to each node from the decomposition. Each spec file contains a single YAML document following the agnostic template from Task 3. Every spec must have: title, purpose, category, input_context (referencing previous specs by name), instructions, constraints, acceptance_criteria, dependencies, handoff, and verification.

- [ ] **Step 2: Generate execution order**

Add to the last spec file (or a separate `execution-order.md`) the execution_order YAML showing which specs can run in parallel.

- [ ] **Step 3: Verify cross-references**

Check that every `handoff` is consumed by a downstream spec's `input_context`. Check that every `dependencies.hard` item exists as a spec title.

- [ ] **Step 4: Commit**

```bash
git add examples/marketing-platform/04-specs/
git commit -m "example: marketing-platform — 10 executable specs with execution order"
```

---

### Task 10: Generate `marketing-platform` example — documentation

**Files:**
- Create: `examples/marketing-platform/05-documentation/ARCHITECTURE.md`
- Create: `examples/marketing-platform/05-documentation/FUNCTIONAL_FLOWS.md`
- Create: `examples/marketing-platform/05-documentation/USER_GUIDE.md`
- Create: `examples/marketing-platform/05-documentation/API_CATALOG.md`

- [ ] **Step 1: Generate ARCHITECTURE.md**

Apply `generate-architecture-doc` to the marketing platform based on the specs. Use the agnostic template from Task 4. Include: overview, tech stack (use generic placeholders like "chosen framework", "chosen database"), component architecture, data model, API contracts, cross-cutting concerns, NFRs, ADRs, glossary.

- [ ] **Step 2: Generate FUNCTIONAL_FLOWS.md**

Apply `generate-functional-flows`. Document all actor flows in plain language: Admin managing users, Marketer creating campaigns, Viewer reading reports. Include state machines for Campaign (draft→scheduled→sending→sent) and Automation (draft→active→paused). Include business rules.

- [ ] **Step 3: Generate USER_GUIDE.md**

Apply `generate-user-guide`. Write for each role: getting started, daily tasks, common scenarios, troubleshooting.

- [ ] **Step 4: Generate API_CATALOG.md**

Apply `generate-api-catalog` with the agnostic template from Task 5. Document interfaces for contacts, campaigns, templates, automations, analytics, billing. Include auth flows (register, login, refresh, revoke).

- [ ] **Step 5: Commit**

```bash
git add examples/marketing-platform/05-documentation/
git commit -m "example: marketing-platform — generated documentation (ARCHITECTURE, FLOWS, USER_GUIDE, API_CATALOG)"
```

---

### Task 11: Generate `marketing-platform` example — README case study

**Files:**
- Create: `examples/marketing-platform/README.md`

- [ ] **Step 1: Write the case study**

Structure:
1. **The Input** — link to `01-input.md`, summarize the idea in 2 sentences
2. **The Process** — which skill was used at each step, in what order
3. **The Outputs** — summary of what was generated: N nodes, N specs, 4 documentation files
4. **Key Observations** — what Blueprint added that a raw prompt wouldn't: structured decomposition, dependency graph, execution order, professional docs
5. **How to Reproduce** — step-by-step commands to reproduce this example with Claude Code

- [ ] **Step 2: Commit**

```bash
git add examples/marketing-platform/README.md
git commit -m "example: marketing-platform — case study README"
```

---

### Task 12: Generate `fraud-detection-pipeline` example — full

**Files:**
- Create: `examples/fraud-detection-pipeline/02-framework.md`
- Create: `examples/fraud-detection-pipeline/03-decomposition.md`
- Create: `examples/fraud-detection-pipeline/04-specs/` (all spec files)
- Create: `examples/fraud-detection-pipeline/05-documentation/` (all doc files)
- Create: `examples/fraud-detection-pipeline/README.md`

- [ ] **Step 1: Apply universal-framework**

Write `02-framework.md`. Project type should be identified as "Machine Learning / Real-time System" (multi-layer: streaming + ML + web app). Use the signals from the rewritten universal-framework.

- [ ] **Step 2: Apply swebok-decompose**

Write `03-decomposition.md`. Expected nodes: Bootstrap, Data Models/Schemas, Auth, Transaction Ingestion, Feature Engineering, Model Training Pipeline, Real-time Scoring Service, Alert & Case Management, Model Monitoring & Drift Detection, Analyst Feedback Loop. Ensure CONSTRUCTION ratio ≥ 40%.

- [ ] **Step 3: Generate specs for each node**

Write one spec file per node in `04-specs/`. Follow the same agnostic format. Include execution order.

- [ ] **Step 4: Generate documentation**

Write ARCHITECTURE.md, FUNCTIONAL_FLOWS.md, USER_GUIDE.md, and API_CATALOG.md (or INTERFACE_CATALOG.md since this system has both APIs and streaming interfaces).

- [ ] **Step 5: Write case study README**

Same structure as marketing-platform. Highlight how the same methodology applied to a completely different project type (ML pipeline vs SaaS).

- [ ] **Step 6: Verify cross-example consistency**

Both examples should use the same skill output formats. If the decomposition YAML looks different between the two, one of them is wrong.

- [ ] **Step 7: Commit**

```bash
git add examples/fraud-detection-pipeline/
git commit -m "example: fraud-detection-pipeline — complete case study (framework, decomposition, specs, docs)"
```

---

## Self-Review

**Spec coverage check:**
- [x] `universal-framework` — output format, deeper types, signals → Task 1
- [x] `swebok-decompose` — agnostic example, multi-layer, frontmatter → Task 2
- [x] `swebok-generate-spec` — agnostic template, spec sequencing, frontmatter → Task 3
- [x] `generate-architecture-doc` — remove Java refs, generic placeholders → Task 4
- [x] `generate-functional-flows` — minor review, frontmatter → Task 6
- [x] `generate-user-guide` — minor review, frontmatter → Task 6
- [x] `generate-api-catalog` — non-REST, auth flows, frontmatter → Task 5
- [x] Delete taskflow-api → Task 7
- [x] Marketing platform example (full) → Tasks 7-11
- [x] Fraud detection example (full) → Task 12

**Placeholder scan:** No TBD/TODO found. All tasks have concrete content or clear instructions.

**Type consistency:** Output format names (project, dimensions, nodes, validation, execution_order) are consistent across tasks 1-3 and examples 8-12.
