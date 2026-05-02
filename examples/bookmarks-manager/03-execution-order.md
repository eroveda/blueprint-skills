# Execution Order — Bookmarks Manager Web App

| Phase | Spec IDs | Titles | Reason |
|-------|----------|--------|--------|
| 1 | B1, F1 | Backend Bootstrap; Frontend Bootstrap | No dependencies — both layers can be initialized in parallel |
| 2 | B2 | Database Schema, Migrations, FTS5 | Requires B1 (Express + SQLite project exists) |
| 3 | B3 | REST API Contract and Shared Types | Requires B2 (schema shapes inform API contract) |
| 4 | B4, F2 | Input Validation + Security; API Client + Types | B4 requires B3 (Zod schemas from contract); F2 requires F1 + B3 contract finalized |
| 5 | B5, F3 | Bookmark CRUD Endpoints; URL Sanitization | B5 requires B3+B4; F3 requires F2 (types available) |
| 6 | B6, B7, B8, F4 | Search Endpoint; Tag Endpoint; Import Endpoint; Data Hooks | B6/B7/B8 require B5; F4 requires F2+F3 |
| 7 | B9, F5, F6, F8 | Error+Health; BookmarkList+Card; SearchBar+Sidebar; Import Modal | B9 requires B5-B8; F5/F6/F8 require F4 |
| 8 | B10, F7, F9 | Backend Tests; Add/Edit Modal; Pagination+Toggle | B10 requires B8+B9; F7 requires F4+F5; F9 requires F5 |
| 9 | F10 | Frontend Component + Hook Tests | Requires F7+F8+F9 (all components exist) |

## Parallelization Notes

- **Phases 1**: B1 and F1 are fully independent — run in parallel.
- **Phase 4**: B4 and F2 can start as soon as B3 is complete.
- **Phase 6**: B6, B7, B8 are all independent of each other (all depend only on B5) — run in parallel. F4 can run in parallel with B6-B8.
- **Phase 7**: B9, F5, F6, F8 can all run in parallel once their respective dependencies are met.
- **Phase 8**: B10, F7, F9 are independent of each other within the phase.
