---
name: generate-testing-strategy
description: |
  Generates TESTING_STRATEGY.md — a consolidated testing strategy document
  that aggregates the acceptance criteria from all executable specs into
  a global testing approach (unit, integration, E2E, load, security).

  Use this skill AFTER swebok-generate-spec has produced 03-SPECS.yaml
  (or the specs/ directory).

  ## When to invoke this skill

  - After all specs have been generated
  - When the user asks for a "testing strategy", "QA plan", "test plan",
    or "TESTING_STRATEGY.md"
  - When planning the testing phase of a project

  ## When NOT to invoke

  - Before specs exist
  - For projects without acceptance criteria defined
---

# Generate Testing Strategy

This skill produces TESTING_STRATEGY.md — a consolidated testing approach
extracted and synthesized from all executable specs.

## Approach

1. Read the source specs:
   - If `03-SPECS.yaml` exists → parse the single file
   - If `specs/` directory exists → read all spec files

2. For each spec, extract:
   - Acceptance criteria
   - Verification command
   - Category (TESTING specs get special treatment)
   - Dependencies (to identify integration boundaries)

3. Categorize tests into 4 levels:
   - **Unit tests**: from acceptance_criteria of CONSTRUCTION specs
   - **Integration tests**: from specs marked TESTING + cross-spec acceptance
   - **End-to-end tests**: complete user flows
   - **Non-functional tests**: load, security, accessibility, performance

4. Generate TESTING_STRATEGY.md with structure below.

## Output

You MUST save the strategy as a file.

**Filename**: `TESTING_STRATEGY.md` (exact name, uppercase, with underscore)
**Format**: Markdown
**Location**: Project root directory (cwd)

The file MUST contain these sections:

### Testing Philosophy
- Approach (TDD, BDD, etc.)
- Coverage targets
- Tools and frameworks recommended (per the tech stack in ARCHITECTURE.md)

### Test Levels

#### 1. Unit Tests
- Scope: individual functions, classes, modules
- Coverage target: 80%+ on business logic
- Tools: based on tech stack
- Source: acceptance criteria from CONSTRUCTION specs
- List: enumerate which specs contribute unit tests

#### 2. Integration Tests
- Scope: interactions between modules, database access, external APIs
- Coverage target: critical paths (100%)
- Tools: based on tech stack
- Source: specs marked TESTING + cross-spec dependencies

#### 3. End-to-End Tests
- Scope: complete user journeys
- Coverage target: top 5 user flows
- Tools: Playwright, Cypress, etc. (based on stack)
- Source: extracted from FUNCTIONAL_FLOWS.md flows

#### 4. Non-Functional Tests
- Performance/Load (NFRs from ARCHITECTURE.md)
- Security (penetration, OWASP Top 10)
- Accessibility (WCAG 2.1 if applicable)
- Compatibility (browsers, devices)

### Test Data Strategy
- Fixtures vs factories vs seeds
- Test database approach
- Mocking external services

### CI/CD Integration
- Where each test level runs (PR, main, nightly)
- Failure handling
- Coverage reporting

### Verification Matrix
A table mapping each spec to its tests:

| Spec ID | Spec Name | Unit | Integration | E2E | Verification Command |
|---------|-----------|------|-------------|-----|----------------------|
| B1      | Bootstrap | -    | ✓           | -   | npm test             |
| B4      | Auth      | ✓    | ✓           | ✓   | npm test:auth        |

### Risks and Gaps
- Specs without acceptance criteria
- Integration boundaries not covered
- External dependencies that need contract tests

After writing the file, confirm to the user with:
- Absolute path to TESTING_STRATEGY.md
- Total test scenarios identified
- Coverage gaps (if any)

DO NOT:
- Invent acceptance criteria not in source specs
- Recommend tools incompatible with the documented tech stack
- Skip generation if some specs lack acceptance criteria (note them as gaps)
