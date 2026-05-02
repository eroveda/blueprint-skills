---
name: swebok-decompose
description: |
  Breaks down a project description into IEEE SWEBOK-categorized work nodes.
  Use after universal-framework (or directly for straightforward projects) to create the work breakdown.
  Do NOT use for documentation generation or spec creation — those have their own skills.
  Trigger with "decompose this project", "break down using SWEBOK", or "create work breakdown".
---

# SWEBOK Decompose

You decompose software project descriptions into work nodes using IEEE SWEBOK v4.0 taxonomy.

## Universal Framework

Every project has four dimensions. Identify them from the input:

```
INPUTS    → What enters the system?    (data, requests, events, sensors)
PROCESOS  → What does the system do?   (transform, validate, calculate)
OUTPUTS   → What does the system produce? (responses, reports, models)
SOPORTE   → What sustains the system?  (auth, config, storage, deploy)
```

## SWEBOK Categories

Classify each node into ONE of:

- **DESIGN** — Data models, schemas, entity relationships, API contracts
- **CONSTRUCTION** — Services, endpoints, business logic, workflows, CRUD operations
- **SECURITY** — Authentication, authorization, data isolation, access control
- **TESTING** — Test suites, integration tests, stress/load tests
- **OPERATIONS** — Configuration, deployment, monitoring, CI/CD

## Critical Rules

1. **CONSTRUCTION nodes must dominate** — they represent what the system DOES
2. **One CONSTRUCTION node per actor-entity-operation group**
3. **Bootstrap is mandatory** — first node is always OPERATIONS for project setup
4. **No invented features** — every node must trace back to the input
5. **CONSTRUCTION ratio must be ≥ 40%** of total nodes

## Multi-Layer Projects

For projects with multiple layers (identified by `universal-framework`):

1. Generate a **separate node list per layer**
2. Each layer has its own Bootstrap node
3. Cross-layer dependencies are marked in `depends_on` with the format `"layer:node_id"`
4. Each layer must independently satisfy the CONSTRUCTION ratio ≥ 40%

## Output Format

For each project, produce:

```yaml
project:
  name: "..."
  type: "API REST | Mobile | ETL | IoT | Blockchain | etc"

actors:
  - name: "..."
    operations: ["..."]

entities:
  - name: "..."
    fields: ["..."]

nodes:
  - id: "1"
    title: "Project Bootstrap and Environment Setup"
    category: "OPERATIONS"
    purpose: "..."
    depends_on: []

  - id: "2"
    title: "..."
    category: "DESIGN | CONSTRUCTION | SECURITY | TESTING | OPERATIONS"
    purpose: "..."
    depends_on: ["1"]

validation:
  total_nodes: N
  construction_count: M
  construction_ratio: "M/N (XX%)"
  expected_operations: ["actor → entity → operation"]
  coverage: "X/Y operations covered"
```

## Anti-Patterns to Avoid

- ❌ Generating only infrastructure nodes (auth, isolation, filters) without CRUD endpoints
- ❌ Vague titles like "Manage System" or "Handle Everything"
- ❌ Splitting one operation across multiple nodes (over-decomposition)
- ❌ Mixing categories in a single node
- ❌ Forgetting the Bootstrap node

## Output

Save the complete decomposition to disk so it persists beyond the conversation:

1. **Write the file** `02-decomposition-output.md` in the user's current working directory
2. **File content must include:**
   - The complete YAML block (`project`, `actors`, `entities`, `nodes`, `validation`)
   - The dependency graph showing node relationships
   - A recommended build order table (phases with parallelizable nodes)
   - A closing suggestion: *"Next step: run `/blueprint-skills:swebok-generate-spec` to generate executable specifications for all nodes."*
3. **After saving**, confirm to the user: *"Saved decomposition to `02-decomposition-output.md`"*

This file is consumed by `swebok-generate-spec` in the next stage.

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
    - "Admin → Billing → configure"
    - "Marketer → Contacts → CRUD"
    - "Marketer → Campaigns → create/schedule/send"
    - "Marketer → Automations → create/configure"
    - "Marketer → Analytics → view"
    - "Viewer → Reports → view"
  coverage: "7/7 operations covered"
```
