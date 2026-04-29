---
name: swebok-generate-spec
description: |
  Converts a SWEBOK node into a complete executable specification.
  Use after swebok-decompose to generate detailed specs ready for code generation by any AI agent.
  Do NOT use for documentation — use the generate-* skills for that.
  Trigger with "generate spec for this node", "create executable specification", or "spec this node".
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
  - Modules/classes/files already created (with paths)
  - Database tables/collections already created
  - Configuration already set
  - Specific module/file/function names from previous specs

instructions:
  - "Numbered, concrete, unambiguous step"
  - "Each instruction implementable without external clarification"
  - "Reference exact module names, file paths, function signatures"

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
  - Modules/services exposed for consumption
  - Endpoints available for consumption
  - Tables/data available for queries

verification: "run project test suite (e.g., pytest, npm test, go test, ./mvnw test, mix test)"
```

## Mandatory Rules per Spec

1. **Verification must be `test`, not `compile`** — the spec is not done until tests pass
2. **All code follows the project structure** defined in the Bootstrap spec
3. **Documentation is incremental** — append to ARCHITECTURE.md, FUNCTIONAL_FLOWS.md, USER_GUIDE.md
4. **Tests are inside the spec** — unit tests + integration tests for the logic created here
5. **Reference exact module/file/function names** from dependency specs in input_context

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

## Output Format

Single YAML/JSON document per spec. No markdown explanations outside the structure.

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
