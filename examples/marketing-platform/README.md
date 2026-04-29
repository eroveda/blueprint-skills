# Case Study: Marketing Automation Platform

> A complete walkthrough of decomposing a Mailchimp-like marketing platform using Blueprint Skills.

## The Input

A marketing automation platform where agencies manage contacts, design email campaigns, set up automations (drip sequences), and track performance. Three actor roles (Admin, Marketer, Viewer), six core feature areas, and explicit business rules like draft-only editing and quota-based billing. See [01-input.md](01-input.md) for the full description.

## The Process

| Step | Skill Used | Output |
|---|---|---|
| 1 | `universal-framework` | Structured inputs/processes/outputs/support analysis with SWEBOK mapping — [02-framework.md](02-framework.md) |
| 2 | `swebok-decompose` | 10-node dependency graph covering all actors, business rules, and constraints — [03-decomposition.md](03-decomposition.md) |
| 3 | `swebok-generate-spec` | 10 executable specs + execution order — [04-specs/](04-specs/) |
| 4 | `generate-architecture-doc` | IEEE 15289-aligned architecture document — [05-documentation/ARCHITECTURE.md](05-documentation/ARCHITECTURE.md) |
| 5 | `generate-functional-flows` | End-to-end functional flow diagrams — [05-documentation/FUNCTIONAL_FLOWS.md](05-documentation/FUNCTIONAL_FLOWS.md) |
| 6 | `generate-user-guide` | User-facing feature guide by role — [05-documentation/USER_GUIDE.md](05-documentation/USER_GUIDE.md) |
| 7 | `generate-api-catalog` | Complete API endpoint catalog — [05-documentation/API_CATALOG.md](05-documentation/API_CATALOG.md) |

## The Outputs

- **10 nodes** in the decomposition (1 operations, 1 design, 1 security, 7 construction)
- **10 executable specs** with acceptance criteria, plus an execution order file
- **4 professional documentation files** (architecture, flows, user guide, API catalog)
- **Total: 18 files** of structured output from a single project idea

## What Blueprint Added

What this decomposition produces that a raw "build me a Mailchimp clone" prompt wouldn't:

1. **Structured dependency graph** -- specs have explicit dependencies and can be executed in parallel phases
2. **Execution order** -- 4 phases from bootstrap to advanced features, with parallelization within each phase
3. **Acceptance criteria** -- every spec has testable criteria, not vague requirements
4. **Professional documentation** -- IEEE 15289-aligned docs generated from the same decomposition
5. **Stack-agnostic** -- the same specs work with any language or framework

## How to Reproduce

```bash
# In any Claude Code session:

# Step 1: Apply the universal framework
> Use the universal-framework skill to analyze this project: [paste input]

# Step 2: Decompose with SWEBOK
> Use swebok-decompose to create the work breakdown

# Step 3: Generate specs for each node
> Use swebok-generate-spec for each node in the decomposition

# Step 4: Generate documentation
> Use generate-architecture-doc on this project
> Use generate-functional-flows on this project
> Use generate-user-guide on this project
> Use generate-api-catalog on this project
```
