---
name: swebok-generate-spec
description: |
  Converts a SWEBOK node into a complete executable specification (Box).
  Use after swebok-decompose to generate detailed specs ready for code generation.
  Trigger with "generate spec for this node" or "create executable specification".
---

# SWEBOK Generate Spec

You convert work nodes into complete, executable specifications that any AI agent (Claude, Qwen, GPT) can implement.

## Spec Structure

Each spec must include:

```yaml
title: "..."
purpose: "Single sentence — what this spec produces"
category: "DESIGN | CONSTRUCTION | SECURITY | TESTING | OPERATIONS"

input_context: |
  What's available before this spec runs:
  - Classes already created (with FQN paths)
  - Database tables already migrated
  - Configuration already set
  - Specific class/file/package names from previous specs

instructions:
  - "Numbered, concrete, unambiguous step"
  - "Each instruction implementable without external clarification"
  - "Reference exact class names, packages, file paths"

constraints:
  - "Explicit restriction (e.g., 'Must use Supabase JWT')"
  - "Business rule (e.g., 'Status transitions: TODO → IN_PROGRESS → DONE')"
  - "Convention (e.g., 'HTTP 201 for creation, 204 for deletion')"

acceptance_criteria:
  - "Testable verification (e.g., 'POST /tasks returns 201 with task_id')"
  - "Test name (e.g., 'test_cannot_transition_todo_to_done_directly passes')"
  - "Coverage requirement (e.g., 'All endpoints have integration tests')"

dependencies:
  hard:
    - "Previous spec whose output this spec imports (compile dependency)"
  integrates_with:
    - "Spec that connects but isn't required to compile"

handoff: |
  What downstream specs receive from this one:
  - Class names exposed for injection
  - Endpoints available for consumption
  - Tables/data available for queries

verification: "./mvnw test"  # NOT compile — must run tests and pass
```

## Mandatory Rules per Spec

1. **Verification must be `test`, not `compile`** — the spec is not done until tests pass
2. **All code uses the base package** defined in the Bootstrap spec
3. **Documentation is incremental** — append to ARCHITECTURE.md, FUNCTIONAL_FLOWS.md, USER_GUIDE.md
4. **Tests are inside the spec** — unit tests + integration tests for the logic created here
5. **Reference exact class names** from dependency specs in input_context

## Output Format

Single YAML/JSON document per spec. No markdown explanations outside the structure.

## Example

For node "Task CRUD Endpoints":

```yaml
title: "Task CRUD Endpoints"
purpose: "Implement REST endpoints for task lifecycle (POST, GET, PATCH, DELETE) with tenant isolation and role-based access"
category: "CONSTRUCTION"

input_context: |
  Java 21 Quarkus project at base package com.taskflow.
  Task entity exists at com.taskflow.domain.entity.Task with fields:
  id, tenant_id, project_id, title, description, assignee_id, due_date, priority, status.
  TenantContext bean injected via JWT filter from Auth spec.
  AuthorizationService.checkRole(role) available.

instructions:
  - "Create TaskDTO and TaskResponseDTO records in com.taskflow.task.dto"
  - "Implement POST /api/tenants/{tenantId}/tasks with @Valid validation"
  - "Implement GET /api/tenants/{tenantId}/tasks with role-based filtering"
  - "Implement PATCH /api/tenants/{tenantId}/tasks/{id} with status transition validation"
  - "Implement DELETE /api/tenants/{tenantId}/tasks/{id} (TEAM_MEMBER own tasks only)"
  - "Add unit tests for all endpoints in TaskResourceTest.java"
  - "Add integration tests for tenant isolation in TaskTenantIsolationIT.java"
  - "Append endpoints to requests.http with example calls"
  - "Update ARCHITECTURE.md with new resource and DTOs"
  - "Update FUNCTIONAL_FLOWS.md with task lifecycle for Team Member"

constraints:
  - "Must use @TenantContext for tenant_id extraction"
  - "Status transitions must prevent direct TODO → DONE"
  - "Team Members can only see/modify tasks assigned to them"
  - "All datetime in UTC, ISO 8601 format"

acceptance_criteria:
  - "test_create_task_returns_201_with_id passes"
  - "test_cannot_transition_todo_to_done_directly returns 422"
  - "test_team_member_cannot_see_other_users_tasks returns 404"
  - "test_tenant_isolation_returns_404_for_cross_tenant passes"
  - "All tests pass with ./mvnw test"

dependencies:
  hard:
    - "Tenant and User Data Models"
    - "JWT Authentication Filter"
  integrates_with:
    - "Rate Limiting Filter"
    - "Tenant Isolation Filter"

handoff: |
  Exposes:
  - TaskService bean (com.taskflow.task.TaskService)
  - TaskRepository bean (com.taskflow.task.TaskRepository)
  - REST endpoints under /api/tenants/{tenantId}/tasks
  Ready for consumption by Dashboard spec and Reporting spec.

verification: "./mvnw test"
```
