---
name: swebok-decompose
description: |
  Breaks down a project description into IEEE SWEBOK-categorized work nodes.
  Use when starting a new project, planning architecture, or estimating scope.
  Trigger with "decompose this project", "break down using SWEBOK", or "plan with SWEBOK".
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

## Example

Input: "Multi-tenant task management API with admin, member, and client roles"

Output:
```yaml
nodes:
  - id: "1"
    title: "Project Bootstrap and Environment Setup"
    category: "OPERATIONS"
  - id: "2"
    title: "Tenant and User Data Models"
    category: "DESIGN"
  - id: "3"
    title: "JWT Authentication Filter"
    category: "SECURITY"
  - id: "4"
    title: "Project CRUD Endpoints"
    category: "CONSTRUCTION"
  - id: "5"
    title: "Task CRUD Endpoints"
    category: "CONSTRUCTION"
  - id: "6"
    title: "Task Status Workflow"
    category: "CONSTRUCTION"
  - id: "7"
    title: "Client Read-Only Dashboard"
    category: "CONSTRUCTION"
  - id: "8"
    title: "User Management Endpoints"
    category: "CONSTRUCTION"

validation:
  construction_ratio: "5/8 (63%)"
  coverage: "5/5 operations covered"
```
