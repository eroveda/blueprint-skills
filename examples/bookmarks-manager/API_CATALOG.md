# API Catalog — Bookmarks Manager Web App

## Base URL

| Environment | URL |
|---|---|
| Local (development) | `http://localhost:3000` |
| Via Vite proxy (frontend dev) | `http://localhost:5173` → proxied to `:3000` |
| Production | Same host as the app (single-port deployment) |

> All API routes are prefixed with `/api`. The health check is at `/health` (no prefix).

## Authentication

This application is single-user and locally hosted. **No authentication is required.** All endpoints are publicly accessible from the configured CORS origin.

CORS is restricted to the origin defined in `CORS_ORIGIN` (default: `http://localhost:5173`). Requests from other origins are rejected by the server.

---

## Common Headers

| Header | Required | Description |
|---|---|---|
| `Content-Type` | Yes (POST/PUT) | Must be `application/json` for JSON bodies |
| `Content-Type` | Yes (POST /import) | Must be `multipart/form-data` for file upload |

---

## Common Errors

All error responses share the same envelope shape:

```json
{
  "error": "ERROR_CODE",
  "message": "Human-readable description",
  "field": "fieldName"
}
```

`field` is only present for validation errors indicating which request field failed.

| Status | Error Code | Meaning |
|---|---|---|
| 400 | `VALIDATION_ERROR` | A required field is missing or has an invalid value |
| 404 | `NOT_FOUND` | The requested resource does not exist |
| 409 | `DUPLICATE_URL` | A bookmark with the given URL already exists |
| 413 | `FILE_TOO_LARGE` | Uploaded file exceeds the `MAX_UPLOAD_SIZE` limit |
| 422 | `UPLOAD_ERROR` | File type not accepted (must be `.html` or `.json`) |
| 422 | `PARSE_ERROR` | File could not be parsed as a recognized bookmark format |
| 500 | `INTERNAL_ERROR` | Unexpected server error |

---

## Endpoints by Resource

---

### Bookmarks — `/api/bookmarks`

#### Bookmark Object Shape

All bookmark endpoints return objects in this shape:

```json
{
  "id": 1,
  "url": "https://example.com/article",
  "title": "Example Article",
  "description": "A helpful article about web development.",
  "favicon": "https://example.com/favicon.ico",
  "tags": ["dev", "reference"],
  "createdAt": "2026-05-02T10:00:00.000Z",
  "updatedAt": "2026-05-02T10:00:00.000Z"
}
```

---

#### Create Bookmark

`POST /api/bookmarks`

Creates a new bookmark. Tags are upserted automatically.

**Request Body**:
```json
{
  "url": "https://example.com/article",
  "title": "Example Article",
  "description": "A helpful article about web development.",
  "tags": ["dev", "reference"]
}
```

**Validation**:
- `url`: required, must be a valid `http://` or `https://` URL
- `title`: optional, string (default: `""`)
- `description`: optional, string (default: `""`)
- `tags`: optional, array of strings (each trimmed and lowercased; default: `[]`)

**Response 201**:
```json
{
  "id": 42,
  "url": "https://example.com/article",
  "title": "Example Article",
  "description": "A helpful article about web development.",
  "favicon": "",
  "tags": ["dev", "reference"],
  "createdAt": "2026-05-02T10:00:00.000Z",
  "updatedAt": "2026-05-02T10:00:00.000Z"
}
```

**Error 400** — Missing or invalid URL:
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Invalid url",
  "field": "url"
}
```

**Error 409** — Duplicate URL:
```json
{
  "error": "DUPLICATE_URL",
  "message": "Bookmark with this URL already exists"
}
```

**cURL example**:
```bash
curl -X POST http://localhost:3000/api/bookmarks \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/article",
    "title": "Example Article",
    "description": "A helpful article about web development.",
    "tags": ["dev", "reference"]
  }'
```

---

#### List / Search Bookmarks

`GET /api/bookmarks`

Returns a paginated list of bookmarks. Supports full-text search, tag filtering, sorting, and pagination.

**Query Parameters**:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `q` | string | — | Full-text search across title, URL, and description |
| `tags[]` | string[] | — | Filter by tags (AND logic — bookmark must have ALL specified tags) |
| `page` | integer ≥ 1 | `1` | Page number |
| `limit` | integer 1–100 | `20` | Items per page |
| `sortBy` | `createdAt` \| `updatedAt` \| `title` | `createdAt` | Sort field |
| `order` | `asc` \| `desc` | `desc` | Sort direction |

**Response 200**:
```json
{
  "data": [
    {
      "id": 42,
      "url": "https://example.com/article",
      "title": "Example Article",
      "description": "A helpful article about web development.",
      "favicon": "",
      "tags": ["dev", "reference"],
      "createdAt": "2026-05-02T10:00:00.000Z",
      "updatedAt": "2026-05-02T10:00:00.000Z"
    }
  ],
  "total": 1,
  "page": 1,
  "limit": 20
}
```

**cURL examples**:

Search all bookmarks:
```bash
curl "http://localhost:3000/api/bookmarks"
```

Full-text search:
```bash
curl "http://localhost:3000/api/bookmarks?q=flexbox"
```

Filter by single tag:
```bash
curl "http://localhost:3000/api/bookmarks?tags[]=css"
```

Filter by multiple tags (AND — both required):
```bash
curl "http://localhost:3000/api/bookmarks?tags[]=css&tags[]=reference"
```

Paginated and sorted:
```bash
curl "http://localhost:3000/api/bookmarks?page=2&limit=10&sortBy=title&order=asc"
```

Search + tag filter combined:
```bash
curl "http://localhost:3000/api/bookmarks?q=grid&tags[]=css"
```

---

#### Get Single Bookmark

`GET /api/bookmarks/:id`

Returns a single bookmark by its numeric ID.

**Path Parameters**:
- `id` — integer, bookmark ID

**Response 200**:
```json
{
  "id": 42,
  "url": "https://example.com/article",
  "title": "Example Article",
  "description": "A helpful article about web development.",
  "favicon": "",
  "tags": ["dev", "reference"],
  "createdAt": "2026-05-02T10:00:00.000Z",
  "updatedAt": "2026-05-02T10:00:00.000Z"
}
```

**Error 404**:
```json
{
  "error": "NOT_FOUND",
  "message": "Bookmark not found"
}
```

**cURL example**:
```bash
curl http://localhost:3000/api/bookmarks/42
```

---

#### Update Bookmark

`PUT /api/bookmarks/:id`

Updates one or more fields of an existing bookmark. All fields are optional — only provided fields are changed. If `tags` is provided, the entire tag set is replaced.

**Path Parameters**:
- `id` — integer, bookmark ID

**Request Body** (all fields optional):
```json
{
  "url": "https://example.com/updated",
  "title": "Updated Title",
  "description": "Now with updated description.",
  "tags": ["dev", "updated"]
}
```

**Validation** (same as create, but all optional):
- `url`: if provided, must be valid `http://` or `https://` URL and not already used by another bookmark
- `tags`: if provided, replaces the entire existing tag set

**Response 200** — Returns the full updated bookmark:
```json
{
  "id": 42,
  "url": "https://example.com/updated",
  "title": "Updated Title",
  "description": "Now with updated description.",
  "favicon": "",
  "tags": ["dev", "updated"],
  "createdAt": "2026-05-02T10:00:00.000Z",
  "updatedAt": "2026-05-02T11:30:00.000Z"
}
```

**Error 404** — Bookmark not found:
```json
{ "error": "NOT_FOUND", "message": "Bookmark not found" }
```

**Error 409** — URL already used by a different bookmark:
```json
{ "error": "DUPLICATE_URL", "message": "Bookmark with this URL already exists" }
```

**cURL example** — update only the title:
```bash
curl -X PUT http://localhost:3000/api/bookmarks/42 \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Title"}'
```

**cURL example** — replace all tags:
```bash
curl -X PUT http://localhost:3000/api/bookmarks/42 \
  -H "Content-Type: application/json" \
  -d '{"tags": ["archive", "read-later"]}'
```

---

#### Delete Bookmark

`DELETE /api/bookmarks/:id`

Permanently deletes a bookmark and all its tag associations.

**Path Parameters**:
- `id` — integer, bookmark ID

**Response 204** — No body returned.

**Error 404**:
```json
{ "error": "NOT_FOUND", "message": "Bookmark not found" }
```

**cURL example**:
```bash
curl -X DELETE http://localhost:3000/api/bookmarks/42
```

---

### Tags — `/api/tags`

#### Tag Object Shape

```json
{
  "id": 7,
  "name": "dev",
  "count": 12
}
```

`count` is the number of bookmarks currently using this tag.

---

#### List All Tags

`GET /api/tags`

Returns all tags sorted by usage count (most used first), then alphabetically for ties. Tags are auto-managed — there is no create/delete endpoint for tags directly.

**Response 200**:
```json
{
  "data": [
    { "id": 3, "name": "reference", "count": 24 },
    { "id": 7, "name": "dev",       "count": 12 },
    { "id": 1, "name": "css",       "count": 8  },
    { "id": 9, "name": "tutorial",  "count": 8  }
  ]
}
```

**cURL example**:
```bash
curl http://localhost:3000/api/tags
```

---

### Import — `/api/import`

#### Import Bookmarks from File

`POST /api/import`

Accepts a browser bookmark export file (`.html` Netscape format or `.json`), parses it, deduplicates against existing bookmarks, and bulk-inserts new entries.

**Request**: `multipart/form-data`

| Field | Type | Required | Description |
|---|---|---|---|
| `file` | File | Yes | Bookmark export file (`.html` or `.json`, max `MAX_UPLOAD_SIZE`) |

**Accepted formats**:
- **Chrome/Edge/Firefox HTML** — Netscape Bookmark Format (`.html`). Folder names become tags.
- **JSON** — flat array `[{url, title, tags}]` or envelope `{bookmarks: [...]}`.

**Response 200**:
```json
{
  "added": 47,
  "skipped": 12,
  "failed": 2,
  "errors": [
    "Skipped 'ftp://old-server/file' — URL must use http or https protocol",
    "Skipped '' — URL must use http or https protocol"
  ]
}
```

| Field | Description |
|---|---|
| `added` | Number of new bookmarks successfully inserted |
| `skipped` | Number of bookmarks whose URL already existed (deduplication) |
| `failed` | Number of bookmarks that could not be saved (e.g., invalid URL) |
| `errors` | Array of human-readable failure messages, one per failed bookmark |

**Error 400** — No file provided:
```json
{ "error": "VALIDATION_ERROR", "message": "No file uploaded" }
```

**Error 413** — File exceeds size limit:
```json
{ "error": "FILE_TOO_LARGE", "message": "File exceeds maximum allowed size" }
```

**Error 422** — Wrong file type:
```json
{ "error": "UPLOAD_ERROR", "message": "Only .html and .json files are accepted" }
```

**Error 422** — Unrecognized format:
```json
{ "error": "PARSE_ERROR", "message": "Could not parse bookmark file — unrecognized format" }
```

**cURL example** — import Chrome HTML export:
```bash
curl -X POST http://localhost:3000/api/import \
  -F "file=@/path/to/bookmarks.html"
```

**cURL example** — import JSON file:
```bash
curl -X POST http://localhost:3000/api/import \
  -F "file=@/path/to/bookmarks.json"
```

---

### Health — `/health`

#### Health Check

`GET /health`

Liveness and readiness probe. Returns the current server status and database connectivity.

**Response 200** — All systems healthy:
```json
{
  "status": "ok",
  "db": "ok",
  "timestamp": "2026-05-02T10:00:00.000Z"
}
```

**Response 200** — Server up but database degraded:
```json
{
  "status": "ok",
  "db": "degraded",
  "timestamp": "2026-05-02T10:00:00.000Z"
}
```

> The HTTP status is always 200 — `db: "degraded"` signals a connectivity issue without triggering a load-balancer restart loop.

**cURL example**:
```bash
curl http://localhost:3000/health
```

---

## Pagination

All list endpoints return a consistent pagination envelope:

```json
{
  "data": [...],
  "total": 143,
  "page": 2,
  "limit": 20
}
```

| Field | Description |
|---|---|
| `total` | Total number of matching items across all pages |
| `page` | Current page number (1-based) |
| `limit` | Items per page |

To compute the total number of pages: `Math.ceil(total / limit)`.

Default values: `page=1`, `limit=20`. Maximum `limit` is `100`.

---

## Data Types Reference

| Field | Format | Example |
|---|---|---|
| `id` | Integer | `42` |
| `url` | String — `http://` or `https://` only | `"https://example.com"` |
| `title` | String | `"My Article"` |
| `description` | String | `"A useful reference."` |
| `favicon` | String (URL or empty) | `"https://example.com/favicon.ico"` |
| `tags` | Array of strings (lowercase) | `["dev", "css"]` |
| `createdAt` | ISO 8601 UTC string | `"2026-05-02T10:00:00.000Z"` |
| `updatedAt` | ISO 8601 UTC string | `"2026-05-02T11:30:00.000Z"` |
| `count` | Integer | `12` |
| `added` | Integer | `47` |
| `skipped` | Integer | `12` |
| `failed` | Integer | `2` |
| `errors` | Array of strings | `["Skipped '...' — reason"]` |

---

## Versioning

The current API does not use URL versioning (`/v1/`). It is a single-version internal API consumed exclusively by the bundled React frontend.

**Current version**: 1.0 (implicit — no version prefix)
**Versioning approach**: If a breaking change is needed in future, introduce `/api/v2/` prefix and maintain `/api/` (v1) in parallel during a deprecation window.
**Deprecation policy**: Communicate breaking changes in release notes with a minimum 2-version notice.

---

## CORS Policy

| Setting | Value |
|---|---|
| Allowed origin | Value of `CORS_ORIGIN` env var (default: `http://localhost:5173`) |
| Allowed methods | `GET`, `POST`, `PUT`, `DELETE` |
| Allowed headers | `Content-Type` |

Requests from other origins will receive a CORS error and no response body.
