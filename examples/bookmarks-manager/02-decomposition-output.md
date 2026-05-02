# SWEBOK Decomposition — Bookmarks Manager Web App

```yaml
project:
  name: "Bookmarks Manager Web App"
  type: "Multi-layer Web Application (REST API + SPA)"

actors:
  - name: "User"
    operations:
      - "create bookmark"
      - "view/list bookmarks"
      - "search bookmarks"
      - "filter bookmarks by tag"
      - "edit bookmark"
      - "delete bookmark"
      - "manage tags"
      - "import browser bookmarks from file"

entities:
  - name: "Bookmark"
    fields: ["id", "url", "title", "description", "favicon", "createdAt", "updatedAt"]
  - name: "Tag"
    fields: ["id", "name", "count"]
  - name: "BookmarkTag"
    fields: ["bookmarkId", "tagId"]
  - name: "ImportResult"
    fields: ["added", "skipped", "failed", "errors[]"]

# ═══════════════════════════════════════════════
# LAYER 1: Backend API (Node.js + Express + SQLite)
# ═══════════════════════════════════════════════
layer_1_backend:
  nodes:
    - id: "B1"
      title: "Backend Bootstrap and Environment Setup"
      category: "OPERATIONS"
      purpose: "Initialize Node.js/Express project, install dependencies (better-sqlite3, multer, zod, dotenv, pino), configure .env, folder structure (src/, data/, migrations/), and npm scripts (dev, start, migrate)"
      depends_on: []

    - id: "B2"
      title: "Database Schema, Migrations, and FTS5 Setup"
      category: "DESIGN"
      purpose: "Define SQLite schema for bookmarks, tags, and bookmark_tags join table. Create FTS5 virtual table over title+url+description. Write migration runner that applies versioned SQL files on startup. Enable WAL mode."
      depends_on: ["B1"]

    - id: "B3"
      title: "REST API Contract and Shared Types"
      category: "DESIGN"
      purpose: "Define OpenAPI-compatible route contracts: endpoints, request/response shapes, error envelope {error, message, field?}, status codes. Define Zod schemas that serve as the single source of truth for validation and TypeScript types."
      depends_on: ["B2"]

    - id: "B4"
      title: "Input Validation, Sanitization, and File Upload Security"
      category: "SECURITY"
      purpose: "Validate all incoming data with Zod middleware. Sanitize and validate URLs (reject non-http/https schemes). Restrict file uploads to .html/.json, enforce MAX_UPLOAD_SIZE. Configure CORS whitelist from CORS_ORIGIN env var. Parameterized queries enforced via better-sqlite3 prepared statements."
      depends_on: ["B3"]

    - id: "B5"
      title: "Bookmark CRUD Endpoints"
      category: "CONSTRUCTION"
      purpose: "Implement POST /bookmarks (create), GET /bookmarks/:id (fetch one), PUT /bookmarks/:id (update fields + tags), DELETE /bookmarks/:id (hard delete). Manage bookmark↔tag associations on create/update: upsert tags, sync join table, recompute tag counts."
      depends_on: ["B3", "B4"]

    - id: "B6"
      title: "Bookmark Search and Tag-Filter Endpoint"
      category: "CONSTRUCTION"
      purpose: "Implement GET /bookmarks with query params (q, tags[], page, limit, sortBy, order). Use FTS5 for full-text search over title+url+description. Combine with tag filter (AND logic across selected tags). Return paginated response {data, total, page, limit}."
      depends_on: ["B5"]

    - id: "B7"
      title: "Tag Management Endpoint"
      category: "CONSTRUCTION"
      purpose: "Implement GET /tags returning all tags sorted by usage count descending, with per-tag bookmark count. Tags are auto-created/deleted as bookmarks are added/removed — no standalone tag CRUD needed."
      depends_on: ["B5"]

    - id: "B8"
      title: "Browser Bookmark Import Endpoint"
      category: "CONSTRUCTION"
      purpose: "Implement POST /import accepting multipart file upload. Detect format (Netscape HTML vs JSON). Parse HTML via DFS traversal extracting <A> elements; map folder hierarchy to tags. Deduplicate by URL against existing bookmarks. Bulk-insert new bookmarks in a single SQLite transaction. Return ImportResult {added, skipped, failed, errors[]}."
      depends_on: ["B4", "B5"]

    - id: "B9"
      title: "Error Handling, Logging, and Health Check"
      category: "OPERATIONS"
      purpose: "Centralized Express error handler that maps Zod errors → 400, not-found → 404, duplicate URL → 409, bad file → 422, unknown → 500 using structured envelope. Request logging via pino/morgan. GET /health endpoint. Graceful SIGTERM shutdown (close DB connection, drain connections)."
      depends_on: ["B5", "B6", "B7", "B8"]

    - id: "B10"
      title: "Backend API Integration Tests"
      category: "TESTING"
      purpose: "Test all endpoints against a real SQLite in-memory database. Cover: bookmark CRUD lifecycle, FTS5 search, tag-filter combinations, import of Chrome HTML fixture + Firefox HTML fixture + JSON fixture, deduplication, error cases (invalid URL, oversized file, bad format). Use supertest + node:test or jest."
      depends_on: ["B8", "B9"]

  validation:
    total_nodes: 10
    construction_count: 4
    construction_ratio: "4/10 (40%)"
    expected_operations:
      - "User → Bookmark → create"
      - "User → Bookmark → read/list"
      - "User → Bookmark → search + filter"
      - "User → Bookmark → update"
      - "User → Bookmark → delete"
      - "User → Tag → list"
      - "User → ImportFile → import"
    coverage: "7/7 operations covered"

# ═══════════════════════════════════════════════
# LAYER 2: Frontend SPA (React + Vite)
# ═══════════════════════════════════════════════
layer_2_frontend:
  nodes:
    - id: "F1"
      title: "Frontend Bootstrap and Vite Configuration"
      category: "OPERATIONS"
      purpose: "Scaffold React+Vite project, install dependencies (react-query, zod, tailwindcss, react-dropzone, sonner). Configure vite.config.js with /api proxy → backend. Set up VITE_API_BASE_URL env var. Configure Tailwind. Define project folder structure (components/, hooks/, api/, types/)."
      depends_on: []

    - id: "F2"
      title: "API Client and Shared Type Definitions"
      category: "DESIGN"
      purpose: "Define TypeScript types: Bookmark, Tag, PaginatedResponse, ImportResult, ApiError. Implement typed fetch wrapper (api/client.ts) with base URL, JSON serialization, and error normalization. Expose per-resource modules: bookmarks.ts, tags.ts, import.ts matching the backend contract."
      depends_on: ["F1"]

    - id: "F3"
      title: "URL Display Sanitization"
      category: "SECURITY"
      purpose: "Utility function that validates displayed and linked URLs reject javascript:/data:/vbscript: schemes before rendering in <a href>. Applied in BookmarkCard to prevent XSS via stored malicious URLs. Client-side file type check before upload (accept='.html,.json', MIME check)."
      depends_on: ["F2"]

    - id: "F4"
      title: "Bookmark Data Hooks (useBookmarks, useTags)"
      category: "CONSTRUCTION"
      purpose: "useBookmarks(filters): wraps react-query useQuery for GET /bookmarks with search+tag+pagination params; provides refetch, optimistic delete mutation, add/edit mutations with cache invalidation. useTags(): wraps GET /tags, auto-refreshes after bookmark mutations. Debounce search input 300ms before triggering query."
      depends_on: ["F2", "F3"]

    - id: "F5"
      title: "BookmarkList, BookmarkCard, and Layout Components"
      category: "CONSTRUCTION"
      purpose: "BookmarkCard: renders favicon, title (linked, sanitized), URL (truncated), description, tag chips, edit/delete action buttons. BookmarkList: renders grid or list of cards, empty state, loading skeletons. AppLayout: top search bar + left tag sidebar + main content area + header with Add/Import buttons."
      depends_on: ["F4"]

    - id: "F6"
      title: "SearchBar and TagSidebar Filter Components"
      category: "CONSTRUCTION"
      purpose: "SearchBar: controlled input wired to useBookmarks search param with 300ms debounce, clear button. TagSidebar: renders all tags as clickable pills with bookmark counts; supports multi-select (toggle), highlights active filters, 'Clear filters' button. Both components push state up to shared filter context."
      depends_on: ["F4"]

    - id: "F7"
      title: "Add/Edit Bookmark Modal"
      category: "CONSTRUCTION"
      purpose: "Modal form with fields: URL (required, validated), title, description, tags (multi-value text input with chip UI). On URL blur, optionally fetch page title from backend. Zod schema validation with inline field errors. Submit calls create or update mutation, closes modal on success, shows toast. Edit mode pre-fills from existing bookmark."
      depends_on: ["F4", "F5"]

    - id: "F8"
      title: "Import Bookmarks Modal"
      category: "CONSTRUCTION"
      purpose: "Modal with react-dropzone file drop zone (accept .html, .json). File type validated client-side. On submit, POST to /import via FormData. Show progress indicator during upload. On success, render ImportResult summary (added/skipped/failed counts, error list). Trigger tags + bookmarks refetch after success."
      depends_on: ["F4", "F3"]

    - id: "F9"
      title: "Pagination Component and View Toggle"
      category: "CONSTRUCTION"
      purpose: "Pagination: renders prev/next buttons and page number pills based on {total, page, limit}; wired to useBookmarks page param. ViewToggle: button group to switch BookmarkList between grid and list layout, persisted to localStorage."
      depends_on: ["F5"]

    - id: "F10"
      title: "Frontend Component and Hook Tests"
      category: "TESTING"
      purpose: "Unit tests for useBookmarks/useTags hooks with msw mock handlers. Component tests for SearchBar debounce behavior, TagSidebar multi-select, BookmarkCard XSS sanitization, ImportModal upload flow and result rendering. Use Vitest + React Testing Library + msw."
      depends_on: ["F7", "F8", "F9"]

  validation:
    total_nodes: 10
    construction_count: 6
    construction_ratio: "6/10 (60%)"
    expected_operations:
      - "User → BookmarkList → view/paginate"
      - "User → SearchBar → search bookmarks"
      - "User → TagSidebar → filter by tag"
      - "User → AddModal → create bookmark"
      - "User → EditModal → update bookmark"
      - "User → BookmarkCard → delete bookmark"
      - "User → ImportModal → import from file"
    coverage: "7/7 operations covered"

# ═══════════════════════════════════════════════
# CROSS-LAYER DEPENDENCIES
# ═══════════════════════════════════════════════
cross_layer_dependencies:
  - from: "F2"
    to: "B3"
    reason: "Frontend API client types are derived from backend REST API contract"
  - from: "F4"
    to: "B5"
    description: "useBookmarks mutations call backend CRUD endpoints"
  - from: "F6"
    to: "B6"
    description: "SearchBar + TagSidebar drive GET /bookmarks query params"
  - from: "F8"
    to: "B8"
    description: "ImportModal POSTs to /import endpoint"
  - from: "F4"
    to: "B7"
    description: "useTags calls GET /tags"
```

---

## Dependency Graph

```
LAYER 1 — BACKEND
─────────────────
B1 (OPERATIONS: Bootstrap)
 └── B2 (DESIGN: Schema + FTS5)
      └── B3 (DESIGN: API Contract)
           └── B4 (SECURITY: Validation + Upload)
                ├── B5 (CONSTRUCTION: Bookmark CRUD)
                │    ├── B6 (CONSTRUCTION: Search + Filter)
                │    ├── B7 (CONSTRUCTION: Tag Endpoint)
                │    └── B8 (CONSTRUCTION: Import Endpoint) ←── B4
                │         └── B9 (OPERATIONS: Error + Health)
                │              └── B10 (TESTING: API Tests)
                └── B8 (CONSTRUCTION: Import) [see above]

LAYER 2 — FRONTEND
──────────────────
F1 (OPERATIONS: Vite Bootstrap)
 └── F2 (DESIGN: API Client + Types) ←── [B3 contract]
      └── F3 (SECURITY: URL Sanitization)
           └── F4 (CONSTRUCTION: useBookmarks + useTags hooks)
                ├── F5 (CONSTRUCTION: BookmarkList + Card + Layout)
                │    ├── F7 (CONSTRUCTION: Add/Edit Modal)
                │    │    └── F10 (TESTING)
                │    └── F9 (CONSTRUCTION: Pagination + Toggle)
                │         └── F10 (TESTING)
                ├── F6 (CONSTRUCTION: SearchBar + TagSidebar)
                │    └── F10 (TESTING)
                └── F8 (CONSTRUCTION: Import Modal)
                     └── F10 (TESTING)
```

---

## Recommended Build Order

| Order | ID  | Title                                          | Category     | Layer    |
|-------|-----|------------------------------------------------|--------------|----------|
| 1     | B1  | Backend Bootstrap and Environment Setup        | OPERATIONS   | Backend  |
| 2     | B2  | Database Schema, Migrations, and FTS5 Setup    | DESIGN       | Backend  |
| 3     | B3  | REST API Contract and Shared Types             | DESIGN       | Backend  |
| 4     | B4  | Input Validation, Sanitization, File Security  | SECURITY     | Backend  |
| 5     | F1  | Frontend Bootstrap and Vite Configuration      | OPERATIONS   | Frontend |
| 6     | B5  | Bookmark CRUD Endpoints                        | CONSTRUCTION | Backend  |
| 7     | B6  | Bookmark Search and Tag-Filter Endpoint        | CONSTRUCTION | Backend  |
| 8     | B7  | Tag Management Endpoint                        | CONSTRUCTION | Backend  |
| 9     | B8  | Browser Bookmark Import Endpoint               | CONSTRUCTION | Backend  |
| 10    | B9  | Error Handling, Logging, and Health Check      | OPERATIONS   | Backend  |
| 11    | B10 | Backend API Integration Tests                 | TESTING      | Backend  |
| 12    | F2  | API Client and Shared Type Definitions         | DESIGN       | Frontend |
| 13    | F3  | URL Display Sanitization                       | SECURITY     | Frontend |
| 14    | F4  | Bookmark Data Hooks (useBookmarks, useTags)    | CONSTRUCTION | Frontend |
| 15    | F5  | BookmarkList, BookmarkCard, and Layout         | CONSTRUCTION | Frontend |
| 16    | F6  | SearchBar and TagSidebar Filter Components     | CONSTRUCTION | Frontend |
| 17    | F7  | Add/Edit Bookmark Modal                        | CONSTRUCTION | Frontend |
| 18    | F8  | Import Bookmarks Modal                         | CONSTRUCTION | Frontend |
| 19    | F9  | Pagination Component and View Toggle           | CONSTRUCTION | Frontend |
| 20    | F10 | Frontend Component and Hook Tests              | TESTING      | Frontend |

---

## Next Step

Run `/blueprint-skills:swebok-generate-spec` for each node, starting with **B1** (Backend Bootstrap), to generate complete executable specifications in `03-SPECS.yaml`.
