# Testing Strategy — Bookmarks Manager Web App

## Testing Philosophy

The project follows a **bottom-up integration testing** approach: business logic is verified through real database interactions (no mocking of the DB layer), while external HTTP calls in the frontend are mocked via MSW (Mock Service Worker). The goal is maximum confidence at minimum test fragility.

- **Backend**: Tests run against a real in-memory SQLite database. No DB mocking. Supertest drives HTTP layer without starting a real server.
- **Frontend**: Components and hooks are tested with React Testing Library + MSW intercepting API calls. No shallow rendering.
- **Principle**: A passing test suite must imply the feature works end-to-end, not just that code compiled.

**Coverage targets**:

| Layer | Type | Target |
|---|---|---|
| Backend | Statement coverage (Jest) | ≥ 80% |
| Backend | Critical paths (CRUD, import, search) | 100% |
| Frontend | Statement coverage (Vitest v8) | ≥ 80% |
| Frontend | User-facing interaction flows | 100% of documented flows |

---

## Tools and Frameworks

| Layer | Purpose | Tool |
|---|---|---|
| Backend | Test runner | Jest (node:test compatible) |
| Backend | HTTP test client | supertest |
| Backend | Test database | SQLite `:memory:` (better-sqlite3) |
| Frontend | Test runner | Vitest |
| Frontend | Component testing | @testing-library/react |
| Frontend | User interaction simulation | @testing-library/user-event |
| Frontend | API mocking | MSW (msw/node in tests, msw/browser in dev) |
| Frontend | Accessibility | vitest-axe / @axe-core/react |
| E2E (future) | Browser automation | Playwright (recommended) |
| Load testing | Throughput validation | autocannon (local CLI) |

---

## Test Levels

### 1. Unit Tests

**Scope**: Individual functions, pure utilities, module-level logic.
**Coverage target**: 80%+ statement coverage on `src/` (backend) and `src/` (frontend).
**Tools**: Jest (backend), Vitest (frontend).

Unit tests are embedded within each construction and security spec. They verify the function/module in isolation using the real database (backend) or MSW mocks (frontend).

**Specs contributing unit tests**:

| Spec | What is unit-tested |
|---|---|
| B4 | `validateBody`, `validateQuery`, `sanitizeUrl`, `uploadMiddleware` |
| B5 | `insertBookmark`, `getBookmarkById`, `updateBookmark`, `deleteBookmark`, `syncTags` |
| B6 | `searchBookmarks` (FTS5, tag filter, pagination, sort) |
| B7 | `getTagCounts` |
| B8 | `parseNetscapeHTML`, `parseJSONBookmarks`, `detectAndParse` |
| B9 | Error handler mapping (ZodError, SQLITE_CONSTRAINT_UNIQUE, multer errors, unknown) |
| F2 | `getBookmarks`, `createBookmark`, API error interceptor, tag param serialization |
| F3 | `sanitizeUrl` (all dangerous schemes), `validateBookmarkFile` (extension, size) |
| F4 | `useBookmarks` (debounce, optimistic delete), `useTags` |
| F5 | `BookmarkCard` (href sanitization, delete, tag click), `BookmarkList` (skeletons, empty) |
| F6 | `SearchBar` (controlled input, clear), `TagSidebar` (multi-select, clear) |
| F7 | `AddEditBookmarkModal` (add mode, edit mode, validation, tag input) |
| F8 | `ImportModal` (file validation, states), `ImportResultSummary` |
| F9 | `Pagination` (boundary states, ellipsis), `ViewToggle` (localStorage) |

---

### 2. Integration Tests

**Scope**: Interactions between modules — HTTP layer ↔ database ↔ business logic.
**Coverage target**: 100% of API endpoint × scenario combinations.
**Tools**: Jest + supertest (backend), Vitest + MSW + React Testing Library (frontend).

Integration tests are concentrated in the two TESTING specs (B10, F10), but every CONSTRUCTION spec also includes integration-style tests embedded in its own test file.

**Backend integration test suites** (`tests/` directory):

| Test File | Coverage Area |
|---|---|
| `tests/bookmarks.test.js` | Bookmark CRUD lifecycle, duplicate URL, tag sync on update (spec B5) |
| `tests/search.test.js` | FTS5 search, tag AND filter, pagination, sort order, empty results (spec B6) |
| `tests/tags.test.js` | Tag list shape, count accuracy, cascade on delete, no duplicate tags (spec B7) |
| `tests/import.test.js` | Chrome HTML, Firefox HTML, JSON import; deduplication; mixed URLs; errors (spec B8) |
| `tests/errorHandler.test.js` | ZodError → 400, constraint → 409, 404 catch-all, health probe (spec B9) |
| `tests/integration/api.test.js` | Full lifecycle: create → search → update → delete; all route × scenario combinations (spec B10) |

**Frontend integration test suites** (`src/test/` directory):

| Test File | Coverage Area |
|---|---|
| `src/api/bookmarks.test.ts` | API client functions, error envelope, tag param serialization (spec F2) |
| `src/hooks/useBookmarks.test.tsx` | Hook state, debounce, optimistic delete, cache invalidation (spec F4) |
| `src/test/integration/App.test.tsx` | Full app flows: add, edit, delete, search, filter, import, paginate (spec F10) |

---

### 3. End-to-End Tests

**Scope**: Complete user journeys through the real running application (browser + backend + SQLite).
**Coverage target**: Top 5 user flows from FUNCTIONAL_FLOWS.md.
**Tools**: Playwright (recommended for future implementation — not in initial scope).
**Status**: Not included in Phase 1 (F10 covers these flows at the integration level via MSW).

**Recommended E2E scenarios** (for future Playwright suite):

| Priority | Flow | Steps |
|---|---|---|
| 1 | Save and find a bookmark | Add bookmark → search by keyword → verify appears |
| 2 | Tag-based organization | Add 3 bookmarks with tags → filter by tag → verify count |
| 3 | Browser import | Export from Chrome → import file → verify bookmarks + tags imported |
| 4 | Edit and delete | Add → edit title → verify updated → delete → verify gone |
| 5 | Combined search + filter | Add bookmarks → select tag + type search → verify AND results |

---

### 4. Non-Functional Tests

#### Performance / Load

| Scenario | Target | Tool | When |
|---|---|---|---|
| API response time (P95) | < 50ms per request | autocannon | Manual, pre-release |
| Import 10,000 bookmarks | < 5 seconds | Integration test with large fixture | CI nightly |
| FTS5 search with 50,000 bookmarks | < 100ms | Seeded SQLite fixture | Manual |

Run locally with:
```bash
npx autocannon -c 10 -d 10 http://localhost:3000/api/bookmarks
```

#### Security

Covered by spec B4 (SECURITY) and F3 (SECURITY):

| Test | Where tested | Verification |
|---|---|---|
| `javascript:` URL rejected at API | `tests/errorHandler.test.js` | POST /api/bookmarks with `url=javascript:alert(1)` → 400 |
| `javascript:` URL rendered as `about:blank` | `src/utils/sanitizeUrl.test.ts` | Unit test on sanitizeUrl |
| File type restriction | `tests/import.test.js` | .exe upload → 422 |
| File size limit | `tests/import.test.js` | 11MB file → 413 |
| SQL injection | Structural — prepared statements enforced | Code review / no raw string queries |
| CORS | `tests/errorHandler.test.js` | OPTIONS request returns correct CORS headers |

#### Accessibility

Covered by spec F10 (TESTING):

| Component | Tool | Standard |
|---|---|---|
| AppLayout | vitest-axe | WCAG 2.1 AA |
| BookmarkCard | vitest-axe | WCAG 2.1 AA |
| AddEditBookmarkModal | vitest-axe | WCAG 2.1 AA |
| ImportModal | vitest-axe | WCAG 2.1 AA |

Run with: `cd frontend && npm test -- a11y`

#### Browser Compatibility

The app uses standard modern browser APIs (Fetch, File API, localStorage, CSS Grid). Target browsers:
- Chrome / Edge ≥ 110
- Firefox ≥ 110
- Safari ≥ 16

No IE11 support. Vite build targets `es2020` by default.

---

## Test Data Strategy

### Backend

- **In-memory SQLite**: Every test suite creates a fresh `Database(':memory:')` instance and runs migrations. No shared state between test files.
- **Fixtures**: HTML and JSON bookmark export files in `tests/fixtures/` — `chrome_bookmarks.html`, `firefox_bookmarks.html`, `custom.json`.
- **Seed helpers**: Inline `insertBookmark` calls within each test. No shared seed scripts.
- **No mocking of DB layer**: Tests hit the real SQLite engine. better-sqlite3 in-memory mode is fast enough (~1ms per operation) for test suites.

### Frontend

- **MSW handlers**: Defined in `src/test/mocks/handlers.ts`. Return realistic fixture data for all API routes.
- **Per-test overrides**: Use `server.use(...)` inside individual tests to simulate error states (500, 409, etc.).
- **QueryClient isolation**: Each test creates a fresh `QueryClient` with `gcTime: 0` to prevent cache leakage between tests.
- **No real network calls**: MSW intercepts all axios requests in test environment.

---

## CI/CD Integration

### Recommended Pipeline Stages

```
PR check:
  - npm test (backend)                    # all backend test files
  - cd frontend && npm test               # all frontend tests
  - cd frontend && npm run build          # verify production build succeeds

Main branch merge:
  - All PR checks
  - cd frontend && npm test -- --coverage # coverage report
  - npm test -- --coverage               # backend coverage report

Nightly:
  - Large-dataset import test (10,000 bookmarks fixture)
  - autocannon load test
  - Playwright E2E suite (when implemented)
```

### Failure Handling

- Any test failure blocks merge.
- Coverage below target (80%) is a warning, not a blocker (adjust to blocker when baseline is established).
- Accessibility violations in axe tests block merge.

### Coverage Reporting

Backend:
```bash
npm test -- --coverage
# Outputs: coverage/ directory, lcov.info for CI upload
```

Frontend:
```bash
cd frontend && npm test -- --coverage
# Uses Vitest v8 provider, outputs coverage/
```

---

## Verification Matrix

| Spec ID | Spec Name | Unit | Integration | E2E | Verification Command |
|---------|-----------|:----:|:-----------:|:---:|----------------------|
| B1 | Backend Bootstrap | - | ✓ | - | `node src/index.js` |
| B2 | Database Schema + FTS5 | - | ✓ | - | `node src/index.js` |
| B3 | REST API Contract | - | ✓ | - | `curl /health` |
| B4 | Input Validation + Security | ✓ | ✓ | - | `npm test -- --testPathPattern=security` |
| B5 | Bookmark CRUD | ✓ | ✓ | - | `npm test -- --testPathPattern=bookmarks` |
| B6 | Search + Tag Filter | ✓ | ✓ | - | `npm test -- --testPathPattern=search` |
| B7 | Tag Endpoint | ✓ | ✓ | - | `npm test -- --testPathPattern=tags` |
| B8 | Import Endpoint | ✓ | ✓ | - | `npm test -- --testPathPattern=import` |
| B9 | Error Handling + Health | ✓ | ✓ | - | `npm test -- --testPathPattern=errorHandler` |
| B10 | Backend Integration Tests | - | ✓ | - | `npm test` |
| F1 | Frontend Bootstrap | - | ✓ | - | `cd frontend && npm run dev` |
| F2 | API Client + Types | ✓ | ✓ | - | `cd frontend && npm test` |
| F3 | URL Sanitization | ✓ | - | - | `cd frontend && npm test -- sanitizeUrl validateFile` |
| F4 | Data Hooks | ✓ | ✓ | - | `cd frontend && npm test -- hooks` |
| F5 | BookmarkList + Card | ✓ | ✓ | - | `cd frontend && npm test -- BookmarkCard BookmarkList` |
| F6 | SearchBar + TagSidebar | ✓ | ✓ | - | `cd frontend && npm test -- SearchBar TagSidebar` |
| F7 | Add/Edit Modal | ✓ | ✓ | ✓* | `cd frontend && npm test -- AddEditBookmarkModal` |
| F8 | Import Modal | ✓ | ✓ | ✓* | `cd frontend && npm test -- ImportModal` |
| F9 | Pagination + ViewToggle | ✓ | ✓ | - | `cd frontend && npm test -- Pagination ViewToggle` |
| F10 | Frontend Integration Tests | - | ✓ | ✓* | `cd frontend && npm test` |

> ✓* = covered at integration level via MSW (not full browser E2E — Playwright suite recommended for future).

---

## Risks and Gaps

### Gaps

| Gap | Impact | Mitigation |
|---|---|---|
| No real Playwright E2E suite | Browser-specific bugs (focus, file picker, drag-drop) may not be caught | F10 covers flows via JSDOM; add Playwright as Phase 2 |
| No load test in CI | Import performance regression may go undetected | Add nightly autocannon + large fixture test |
| FTS5 query escaping edge cases | Specially crafted search terms could break search (not a security issue — FTS5 match syntax errors) | Add fuzz-style search tests with special characters |
| Favicon field always empty | Favicon fetching is marked optional in B5 and never implemented in initial scope | Document as known limitation; no tests cover favicon population |
| No contract test between frontend API client and backend | F2 types could drift from actual B3 response shapes | Use B3's OpenAPI-compatible schemas to generate types (future) |

### Specs Without Gaps

All 20 specs have acceptance criteria and verification commands. No orphan specs.

### Total Test Scenarios Identified

| Layer | Spec-embedded ACs | Integration test cases (B10/F10) | Total |
|---|---|---|---|
| Backend | 55 (B1–B9) | 25+ (B10) | ~80 |
| Frontend | 55 (F1–F9) | 15+ (F10) | ~70 |
| **Total** | **110** | **40+** | **~150** |
