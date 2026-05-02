# Architecture — Bookmarks Manager Web App

## Overview

A personal bookmarks manager web application that allows a single user to save, organize, search, and import browser bookmarks. The system consists of a Node.js/Express REST API backed by SQLite, served alongside a React single-page application built with Vite. The primary actor is the **User**, who manages a personal collection of bookmarks organized with free-form tags and searchable via full-text search.

**Scope**: Single-user, locally hosted. No authentication layer (single-user assumption). No cloud sync.

---

## Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Backend Runtime | Node.js | ≥ 18 | Server-side JavaScript runtime |
| Backend Framework | Express | ^4.x | HTTP routing and middleware |
| Database | SQLite (via better-sqlite3) | ^9.x | Embedded relational database |
| Full-Text Search | SQLite FTS5 | built-in | Full-text search over bookmarks |
| File Upload | multer | ^1.x | Multipart form-data handling |
| Input Validation | Zod | ^3.x | Schema-first validation + type inference |
| Request Logging | pino + pino-http | ^8.x | Structured JSON request logging |
| Frontend Framework | React | ^18.x | Component-based SPA |
| Build Tool | Vite | ^5.x | Dev server, HMR, and production bundler |
| Server State | @tanstack/react-query | ^5.x | Async data fetching and caching |
| HTTP Client | axios | ^1.x | Typed HTTP client with interceptors |
| Styling | Tailwind CSS | ^4.x (via @tailwindcss/vite) | Utility-first CSS |
| Notifications | sonner | ^1.x | Toast notification system |
| File Drag-Drop | react-dropzone | ^14.x | Browser file drop zone |
| Frontend Testing | Vitest + @testing-library/react | ^1.x / ^14.x | Component and hook tests |
| API Mocking | msw (Mock Service Worker) | ^2.x | Request interception in tests |
| Backend Testing | Jest + supertest | ^29.x / ^6.x | API integration tests |

---

## Component Architecture

### Module: `src/db` (Backend — Database Layer)

- **Responsibility**: SQLite connection singleton, migration runner, and low-level query helpers. Owns all prepared statements.
- **Public API**:
  - `db` — better-sqlite3 `Database` instance (WAL mode, foreign keys ON)
  - `runMigrations(db)` — applies pending SQL migration files from `migrations/`
  - `insertBookmark(db, fields)`, `getBookmarkById(db, id)`, `updateBookmark(db, id, fields)`, `deleteBookmark(db, id)`
  - `syncTags(db, bookmarkId, tagNames)` — atomic upsert + join-table sync
  - `getTagCounts(db)` — tags sorted by usage frequency
  - `searchBookmarks(db, filters)` — FTS5 + tag filter + pagination query
- **Dependencies**: better-sqlite3, `migrations/*.sql`
- **Database tables owned**: `bookmarks`, `tags`, `bookmark_tags`, `bookmarks_fts` (FTS5 virtual), `schema_migrations`

---

### Module: `src/middleware` (Backend — Cross-Cutting)

- **Responsibility**: Reusable Express middleware for validation, security enforcement, and file upload.
- **Public API**:
  - `validateBody(schema)` — Zod-based body validation middleware factory
  - `validateQuery(schema)` — Zod-based query param validation middleware factory
  - `sanitizeUrl(url)` — pure function rejecting non-http/https URLs (returns normalized URL or throws)
  - `uploadMiddleware` — multer single-file middleware (memory storage, size/type restricted)
  - `errorHandler` — centralized 4-arg Express error middleware
- **Dependencies**: `zod`, `multer`, `src/schemas.js`
- **Database tables owned**: none

---

### Module: `src/routes` (Backend — HTTP Layer)

- **Responsibility**: Express routers binding HTTP routes to business logic. One file per resource.
- **Public API** (routes registered under `/api/`):
  - `bookmarks.js` → `/api/bookmarks` — CRUD + search
  - `tags.js` → `/api/tags` — tag listing
  - `import.js` → `/api/import` — file upload + parsing
  - `health.js` → `/health` — liveness/readiness probe
- **Dependencies**: `src/db/*`, `src/middleware/*`, `src/schemas.js`, `src/parsers/*`
- **Database tables owned**: none (delegates to `src/db`)

---

### Module: `src/parsers` (Backend — Import Parsing)

- **Responsibility**: Parse browser bookmark export files into a normalized `{url, title, tags[]}` format.
- **Public API**:
  - `parseNetscapeHTML(htmlString)` — parses Chrome/Firefox Netscape HTML exports; maps folder hierarchy to tags
  - `parseJSONBookmarks(jsonString)` — parses flat or enveloped JSON bookmark arrays
  - `detectAndParse(buffer, originalname)` — format detection + dispatch
- **Dependencies**: none (pure functions, no DB access)
- **Database tables owned**: none

---

### Module: `src/api` (Frontend — API Client)

- **Responsibility**: Typed HTTP functions wrapping all backend endpoints. Single source of truth for API calls.
- **Public API**:
  - `getBookmarks(filters)`, `getBookmark(id)`, `createBookmark(input)`, `updateBookmark(id, input)`, `deleteBookmark(id)`
  - `getTags()`
  - `importBookmarks(file)`
- **Dependencies**: `axios`, `src/types/index.ts`, `import.meta.env.VITE_API_BASE_URL`
- **Database tables owned**: none

---

### Module: `src/hooks` (Frontend — Server State)

- **Responsibility**: React Query-based hooks encapsulating all server state, mutations, and cache management.
- **Public API**:
  - `useBookmarks(filters)` — paginated bookmark list with debounced search, optimistic delete, create/update mutations
  - `useTags()` — tag list with automatic revalidation after bookmark mutations
  - `useImport()` — file import mutation with result + reset
- **Dependencies**: `@tanstack/react-query`, `src/api/*`, `src/types/*`
- **Database tables owned**: none

---

### Module: `src/components` (Frontend — UI Layer)

- **Responsibility**: React components organized by domain. No direct API access — all data via hooks props.
- **Sub-modules**:
  - `bookmarks/` — `BookmarkCard`, `BookmarkList`, `BookmarkCardSkeleton`, `EmptyState`, `AddEditBookmarkModal`, `TagInput`
  - `search/` — `SearchBar`
  - `tags/` — `TagSidebar`
  - `import/` — `ImportModal`, `ImportResultSummary`
  - `layout/` — `AppLayout`
  - `ui/` — `Modal`, `Pagination`, `ViewToggle`
- **Dependencies**: `src/hooks/*`, `src/types/*`, `src/utils/*`, Tailwind CSS, sonner

---

### Module: `src/utils` (Frontend — Utilities)

- **Responsibility**: Pure utility functions with no React dependencies.
- **Public API**:
  - `sanitizeUrl(url)` — returns safe URL string or `about:blank` for dangerous schemes
  - `isSafeUrl(url)` — boolean predicate
  - `validateBookmarkFile(file)` — checks extension and size; returns `{ valid, error? }`
- **Dependencies**: none

---

## Data Model

### Entities

#### `bookmarks`
| Field | Type | Constraints |
|---|---|---|
| `id` | INTEGER | PRIMARY KEY AUTOINCREMENT |
| `url` | TEXT | NOT NULL, UNIQUE |
| `title` | TEXT | NOT NULL, DEFAULT '' |
| `description` | TEXT | NOT NULL, DEFAULT '' |
| `favicon` | TEXT | NOT NULL, DEFAULT '' |
| `created_at` | TEXT | NOT NULL, ISO 8601 UTC timestamp |
| `updated_at` | TEXT | NOT NULL, ISO 8601 UTC timestamp |

**Relationships**: many-to-many with `tags` via `bookmark_tags`.
**Indexes**: PRIMARY KEY on `id`; UNIQUE on `url`; FTS5 virtual index via `bookmarks_fts`.

---

#### `tags`
| Field | Type | Constraints |
|---|---|---|
| `id` | INTEGER | PRIMARY KEY AUTOINCREMENT |
| `name` | TEXT | NOT NULL, UNIQUE |

**Relationships**: many-to-many with `bookmarks` via `bookmark_tags`.
**Notes**: Tags are auto-created on bookmark create/update; auto-orphaned on delete (no cleanup — count query handles this).

---

#### `bookmark_tags` (join table)
| Field | Type | Constraints |
|---|---|---|
| `bookmark_id` | INTEGER | NOT NULL, FK → bookmarks(id) ON DELETE CASCADE |
| `tag_id` | INTEGER | NOT NULL, FK → tags(id) ON DELETE CASCADE |

**Primary Key**: composite `(bookmark_id, tag_id)`.

---

#### `bookmarks_fts` (FTS5 virtual table)
- Content table: `bookmarks`, content rowid: `id`
- Indexed columns: `title`, `url`, `description`
- Kept in sync via three SQLite triggers: `bookmarks_fts_insert`, `bookmarks_fts_update`, `bookmarks_fts_delete`

---

#### `schema_migrations`
| Field | Type | Constraints |
|---|---|---|
| `version` | TEXT | PRIMARY KEY (migration filename) |
| `applied_at` | TEXT | NOT NULL, ISO 8601 UTC timestamp |

---

### Schema Migrations

| # | File | Description |
|---|---|---|
| 001 | `001_initial_schema.sql` | Creates all tables, FTS5 virtual table, sync triggers |

---

## API Contracts

### Resource: Bookmarks — `/api/bookmarks`

| Method | Path | Request | Response | Status Codes |
|---|---|---|---|---|
| POST | `/api/bookmarks` | `{ url, title?, description?, tags? }` | `Bookmark` | 201, 400, 409 |
| GET | `/api/bookmarks` | Query: `q?, tags[]?, page?, limit?, sortBy?, order?` | `PaginatedResponse<Bookmark>` | 200, 400 |
| GET | `/api/bookmarks/:id` | — | `Bookmark` | 200, 404 |
| PUT | `/api/bookmarks/:id` | Partial `{ url?, title?, description?, tags? }` | `Bookmark` | 200, 400, 404, 409 |
| DELETE | `/api/bookmarks/:id` | — | — (no body) | 204, 404 |

**Bookmark response shape**:
```
{ id, url, title, description, favicon, tags: string[], createdAt, updatedAt }
```

**Search params**:
- `q` — full-text search string (FTS5, escaped)
- `tags[]` — AND filter; bookmark must have ALL specified tags
- `page` — default 1
- `limit` — default 20, max 100
- `sortBy` — `createdAt` | `updatedAt` | `title` (default `createdAt`)
- `order` — `asc` | `desc` (default `desc`)

---

### Resource: Tags — `/api/tags`

| Method | Path | Request | Response | Status Codes |
|---|---|---|---|---|
| GET | `/api/tags` | — | `{ data: Tag[] }` | 200 |

**Tag response shape**: `{ id, name, count }` — sorted by `count DESC, name ASC`.

---

### Resource: Import — `/api/import`

| Method | Path | Request | Response | Status Codes |
|---|---|---|---|---|
| POST | `/api/import` | `multipart/form-data`, field `file` (.html or .json) | `ImportResult` | 200, 400, 413, 422 |

**ImportResult shape**: `{ added, skipped, failed, errors: string[] }`

---

### Resource: Health — `/health`

| Method | Path | Response | Status Codes |
|---|---|---|---|
| GET | `/health` | `{ status: 'ok'|'degraded', db: 'ok'|'degraded', timestamp }` | 200 |

---

### Error Envelope

All error responses use the same shape:
```
{ error: string, message: string, field?: string }
```

| Error Code | HTTP Status | Trigger |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Zod schema failure |
| `DUPLICATE_URL` | 409 | Unique constraint on `url` |
| `NOT_FOUND` | 404 | Resource not found |
| `FILE_TOO_LARGE` | 413 | File exceeds `MAX_UPLOAD_SIZE` |
| `UPLOAD_ERROR` | 422 | Multer error (wrong type, etc.) |
| `PARSE_ERROR` | 422 | Unrecognizable bookmark file format |
| `INTERNAL_ERROR` | 500 | Unhandled server error |

---

## Cross-Cutting Concerns

### Authentication & Authorization

No authentication layer. The application is designed for single-user local deployment. CORS is configured to restrict API access to the configured origin (`CORS_ORIGIN` env var, default `http://localhost:5173`), providing a minimal access boundary in development.

**Future consideration**: Add HTTP Basic Auth or a simple API key header if exposed beyond localhost.

---

### Input Validation & Sanitization

**Backend**:
- All incoming request bodies and query params validated with Zod schemas defined in `src/schemas.js`
- URL scheme validation: only `http:` and `https:` accepted; rejections return 400
- All DB queries use better-sqlite3 prepared statements — SQL injection is structurally prevented
- Imported bookmark files: extension whitelist (`.html`, `.json`), max size from env, memory-only storage

**Frontend**:
- `sanitizeUrl(url)` applied to every `<a href>` rendering a stored URL — maps dangerous schemes to `about:blank`
- Client-side file validation (`validateBookmarkFile`) before upload — blocks invalid files before network request

---

### Logging & Observability

| Concern | Tool | Output |
|---|---|---|
| Request logging | pino-http | Structured JSON: method, path, statusCode, responseTime |
| Application logging | pino | Structured JSON with level, msg, timestamp |
| Error logging | errorHandler → pino | Full stack trace on 500 errors (server-side only) |
| Health check | `GET /health` | DB connectivity probe + timestamp |

Log level controlled via `LOG_LEVEL` env var (default `info`).

---

### Configuration

All configuration is environment-variable driven:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `3000` | Express listen port |
| `DB_PATH` | `./data/bookmarks.db` | SQLite file path (`:memory:` for tests) |
| `CORS_ORIGIN` | `http://localhost:5173` | Allowed CORS origin |
| `MAX_UPLOAD_SIZE` | `10mb` | Maximum import file size |
| `LOG_LEVEL` | `info` | Pino log level |
| `VITE_API_BASE_URL` | `/api` | Frontend API base path (Vite env) |

---

### Dev vs Production Topology

**Development**:
```
Browser → Vite dev server (:5173) → [proxy /api → Express (:3000)]
                                  → [proxy /health → Express (:3000)]
```

**Production**:
```
Browser → Express (:3000) → serves React build from dist/
                          → handles /api/* and /health routes inline
```
Express uses `express.static('frontend/dist')` in production mode, enabling single-port deployment.

---

## Non-Functional Requirements

| NFR | Target | Validation |
|---|---|---|
| API response time (P95) | < 50ms (local SQLite) | Manual load test with autocannon |
| Import throughput | 10,000 bookmarks in < 5s | Integration test with large fixture |
| Frontend initial load | < 2s on localhost | Vite bundle size audit |
| Test coverage — backend | > 80% statements | Jest coverage report |
| Test coverage — frontend | > 80% statements | Vitest coverage (v8 provider) |
| SQLite WAL concurrency | Supports parallel reads | Enabled via `PRAGMA journal_mode=WAL` |

---

## Architectural Decisions (ADRs)

### ADR-001: SQLite as the Database Engine

- **Status**: Accepted
- **Context**: The application is single-user and locally hosted. A server-based database (PostgreSQL, MySQL) would require a running daemon and adds operational overhead.
- **Decision**: Use SQLite via better-sqlite3. Enable WAL mode for concurrent reads. Use FTS5 for full-text search.
- **Consequences**: Zero-configuration database, single file backup, no network latency. Trade-off: not suitable for multi-user or distributed deployment without significant rearchitecture.

---

### ADR-002: FTS5 Triggers for Search Index Sync

- **Status**: Accepted
- **Context**: SQLite FTS5 content tables require explicit sync when the source table changes. Manual sync in application code is error-prone.
- **Decision**: Use three SQLite triggers (`_insert`, `_update`, `_delete`) on the `bookmarks` table to automatically keep `bookmarks_fts` synchronized.
- **Consequences**: FTS index is always consistent with the source table. No application-level sync code required. Triggers add minor write overhead.

---

### ADR-003: CommonJS throughout Backend

- **Status**: Accepted
- **Context**: better-sqlite3 is a native Node.js addon that historically has had friction with ES module interop. The project targets Node.js ≥ 18 but prioritizes simplicity.
- **Decision**: Use CommonJS (`require`/`module.exports`) throughout the backend.
- **Consequences**: Consistent module system, no ESM/CJS interop issues. Trade-off: cannot use top-level `await` without wrapping; frontend uses ESM (Vite handles this separately).

---

### ADR-004: In-Memory multer Storage for Imports

- **Status**: Accepted
- **Context**: Uploaded bookmark files are transient — parsed and discarded. Writing to disk introduces cleanup complexity and potential security surface.
- **Decision**: Use multer `memoryStorage()` — files are held in `req.file.buffer` and never written to disk.
- **Consequences**: Simple, secure, and cleanup-free. Trade-off: large files consume server memory for the duration of the request. Mitigated by `MAX_UPLOAD_SIZE` limit (default 10MB).

---

### ADR-005: Optimistic UI Delete in React Query

- **Status**: Accepted
- **Context**: Delete operations should feel instant to the user. Waiting for the server round-trip before removing the card creates noticeable lag.
- **Decision**: Implement optimistic delete in `useBookmarks`: snapshot cache → remove item → restore on error → invalidate on settle.
- **Consequences**: Perceived performance improvement. Risk: if delete fails, the item briefly disappears then reappears with an error toast. Acceptable UX trade-off for a local app.

---

### ADR-006: Folder-to-Tag Mapping for Netscape HTML Import

- **Status**: Accepted
- **Context**: Browser bookmark exports use hierarchical folders. The bookmarks manager uses flat tags. The mapping strategy must be decided.
- **Decision**: Each ancestor folder name in the bookmark tree becomes a tag on the bookmark. Folder names are lowercased and trimmed.
- **Consequences**: Folder hierarchy is flattened to a tag set. A bookmark in `Dev Tools > Chrome Extensions` gets tags `['dev tools', 'chrome extensions']`. Intuitive for most users; deep hierarchies may produce many tags.

---

## Standards Compliance

- **IEEE SWEBOK v4.0** — Work nodes categorized as DESIGN, CONSTRUCTION, SECURITY, TESTING, OPERATIONS
- **ISO/IEC/IEEE 12207:2017** — Software lifecycle processes (requirements → design → construction → testing → deployment)
- **ISO/IEC/IEEE 15289:2019** — Life-cycle information items (this document)
- **OWASP Top 10** — XSS mitigated via URL sanitization; SQL injection mitigated via prepared statements; file upload security via type/size restrictions

---

## Glossary

| Term | Definition |
|---|---|
| Bookmark | A saved web URL with optional title, description, tags, and favicon |
| Tag | A free-form label associated with one or more bookmarks; auto-created on use |
| FTS5 | SQLite full-text search engine extension used for searching bookmark content |
| Netscape HTML | Browser bookmark export format (`.html`) used by Chrome, Firefox, and Edge |
| Import | The process of parsing a browser bookmark export file and bulk-inserting new bookmarks |
| WAL mode | SQLite Write-Ahead Logging — enables concurrent read access during writes |
| Optimistic UI | Updating the UI before server confirmation, with rollback on failure |
| PaginatedResponse | API response envelope: `{ data, total, page, limit }` |
| ImportResult | API response from `/import`: `{ added, skipped, failed, errors[] }` |
| Deduplication | Skipping bookmarks whose URL already exists in the database during import |
