# Blueprint Skills

> Domain decomposition + professional documentation skills for Claude Code.
> Generate IEEE SWEBOK-aligned specs and complete project documentation automatically.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude_Code-Skills-blue)](https://claude.com)

## Why Blueprint?

Most AI coding tools jump straight to code generation. The result: working code that lacks structure, documentation, tests, and maintainability.

Blueprint Skills add the **missing layer between idea and execution**:

- 🧱 **Domain decomposition** — breaks any project into the right components using IEEE SWEBOK taxonomy
- 📐 **Universal framework** — INPUTS → PROCESOS → OUTPUTS → SOPORTE works for any software type (APIs, ETL, mobile, IoT, blockchain, ML)
- 📚 **Professional documentation** — auto-generates ARCHITECTURE.md, FUNCTIONAL_FLOWS.md, USER_GUIDE.md, API_CATALOG.md
- 🔌 **Complementary** — works alongside [Superpowers](https://github.com/obra/superpowers) and other Claude Code workflows

## Installation

```bash
# Add the marketplace
claude plugin marketplace add eroveda/blueprint-skills

# Install the plugin
claude plugin install blueprint-skills@blueprint-skills
```

Or clone manually:

```bash
git clone https://github.com/eroveda/blueprint-skills.git ~/.claude/plugins/blueprint-skills
```

## Skills Included

### Decomposition

| Skill | What it does |
|---|---|
| `swebok-decompose` | Breaks a project description into IEEE SWEBOK categorized nodes (DESIGN, CONSTRUCTION, SECURITY, TESTING, OPERATIONS) |
| `swebok-generate-spec` | Converts each node into an executable specification with instructions, constraints, and acceptance criteria |
| `universal-framework` | Applies INPUTS → PROCESOS → OUTPUTS → SOPORTE to any software type |

### Documentation

| Skill | What it does |
|---|---|
| `generate-architecture-doc` | Creates ARCHITECTURE.md following IEEE 15289 |
| `generate-functional-flows` | Extracts actor → entity → operation flows in plain language |
| `generate-user-guide` | Creates USER_GUIDE.md from API/features |
| `generate-api-catalog` | Maps endpoints with request/response examples |

## Quick Start

```bash
# In any Claude Code session:
> Use the swebok-decompose skill to break down this project:
> "A multi-tenant task management API with admin, member, and client roles"
```

Claude will return:
- A SWEBOK-categorized list of components
- Dependency graph
- Suggested execution order
- Coverage validation against expected operations

## Compatible With

✅ **Superpowers** — Blueprint adds *what to build*, Superpowers handles *how to build it*. Perfect together.
✅ **Claude Code** — native plugin format
✅ **Cursor, Codex CLI, Gemini CLI** — skills are portable

## Standards

Blueprint is aligned with:

- **SWEBOK v4.0** (ISO/IEC TR 19759) — Software Engineering Body of Knowledge
- **ISO/IEC/IEEE 12207** — Software Life Cycle Processes
- **ISO/IEC/IEEE 15289** — Documentation content standards
- **ISO/IEC/IEEE 16326** — Project planning standards

## Validated Case Study

One complete walkthrough generated end-to-end with the seven skills. No manual edits were applied to the outputs.

### [IoT Machine Monitoring](examples/iot-machine-monitoring/)

**Industrial vibration monitoring with ESP32 sensors, MQTT backend, and a web dashboard.**

From a 3-sentence prompt, the skills produced:

- **36 work nodes** across 3 layers (firmware + backend + frontend), 22 CONSTRUCTION (61% ratio)
- **36 executable specs** in 6 execution phases with parallelization identified
- **IEEE 15289-compliant ARCHITECTURE.md** (615 lines) with 6 ADRs, NFRs, and a tech glossary
- **Plain-language FUNCTIONAL_FLOWS.md** (533 lines) with 3 state machines and 38 numbered business rules
- **End-user USER_GUIDE.md** (520 lines) with role-based onboarding, troubleshooting, and FAQs
- **API_CATALOG.md** (1,605 lines) covering 40 endpoints across REST, WebSocket, and 5 MQTT topics

Total: **5,367 lines of professional documentation** from a 3-sentence input. Standards referenced include IEEE SWEBOK v4.0, ISO/IEC/IEEE 12207, ISO/IEC/IEEE 15289, and ISO 10816 (industrial vibration severity).

→ [Read the full case study](examples/iot-machine-monitoring/README.md)

### Coming Soon

Additional case studies validating the framework on simpler (REST APIs, CRUD SaaS) and different (ETL pipelines, mobile apps) project types are planned for v0.2. Contributions from real-world usage are welcome via pull requests.

## Philosophy

> "Stop generating bad code. Start specifying good code."

The cheapest bug is the one you prevent in the spec. Blueprint Skills enforce structure *before* code generation, so AI agents (Claude, Qwen, GPT) produce maintainable software from day one.

## Roadmap

- [x] v0.1 — Core decomposition skills + documentation skills
- [ ] v0.2 — Subagents (builder, tester, reviewer) for orchestration
- [ ] v0.3 — MCP server for persistent project context
- [ ] v0.4 — CLI tool for standalone execution

## Contributing

Pull requests welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE)

## Author

Built by [eroveda](https://github.com/eroveda) based on lessons from building [openFactory](https://github.com/eroveda/openfactory-api) — a complete pre-production engine for software.
