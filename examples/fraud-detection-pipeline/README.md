# Case Study: Fraud Detection Pipeline

> A complete walkthrough of decomposing a real-time ML fraud detection system using Blueprint Skills.

## The Input

A real-time fraud detection pipeline that ingests financial transactions at 10,000+ TPS, computes features, scores transactions with ML models in under 100ms, and routes suspicious activity to human analysts for review. Four actor roles (Data Engineer, Data Scientist, Fraud Analyst, Admin), streaming ingestion, a feature store with point-in-time correctness, and a continuous feedback loop where analyst decisions improve future models. See [01-input.md](01-input.md) for the full description.

## The Process

| Step | Skill Used | Output |
|---|---|---|
| 1 | `universal-framework` | Multi-layer analysis (streaming + ML + web app) with SWEBOK mapping -- [02-framework.md](02-framework.md) |
| 2 | `swebok-decompose` | 10-node dependency graph covering all actors, business rules, and constraints -- [03-decomposition.md](03-decomposition.md) |
| 3 | `swebok-generate-spec` | 10 executable specs + execution order -- [04-specs/](04-specs/) |
| 4 | `generate-architecture-doc` | IEEE 15289-aligned architecture with ML-specific ADRs -- [05-documentation/ARCHITECTURE.md](05-documentation/ARCHITECTURE.md) |
| 5 | `generate-functional-flows` | End-to-end functional flows with state machines for transactions and models -- [05-documentation/FUNCTIONAL_FLOWS.md](05-documentation/FUNCTIONAL_FLOWS.md) |
| 6 | `generate-user-guide` | User-facing guide for all four roles -- [05-documentation/USER_GUIDE.md](05-documentation/USER_GUIDE.md) |
| 7 | `generate-interface-catalog` | REST APIs, streaming interfaces, and CLI tools -- [05-documentation/INTERFACE_CATALOG.md](05-documentation/INTERFACE_CATALOG.md) |

## The Outputs

- **10 nodes** in the decomposition (1 operations, 1 design, 1 security, 7 construction)
- **10 executable specs** with acceptance criteria, plus an execution order file
- **4 professional documentation files** (architecture, flows, user guide, interface catalog)
- **Total: 18 files** of structured output from a single project idea

## A Different Domain, the Same Methodology

This example demonstrates that Blueprint Skills work equally well for an ML pipeline as for a [SaaS web application](../marketing-platform/). The methodology is domain-agnostic:

- The **universal framework** identified this as a multi-layer system (streaming + ML + web app) instead of a single-layer SaaS app -- but the INPUTS/PROCESOS/OUTPUTS/SOPORTE analysis applied identically.
- The **SWEBOK decomposition** produced the same ratio (7/10 construction nodes, 70%) despite covering ML training pipelines, streaming ingestion, and real-time inference instead of CRUD endpoints.
- The **specs** reference domain-specific concerns (point-in-time correctness, scoring latency, A/B testing) but follow the exact same format: instructions, constraints, acceptance criteria, dependencies, handoff.
- The **documentation** adapted naturally: INTERFACE_CATALOG instead of API_CATALOG (because this system has streaming and CLI interfaces alongside REST), state machines for both transactions and model lifecycle, and ADRs addressing ML-specific decisions (feature store design, scoring latency budget, model versioning).

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
> Use generate-interface-catalog on this project
```
