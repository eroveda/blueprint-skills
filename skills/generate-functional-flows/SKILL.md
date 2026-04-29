---
name: generate-functional-flows
description: |
  Creates FUNCTIONAL_FLOWS.md describing user-facing flows in plain language.
  Use when documenting for non-technical stakeholders (managers, PMs, clients).
  Trigger with "generate functional flows" or "document user flows".
---

# Generate Functional Flows Documentation

You create FUNCTIONAL_FLOWS.md that describes what the system does in language any business person can understand. NO technical jargon. NO code. NO HTTP details.

## Document Structure

```markdown
# Functional Flows — [Project Name]

## Actors

| Actor | Role | Permissions |
|---|---|---|
| ... | ... | ... |

## Flows by Actor

### [Actor Name]

#### Flow: [Flow name in plain language]

**Goal**: What the actor wants to achieve

**Steps**:
1. The actor [does something]
2. The system [validates/calculates/etc.]
3. The actor sees [result]
4. If [error condition], the actor sees [error message]

**Outcome**: What's been accomplished

**Possible errors**:
- Error 1 with how the user recovers
- Error 2 with how the user recovers

(Repeat for each flow)

## Cross-Actor Flows

Flows involving multiple actors:

### Flow: [Name]
- Actor A initiates by [action]
- Actor B receives notification and [responds]
- System updates [state]
- Both see [final state]

## State Machines

For entities with lifecycle states:

### [Entity] State Machine
- Initial state: ...
- Transitions:
  - From [state] → [state] (triggered by [actor action])
  - From [state] → [state] (triggered by [actor action])
- Forbidden transitions:
  - [state] cannot go directly to [state] — must pass through [state]

## Business Rules

Numbered list of rules that govern the system:

1. Only [actor] can [action]
2. [Entity] cannot be [action] until [condition]
3. [Validation rule in plain language]
```

## Style Rules

- ✅ "The Admin creates a new project"
- ❌ "POST /api/projects with admin role"

- ✅ "The system rejects the request and shows: 'You can only edit your own tasks'"
- ❌ "Returns HTTP 403 Forbidden"

- ✅ "A task moves from 'In Progress' to 'Review' when the team member submits it"
- ❌ "PATCH /tasks/{id} sets status from IN_PROGRESS to REVIEW"

## How to Generate

1. Identify all actors from auth/role configuration
2. For each actor, list the operations they can perform
3. For each operation, describe it as a flow in plain language
4. Identify state machines (entities with status fields)
5. Extract business rules from validation logic
6. Save as FUNCTIONAL_FLOWS.md

## Quality Checklist

- [ ] Every actor in the system is documented
- [ ] Every endpoint has a corresponding flow
- [ ] State machines are diagrammed in plain text
- [ ] Business rules are numbered and unambiguous
- [ ] No technical terms (HTTP, JSON, JWT, SQL)
- [ ] Error messages match actual UI/API responses
- [ ] A non-technical person can read end-to-end and understand the system
