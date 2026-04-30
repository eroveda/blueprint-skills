# Case Study: Industrial Machine Monitoring System

**The first end-to-end execution of blueprint-skills on a complex multi-layer project.**

---

## TL;DR

Starting from a **3-line project description**, the 7 skills in this plugin generated a complete blueprint for an industrial IoT monitoring system spanning ESP32 firmware, backend API, and web dashboard:

| Output | Lines | Purpose |
|---|---|---|
| `01-universal-framework-output.md` | ~250 | Layered analysis with INPUTS/PROCESOS/OUTPUTS per layer |
| `02-decomposition-output.md` | ~350 | 36 work nodes across 3 layers, 61% CONSTRUCTION ratio |
| `03-SPECS.yaml` | 2,094 | 36 executable specifications in 6 execution phases |
| `04-ARCHITECTURE.md` | 615 | IEEE 15289-compliant architecture (6 ADRs, NFRs, glossary) |
| `05-FUNCTIONAL_FLOWS.md` | 533 | Plain-language flows, 3 state machines, 38 business rules |
| `06-USER_GUIDE.md` | 520 | End-user documentation by role with troubleshooting |
| `07-API_CATALOG.md` | 1,605 | 40 endpoints (REST + WebSocket + MQTT) with cURL examples |
| **Total** | **~5,967** | **Complete project blueprint ready for implementation** |

**Cost**: Generated within a single Claude Code Max session (Sonnet 4.6). After full execution, weekly Sonnet usage was at 14% and current session at 65% — suggesting the equivalent on the Anthropic API would cost approximately **$1–4 USD** for a project of this complexity.

**Time**: All 7 skills executed in approximately 25–35 minutes of model time (one skill alone — `swebok-generate-spec` — took ~9 minutes due to the volume of output).

---

## The Input

The user invoked `/blueprint-skills:universal-framework` in an empty directory and provided this prompt:

> An IoT system with ESP32 sensors monitoring industrial machines.
> Sensors send vibration data via MQTT. Backend stores time series.
> Engineers see anomalies on a web dashboard.

Three sentences. No tech stack specified. No actor list. No constraint list.

See [`00-input.md`](00-input.md) for the exact prompt.

---

## The Process

The skills self-orchestrated. After each skill completed, it suggested the next one to run:

```
1. /blueprint-skills:universal-framework
   → output: layered analysis identifying 3 layers (IoT + Backend + Frontend)
   → suggested next: swebok-decompose

2. /blueprint-skills:swebok-decompose
   → output: 36 nodes (12 firmware + 14 backend + 10 frontend)
   → validation: 22 CONSTRUCTION (61%), 12/12 expected operations covered
   → suggested next: swebok-generate-spec

3. /blueprint-skills:swebok-generate-spec
   → output: SPECS.yaml — 36 executable specs in 6 dependency phases
   → each spec includes: input_context, instructions, constraints,
     acceptance_criteria, dependencies (hard + integrates_with),
     handoff, verification command
   → suggested next: documentation skills (any order)

4. /blueprint-skills:generate-architecture-doc
   → output: ARCHITECTURE.md (IEEE 15289 compliant)

5. /blueprint-skills:generate-functional-flows
   → output: FUNCTIONAL_FLOWS.md (state machines + 38 business rules)

6. /blueprint-skills:generate-user-guide
   → output: USER_GUIDE.md (end-user documentation)

7. /blueprint-skills:generate-api-catalog
   → output: API_CATALOG.md (REST + WebSocket + MQTT reference)
```

No manual edits. No retries beyond the initial plugin installation (see "Reproducibility notes" below).

---

## What Makes This Significant

### 1. The decomposition is rigorous, not generic

`swebok-decompose` didn't just list components — it produced:

- **A dependency graph** between the three layers (MQTT contracts between firmware↔backend, REST/WebSocket between backend↔frontend)
- **Cross-layer contract identification**: which specs produce the message schemas, which consume them
- **A 6-phase execution plan** with parallelism identified (e.g., all 3 bootstraps run in parallel; Phase 4 contains 13 specs that can run in parallel after their dependencies)
- **Validation against expected operations**: 12 actor-entity-operation pairs extracted from the brief, all 12 covered

### 2. The architecture decisions are real engineering, not boilerplate

The ARCHITECTURE.md includes 6 ADRs with concrete trade-offs. Example (ADR-002):

> **Context**: Streaming raw 1kHz × 3-axis × 50 devices would generate ~900KB/s sustained MQTT traffic.
>
> **Decision**: Perform FFT and feature extraction on the ESP32, transmit only summaries (~200 bytes/message at ~1Hz).
>
> **Consequence**: Reduces bandwidth from 900KB/s to ~10KB/s. Enables offline alerting via edge thresholds. Increases firmware complexity. Raw waveforms not retained.

This is the kind of analysis a senior engineer makes during architecture review. Generated from a 3-line prompt.

### 3. The specifications are immediately executable

Each of the 36 specs in `SPECS.yaml` has:

- A **verification command** specific to its sub-component (e.g., `idf.py -T sensor_driver test`, not just `make test`)
- **Acceptance criteria** that can be automated (e.g., "Sample timestamps are monotonically increasing")
- **Hard vs. integrates_with dependencies** — for parallel execution planning
- **Handoff** describing what the next spec receives

A spec runner (Claude Code, Qwen via CLI, or any agentic tool) can execute these without further interpretation.

### 4. The documentation crosses audiences correctly

- `ARCHITECTURE.md` → senior engineers (tech stack, NFRs, ADRs)
- `FUNCTIONAL_FLOWS.md` → product managers (state machines, business rules in plain language)
- `USER_GUIDE.md` → end users (no jargon, troubleshooting, FAQs)
- `API_CATALOG.md` → integrators / frontend devs (cURL examples, error codes)

Same source, four audiences, consistent terminology across all four documents.

### 5. The framework is universal

This was IoT + backend + frontend. The same `swebok-decompose` skill, with no modification, also works for:

- Pure REST APIs
- Data pipelines / ETL
- Mobile apps
- Blockchain / smart contracts
- Machine learning training pipelines
- Research simulations

The `universal-framework` skill applies INPUTS → PROCESOS → OUTPUTS → SOPORTE to identify the structure regardless of project type. SWEBOK categorization (DESIGN / CONSTRUCTION / SECURITY / TESTING / OPERATIONS) is domain-agnostic.

---

## Standards Compliance

The generated outputs are aligned with:

- **IEEE SWEBOK v4.0** (ISO/IEC TR 19759) — work node categorization
- **ISO/IEC/IEEE 12207** — software lifecycle processes
- **ISO/IEC/IEEE 15289** — documentation content (ARCHITECTURE.md structure)
- **ISO/IEC/IEEE 16326** — project planning (execution phases in SPECS.yaml)
- **ISO 10816 / 20816** — mechanical vibration severity (referenced in ARCHITECTURE.md, threshold values)

The standards aren't just listed — they're applied. The vibration thresholds in the spec align with ISO 10816 zones for industrial machinery.

---

## Reproducibility Notes

**Plugin installation:** The plugin was installed locally via symlink during testing. Several installation attempts were made before reaching the working configuration; this case study uses the final, working setup. Public marketplace publication is pending.

**Model**: Claude Sonnet 4.6 (via Claude Code, Max plan).

**Concurrent usage**: At the time of this execution, the same Claude Code Max plan was being used for two unrelated parallel projects (a trading system and a mobile app for inspectors). The 65% session usage and 14% weekly Sonnet usage shown after this execution include all three workloads — actual cost attributable to blueprint-skills is therefore lower than the headline numbers.

**No manual edits**: All output files in this directory are exactly as the skills produced them. They have not been edited for presentation. What you see is what the user got.

**Variance**: Re-running the skills on the same prompt will produce structurally similar output (same number of layers, same SWEBOK categorization, same 6-phase execution plan, same business rules) but with phrasing variations and possibly different exact node counts (likely 30–40 range). The framework is deterministic in structure; the LLM provides flexibility in detail.

---

## How to Reproduce

1. Install `blueprint-skills` (see main repo [README](../../README.md))
2. Open Claude Code in any empty directory
3. Run `/blueprint-skills:universal-framework`
4. When prompted, provide a 1–3 sentence project description
5. Follow the skill chain as suggested by each output

For comparable IoT input, use the prompt in [`00-input.md`](00-input.md).

---

## Files in this Case Study

| File | What it is |
|---|---|
| [`00-input.md`](00-input.md) | Original 3-sentence prompt provided to the first skill |
| [`01-universal-framework-output.md`](01-universal-framework-output.md) | Output of `universal-framework`: layered analysis with dimensions and SWEBOK mapping |
| [`02-decomposition-output.md`](02-decomposition-output.md) | Output of `swebok-decompose`: 36 nodes, dependency graph, validation |
| [`03-SPECS.yaml`](03-SPECS.yaml) | Output of `swebok-generate-spec`: 36 executable specifications |
| [`04-ARCHITECTURE.md`](04-ARCHITECTURE.md) | Output of `generate-architecture-doc`: IEEE 15289 architecture document |
| [`05-FUNCTIONAL_FLOWS.md`](05-FUNCTIONAL_FLOWS.md) | Output of `generate-functional-flows`: flows, state machines, business rules |
| [`06-USER_GUIDE.md`](06-USER_GUIDE.md) | Output of `generate-user-guide`: end-user documentation |
| [`07-API_CATALOG.md`](07-API_CATALOG.md) | Output of `generate-api-catalog`: full API reference |

---

## What's Not Included (Yet)

This case study captures the **specification phase**. The next phase — generating the actual code from these specs and verifying it compiles — is the subject of a future case study.

Independent prior work has validated that specs of similar structure produced by earlier prototypes can be executed by:

- Claude Code (Sonnet) — generating compileable Java/Quarkus projects from REST API specs
- Qwen 2.5-coder 7B (locally via Ollama) — generating ~33 Java files from micro-specs in 3 minutes with zero API cost

Reproducing those results from these IoT specs is the next milestone.

---

## Feedback

This is the v0.1 of blueprint-skills. The skills work as shown above, but there's room to improve:

- Splitting `SPECS.yaml` into one file per spec (for easier consumption by spec runners)
- A `BLUEPRINT.md` summary document tying all the outputs together
- A `TESTING_STRATEGY.md` consolidating the acceptance criteria across specs

Issues, suggestions, and pull requests welcome at [github.com/eroveda/blueprint-skills](https://github.com/eroveda/blueprint-skills).
