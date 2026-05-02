# Universal Framework Analysis — Bookmarks Manager Web App

This project is a multi-layer web application consisting of a Node.js/Express/SQLite backend REST API and a React/Vite single-page frontend. Each layer is analyzed independently below, then linked via HTTP API contracts.

```yaml
project:
  name: "Bookmarks Manager Web App"
  type: "Multi-layer Web Application"
  layers:
    - name: "Backend API"
      type: "REST API"
      stack: "Node.js + Express + SQLite"
    - name: "Frontend SPA"
      type: "Web App (Single-Page Application)"
      stack: "React + Vite"

# ─────────────────────────────────────────────
# LAYER 1: Backend API (Node.js + Express + SQLite)
# ─────────────────────────────────────────────
layer_1_backend:
  dimensions:
    inputs:
      - category: "HTTP Requests from Frontend"
        items:
          - "POST /bookmarks — create bookmark (url, title, description, tags[])"
          - "GET /bookmarks — list/search bookmarks (query params: q, tags, page, limit)"
          - "GET /bookmarks/:id — fetch single bookmark"
          - "PUT /bookmarks/:id — update bookmark fields"
          - "DELETE /bookmarks/:id — remove bookmark"
          - "GET /tags — list all tags with counts"
          - "POST /import — upload browser bookmark file (HTML/JSON)"
      - category: "Uploaded Browser Bookmark Files"
        items:
          - "Chrome/Edge HTML export (Netscape Bookmark Format)"
          - "Firefox HTML export"
          - "JSON bookmark exports (custom formats)"
          - "Multipart form-data file upload via /import endpoint"
      - category: "Query Parameters"
        items:
          - "Full-text search string (q)"
          - "Tag filter array (tags[])"
          - "Pagination params (page, limit)"
          - "Sort field and order (sortBy, order)"

    processes:
      - category: "Bookmark CRUD"
        items:
          - "Validate and sanitize incoming URL (must be valid URL)"
          - "Auto-fetch page title if title not provided (optional via fetch)"
          - "Insert/update bookmark record in SQLite"
          - "Soft-delete or hard-delete bookmark by ID"
      - category: "Tag Management"
        items:
          - "Parse and normalize tag strings (trim, lowercase)"
          - "Upsert tags into tags table"
          - "Create bookmark↔tag join records"
          - "Recompute tag usage counts on change"
          - "Return tag list sorted by usage frequency"
      - category: "Search"
        items:
          - "Full-text search over title, URL, description using SQLite FTS5"
          - "Filter by one or more tags (AND/OR logic)"
          - "Combine text search + tag filter in single query"
          - "Paginate results with total count metadata"
      - category: "Browser Bookmark Import"
        items:
          - "Detect file type (HTML Netscape format vs JSON)"
          - "Parse HTML bookmark tree (DFS traversal, extract folders as tags)"
          - "Deduplicate URLs against existing bookmarks"
          - "Bulk-insert new bookmarks in a transaction"
          - "Return import summary (added, skipped, failed counts)"
      - category: "Data Persistence"
        items:
          - "SQLite database initialization and schema creation on startup"
          - "Run migrations for schema changes"
          - "Parameterized queries to prevent SQL injection"
          - "Transaction management for bulk imports"

    outputs:
      - category: "JSON API Responses"
        items:
          - "Bookmark object: {id, url, title, description, tags[], createdAt, updatedAt, favicon}"
          - "Paginated list: {data: Bookmark[], total, page, limit}"
          - "Tag list: {data: [{id, name, count}]}"
          - "Import result: {added, skipped, failed, errors[]}"
          - "Structured error responses: {error, message, field?}"
      - category: "HTTP Status Codes"
        items:
          - "200 OK, 201 Created, 204 No Content"
          - "400 Bad Request (validation errors)"
          - "404 Not Found"
          - "409 Conflict (duplicate URL)"
          - "422 Unprocessable Entity (bad file format)"
          - "500 Internal Server Error"

    support:
      - category: "Database Management"
        items:
          - "SQLite file at configurable path (default: ./data/bookmarks.db)"
          - "Schema migrations (better-sqlite3 or custom migration runner)"
          - "FTS5 virtual table for full-text search"
          - "WAL mode enabled for concurrent reads"
      - category: "Middleware & Cross-cutting"
        items:
          - "CORS configuration (allow React dev server origin)"
          - "Request body parsing (JSON + multipart for file uploads)"
          - "Input validation middleware (Zod or express-validator)"
          - "Error handling middleware (centralized, structured responses)"
          - "Request logging (morgan or pino)"
      - category: "Configuration"
        items:
          - "Environment variables: PORT, DB_PATH, CORS_ORIGIN, MAX_UPLOAD_SIZE"
          - ".env file support via dotenv"
      - category: "Deployment"
        items:
          - "npm start / node server.js entrypoint"
          - "Health check endpoint: GET /health"
          - "Graceful shutdown handling (SIGTERM)"

  mapping_to_swebok:
    design_candidates:
      - "Bookmark data model (url, title, description, tags[], timestamps, favicon)"
      - "Tag data model (id, name)"
      - "Bookmark↔Tag join table"
      - "REST API contract (OpenAPI spec)"
      - "Import file format adapters interface"
    construction_candidates:
      - "Express server setup and routing"
      - "SQLite schema + FTS5 setup"
      - "CRUD route handlers"
      - "Tag management logic"
      - "Full-text search query builder"
      - "Browser bookmark HTML parser"
      - "Bulk import transaction handler"
      - "File upload middleware (multer)"
    security_candidates:
      - "URL validation and sanitization"
      - "Parameterized SQL queries"
      - "File upload size and type restrictions"
      - "CORS whitelist"
    operations_candidates:
      - "DB migration runner"
      - "Health check endpoint"
      - "Structured error logging"
      - "Graceful shutdown"

# ─────────────────────────────────────────────
# LAYER 2: Frontend SPA (React + Vite)
# ─────────────────────────────────────────────
layer_2_frontend:
  dimensions:
    inputs:
      - category: "User Interactions"
        items:
          - "Search text input (real-time with debounce)"
          - "Tag selection/deselection (sidebar filter)"
          - "Bookmark form submission (add/edit)"
          - "Delete button click with confirmation"
          - "Browser bookmark file selection (file picker or drag-and-drop)"
          - "Import button trigger"
          - "Pagination navigation (next/prev/page number)"
      - category: "API Responses"
        items:
          - "Bookmark list payload from GET /bookmarks"
          - "Single bookmark from GET /bookmarks/:id"
          - "Tag list from GET /tags"
          - "Import result from POST /import"
          - "Error payloads from failed requests"
      - category: "Browser Bookmark Files"
        items:
          - "HTML file via <input type='file'>"
          - "Drag-and-drop onto import zone"

    processes:
      - category: "State Management"
        items:
          - "Bookmark list state (data, loading, error, pagination)"
          - "Active tag filters state (multi-select)"
          - "Search query state with debounce (300ms)"
          - "Selected/editing bookmark state"
          - "Import modal open/close state and progress"
      - category: "Data Fetching"
        items:
          - "Fetch bookmarks on mount and on filter/search change"
          - "Debounce search input before triggering API call"
          - "Optimistic UI update on delete (remove from list immediately)"
          - "Refetch tags list after add/edit/delete/import"
          - "Handle loading and error states per request"
      - category: "UI Logic"
        items:
          - "Render bookmark cards (favicon, title, URL, tags, actions)"
          - "Render tag sidebar with active-filter highlighting"
          - "Toggle between list and grid view"
          - "Render add/edit bookmark modal with form validation"
          - "Render import modal with file drop zone and result summary"
          - "Show toast notifications for success/error feedback"
          - "Truncate long URLs and descriptions with expand toggle"

    outputs:
      - category: "Rendered UI"
        items:
          - "Bookmark list/grid with cards"
          - "Search bar (top)"
          - "Tag filter sidebar (left or top)"
          - "Add/Edit bookmark modal"
          - "Import bookmarks modal"
          - "Pagination controls"
          - "Empty state (no results)"
          - "Loading skeletons"
          - "Toast notifications"
      - category: "API Requests"
        items:
          - "GET /bookmarks?q=...&tags=...&page=..."
          - "POST /bookmarks with bookmark payload"
          - "PUT /bookmarks/:id"
          - "DELETE /bookmarks/:id"
          - "POST /import with FormData file"
          - "GET /tags"

    support:
      - category: "Build & Dev"
        items:
          - "Vite dev server with HMR"
          - "Vite production build (dist/)"
          - "Environment variable: VITE_API_BASE_URL"
          - "Proxy config in vite.config.js for /api → backend in dev"
      - category: "Dependencies"
        items:
          - "React 18 + React DOM"
          - "react-query or SWR for server state and caching"
          - "Zod or yup for client-side form validation"
          - "Tailwind CSS or CSS Modules for styling"
          - "react-dropzone for file drag-and-drop"
          - "react-hot-toast or sonner for notifications"
      - category: "Routing"
        items:
          - "Single-page (no routing needed) or react-router for /bookmarks, /import views"

  mapping_to_swebok:
    design_candidates:
      - "Bookmark card component interface"
      - "Tag sidebar component interface"
      - "Add/Edit modal form schema"
      - "Import modal state machine"
      - "API client module (typed fetch wrapper)"
    construction_candidates:
      - "BookmarkList + BookmarkCard components"
      - "SearchBar component with debounce"
      - "TagSidebar component"
      - "AddEditBookmarkModal component"
      - "ImportModal component (file drop + progress)"
      - "Pagination component"
      - "API client (bookmarks, tags, import)"
      - "useBookmarks, useTags custom hooks"
    security_candidates:
      - "Sanitize displayed URLs to prevent javascript: scheme XSS"
      - "File type validation on client before upload"
    operations_candidates:
      - "VITE_API_BASE_URL env var"
      - "Vite proxy for local dev"
      - "Build output for static hosting or Express static serving"

# ─────────────────────────────────────────────
# CROSS-LAYER CONTRACTS
# ─────────────────────────────────────────────
contracts:
  - "REST API consumed by frontend over HTTP/JSON"
  - "Backend serves built React app as static files in production (Express static)"
  - "Shared data shape: Bookmark {id, url, title, description, tags[], createdAt, updatedAt, favicon}"
  - "Error envelope: {error: string, message: string, field?: string}"

mapping_to_swebok:
  design_candidates:
    - "Shared Bookmark and Tag type definitions"
    - "OpenAPI spec as the cross-layer contract"
    - "Database schema (bookmarks, tags, bookmark_tags)"
  construction_candidates:
    - "Backend route handlers"
    - "Frontend components and hooks"
    - "Browser bookmark parser utility"
  security_candidates:
    - "URL sanitization (both layers)"
    - "File upload validation (both layers)"
    - "CORS and input validation (backend)"
  operations_candidates:
    - "Production: Express serves React build + API on single port"
    - "Dev: Vite dev server + Vite proxy to Express"
    - "SQLite DB file path management"

next_step: "Pass this analysis to swebok-decompose to generate the work breakdown"
```

## Next Step

Run `/blueprint-skills:swebok-decompose` to break this analysis into IEEE SWEBOK-categorized work nodes, producing `02-SWEBOK-decomposition.md`.
