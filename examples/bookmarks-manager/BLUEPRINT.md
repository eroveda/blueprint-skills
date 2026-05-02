# BLUEPRINT — Bookmarks Manager Web App

> Master index for the complete project blueprint. Start here.

---

## Project Overview

| Field | Value |
|---|---|
| **Project Name** | Bookmarks Manager Web App |
| **Project Type** | Multi-layer Web Application (REST API + SPA) |
| **Layers** | 2 — Backend API + Frontend SPA |
| **Primary Actor** | Single User (personal, no auth) |
| **Backend Stack** | Node.js ≥ 18, Express ^4, SQLite (better-sqlite3 ^9), Zod ^3 |
| **Frontend Stack** | React ^18, Vite ^5, @tanstack/react-query ^5, Tailwind CSS ^4 |
| **Test Stack** | Jest + supertest (backend), Vitest + Testing Library + MSW (frontend) |

---

## Blueprint Files

| File | Purpose | Lines | Status |
|---|---|---|---|
| [`BLUEPRINT.md`](BLUEPRINT.md) | Master index — this file | — | ✅ |
| [`01-universal-framework-output.md`](01-universal-framework-output.md) | INPUTS/PROCESOS/OUTPUTS/SOPORTE analysis per layer | 295 | ✅ |
| [`02-decomposition-output.md`](02-decomposition-output.md) | SWEBOK work node breakdown (20 nodes, 2 layers) + dependency graph | 276 | ✅ |
| [`03-execution-order.md`](03-execution-order.md) | Build phase table (9 phases, parallel opportunities) | 21 | ✅ |
| [`specs/`](specs/) | 20 executable YAML specs (one per work node) | 1,326 total | ✅ |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | IEEE 15289 architecture doc — modules, data model, ADRs | 438 | ✅ |
| [`FUNCTIONAL_FLOWS.md`](FUNCTIONAL_FLOWS.md) | Plain-language user flows, state machines, business rules | 250 | ✅ |
| [`USER_GUIDE.md`](USER_GUIDE.md) | End-user guide — getting started, tasks, troubleshooting, FAQs | 363 | ✅ |
| [`API_CATALOG.md`](API_CATALOG.md) | Full REST API reference with cURL examples | 555 | ✅ |
| [`TESTING_STRATEGY.md`](TESTING_STRATEGY.md) | Test levels, coverage targets, CI/CD pipeline, verification matrix | 282 | ✅ |

**Spec files** (in `specs/`):

| File | Node | Category |
|---|---|---|
| `B1_backend_bootstrap.yaml` | Backend Bootstrap | OPERATIONS |
| `B2_database_schema.yaml` | DB Schema + FTS5 + Migrations | DESIGN |
| `B3_api_contract.yaml` | REST API Contract + Zod Schemas | DESIGN |
| `B4_security_validation.yaml` | Validation + Sanitization + Upload Security | SECURITY |
| `B5_bookmark_crud.yaml` | Bookmark CRUD Endpoints | CONSTRUCTION |
| `B6_search_filter.yaml` | FTS5 Search + Tag Filter Endpoint | CONSTRUCTION |
| `B7_tags_endpoint.yaml` | Tag Management Endpoint | CONSTRUCTION |
| `B8_import_endpoint.yaml` | Browser Bookmark Import Endpoint | CONSTRUCTION |
| `B9_error_health.yaml` | Error Handling + Health Check | OPERATIONS |
| `B10_backend_tests.yaml` | Backend Integration Test Suite | TESTING |
| `F1_frontend_bootstrap.yaml` | Vite + React Bootstrap | OPERATIONS |
| `F2_api_client_types.yaml` | API Client + TypeScript Types | DESIGN |
| `F3_url_sanitization.yaml` | XSS URL Sanitization + File Validation | SECURITY |
| `F4_data_hooks.yaml` | useBookmarks + useTags + useImport hooks | CONSTRUCTION |
| `F5_bookmark_list_card.yaml` | BookmarkList + BookmarkCard + AppLayout | CONSTRUCTION |
| `F6_search_tag_sidebar.yaml` | SearchBar + TagSidebar Components | CONSTRUCTION |
| `F7_add_edit_modal.yaml` | Add/Edit Bookmark Modal | CONSTRUCTION |
| `F8_import_modal.yaml` | Import Bookmarks Modal | CONSTRUCTION |
| `F9_pagination_view_toggle.yaml` | Pagination + View Toggle | CONSTRUCTION |
| `F10_frontend_tests.yaml` | Frontend Integration + Accessibility Tests | TESTING |

---

## Project Metrics

| Metric | Value |
|---|---|
| Total work nodes | 20 (10 backend + 10 frontend) |
| DESIGN nodes | 4 (B2, B3, F2 + cross-layer) |
| CONSTRUCTION nodes | 10 (B5, B6, B7, B8 + F4, F5, F6, F7, F8, F9) |
| SECURITY nodes | 2 (B4, F3) |
| TESTING nodes | 2 (B10, F10) |
| OPERATIONS nodes | 4 (B1, B9, F1 + implied) |
| **Construction ratio** | **50% (10/20)** ✅ above 40% minimum |
| Executable specs | 20 YAML files |
| Execution phases | 9 (with parallel opportunities in phases 1, 4, 6, 7, 8) |
| REST API endpoints | 8 (POST/GET/GET:id/PUT/DELETE /bookmarks, GET /tags, POST /import, GET /health) |
| Database tables | 5 (bookmarks, tags, bookmark_tags, bookmarks_fts, schema_migrations) |
| Business rules | 14 (documented in FUNCTIONAL_FLOWS.md) |
| Acceptance criteria | ~150 total test scenarios across all specs |
| Architectural decisions (ADRs) | 6 (SQLite, FTS5 triggers, CommonJS, multer memory, optimistic delete, folder→tag) |

---

## Quick Navigation

### For Developers
- **[ARCHITECTURE.md](ARCHITECTURE.md)** — Module breakdown, data schema, REST contracts, ADRs, tech stack versions. Start here before writing code.
- **[specs/](specs/)** — One YAML file per work node. Each contains exact instructions, constraints, acceptance criteria, and the verification command. Execute in the order from `03-execution-order.md`.
- **[03-execution-order.md](03-execution-order.md)** — 9-phase build order. Phases 1, 4, 6, 7, and 8 contain parallelizable work.

### For Product Managers / Stakeholders
- **[FUNCTIONAL_FLOWS.md](FUNCTIONAL_FLOWS.md)** — What the system does in plain language. Actor flows, state machines, and 14 business rules. No code or HTTP details.

### For End Users
- **[USER_GUIDE.md](USER_GUIDE.md)** — Step-by-step instructions for every user task: saving, searching, filtering, importing, editing, and deleting bookmarks. Includes troubleshooting and FAQs.

### For Frontend / API Integrators
- **[API_CATALOG.md](API_CATALOG.md)** — Every REST endpoint documented with full request/response JSON examples, cURL commands, validation rules, error codes, and pagination details.

### For QA / Test Engineers
- **[TESTING_STRATEGY.md](TESTING_STRATEGY.md)** — Test philosophy, 4 test levels (unit/integration/E2E/non-functional), verification matrix, CI/CD pipeline stages, coverage targets, and identified gaps.

### For Architecture / System Design
- **[01-universal-framework-output.md](01-universal-framework-output.md)** — Raw INPUTS/PROCESOS/OUTPUTS/SOPORTE analysis for both layers. The starting point of the blueprint chain.
- **[02-decomposition-output.md](02-decomposition-output.md)** — Full SWEBOK decomposition with dependency graph and recommended build order table.

---

## Standards Compliance

| Standard | Application |
|---|---|
| **IEEE SWEBOK v4.0** | Work node categorization (DESIGN, CONSTRUCTION, SECURITY, TESTING, OPERATIONS) |
| **ISO/IEC/IEEE 15289:2019** | Documentation structure for ARCHITECTURE.md and this blueprint index |
| **ISO/IEC/IEEE 12207:2017** | Software lifecycle process coverage (requirements → design → construction → testing → deployment) |
| **OWASP Top 10** | XSS mitigation (URL sanitization), SQL injection prevention (prepared statements), file upload security |
| **WCAG 2.1 AA** | Accessibility target for all UI components (verified via axe in F10) |

---

## Generation Info

| Field | Value |
|---|---|
| **Date generated** | 2026-05-02 |
| **Blueprint skills used** | `universal-framework` → `swebok-decompose` → `swebok-generate-spec` → `generate-architecture-doc` → `generate-functional-flows` → `generate-user-guide` → `generate-api-catalog` → `generate-testing-strategy` → `generate-blueprint-summary` |
| **Spec mode** | Mode 2 — multiple YAML files (>15 specs) in `specs/` directory |
| **Total blueprint files** | 10 (index + 9 content files) + 20 spec YAMLs = **30 files** |

### Notes and Assumptions

- **Single-user, no auth**: The app assumes local deployment for one user. No authentication layer was designed. If multi-user support is needed, a significant rearchitecture is required (add auth, tenant isolation, user-scoped queries).
- **No favicon fetching**: The `favicon` field is defined in the data model but populating it (via URL fetch) was marked optional in B5 and is not implemented in initial scope. The field always returns `""`.
- **No Playwright E2E**: End-to-end browser tests are recommended (Playwright) but not in initial scope. F10 covers the same flows at integration level via MSW + JSDOM.
- **Versioning**: The API does not use `/v1/` prefix in the initial version. The ARCHITECTURE.md documents a migration path to `/api/v2/` if breaking changes are needed.
- **Production hosting**: In production, Express serves the React build as static files on the same port. No reverse proxy (nginx) is required for basic deployment.
