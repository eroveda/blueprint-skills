# Example: TaskFlow API

A complete walkthrough of decomposing a multi-tenant task management API using Blueprint Skills.

## Input

```
A multi-tenant task management API for small agencies.
Tech stack: Java 21, Quarkus, PostgreSQL.
Actors:
- Agency Admin who manages users and projects
- Team Member who creates and completes tasks
- Client who views project progress (read-only)

Business rules:
- A task cannot move to done without passing through review
- Only Agency Admin can delete projects
- Team Members only see tasks assigned to them or to their projects

Constraints: Supabase JWT auth, 100 req/min per tenant.
```

## Step 1: Apply universal-framework

```
INPUTS:
- HTTP requests with JWT
- Task/Project payloads

PROCESOS:
- CRUD for projects (Admin)
- CRUD for tasks (Members)
- Status transitions
- Dashboard aggregation (Client)
- User management (Admin)

OUTPUTS:
- HTTP responses (JSON)
- Dashboard data

SOPORTE:
- JWT authentication
- Tenant isolation
- Rate limiting
- Database persistence
- Project bootstrap
```

## Step 2: Apply swebok-decompose

```yaml
nodes:
  - id: 1
    title: "Project Bootstrap and Environment Setup"
    category: OPERATIONS
  - id: 2
    title: "Tenant, User, Project, Task Data Models"
    category: DESIGN
  - id: 3
    title: "JWT Authentication Filter"
    category: SECURITY
  - id: 4
    title: "Tenant Isolation Filter"
    category: SECURITY
  - id: 5
    title: "Project CRUD Endpoints"
    category: CONSTRUCTION
  - id: 6
    title: "Task CRUD Endpoints"
    category: CONSTRUCTION
  - id: 7
    title: "Task Status Workflow"
    category: CONSTRUCTION
  - id: 8
    title: "Client Dashboard Read Endpoints"
    category: CONSTRUCTION
  - id: 9
    title: "User Management Endpoints"
    category: CONSTRUCTION

validation:
  total_nodes: 9
  construction_count: 5
  construction_ratio: "5/9 (55%)" ✅
  bootstrap_present: true ✅
  coverage:
    - "Admin → Project CRUD ✅"
    - "Admin → User CRUD ✅"
    - "Member → Task CRUD ✅"
    - "Member → Task Workflow ✅"
    - "Client → Dashboard READ ✅"
```

## Step 3: Apply swebok-generate-spec to each node

(Each node becomes a complete spec — see [specs/](specs/) directory)

## Step 4: Generate Documentation

After implementation, run documentation skills:

- `generate-architecture-doc` → ARCHITECTURE.md
- `generate-functional-flows` → FUNCTIONAL_FLOWS.md
- `generate-user-guide` → USER_GUIDE.md
- `generate-api-catalog` → API_CATALOG.md

## Validation

This decomposition was validated by generating actual code:

- **With Sonnet**: 50 Java files, BUILD SUCCESS in 20 minutes
- **With Qwen 7B (local)**: 33 Java files in 3 minutes, correct business logic
- **Total cost**: ~$0.12 with Haiku for spec generation, $0 for execution

The same framework worked across 14+ different project types (REST APIs, ETL, mobile, IoT, blockchain, ML, etc.).
