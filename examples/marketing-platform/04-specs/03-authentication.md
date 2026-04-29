# Spec: Authentication and Role-Based Access

```yaml
title: "Authentication and Role-Based Access"
purpose: "Implement authentication with token rotation, role-based access control, multi-tenant data isolation, and API key management"
category: "SECURITY"

input_context: |
  From "Project Bootstrap and Environment Setup":
  - Project directory structure and module organization conventions
  - Database connection and migration tooling
  - Test runner configuration and conventions
  - Seed data with agency, admin, marketer, and viewer user records

instructions:
  - "Create a migration for the User entity with fields: id, email (unique), password_hash, first_name, last_name, role (enum: admin, marketer, viewer), agency_id (foreign key), is_active (boolean), created_at (UTC), updated_at (UTC)"
  - "Create a migration for the RefreshToken entity with fields: id, user_id (foreign key), token_hash, expires_at (UTC), revoked_at (nullable, UTC), created_at (UTC)"
  - "Create a migration for the ApiKey entity with fields: id, name, key_hash, agency_id (foreign key), created_by (foreign key to User), permissions (JSON array), expires_at (nullable, UTC), last_used_at (nullable, UTC), created_at (UTC)"
  - "Implement user registration endpoint: accept email, password, first_name, last_name; hash password with a secure algorithm (bcrypt/argon2 equivalent); assign to the requesting agency"
  - "Implement login endpoint: validate credentials, generate short-lived access token (15-minute expiry) and long-lived refresh token (7-day expiry), return both tokens"
  - "Implement refresh token endpoint: validate refresh token, revoke the used token, issue new access and refresh token pair (rotation)"
  - "Implement logout endpoint: revoke the current refresh token"
  - "Create authentication middleware that extracts and validates the access token from the Authorization header, attaches the authenticated user and agency context to the request"
  - "Create authorization middleware that checks the user role against the required role for each endpoint; define permission matrix — Admin: full access; Marketer: read/write contacts, campaigns, templates, automations, analytics; Viewer: read-only access to campaigns, contacts, and reports"
  - "Implement agency-level data isolation: add a query scope or filter that automatically restricts all database queries to the authenticated user's agency_id — no endpoint should ever return data from another agency"
  - "Create API key generation endpoint (Admin only): generate a random key, store the hash, return the raw key once"
  - "Create API key authentication middleware: accept API key via X-API-Key header, validate against stored hashes, attach agency context, enforce the permissions array"
  - "Write unit tests for password hashing and verification"
  - "Write integration tests for login, token refresh, token rotation (old refresh token rejected after rotation), and role-based endpoint access"

constraints:
  - "Access tokens must have a maximum lifetime of 15 minutes"
  - "Refresh tokens must be rotated on every use — the old token becomes invalid after a new one is issued"
  - "Passwords must be hashed with a computationally expensive algorithm — never stored in plaintext"
  - "Agency data isolation must be enforced at the query layer, not just the API layer"
  - "API keys must never be stored in plaintext — store only the hash"
  - "Failed login attempts should return a generic error message that does not reveal whether the email exists"

acceptance_criteria:
  - "Login with valid credentials returns access token and refresh token"
  - "Login with invalid credentials returns HTTP 401 with a generic error message"
  - "Accessing a protected endpoint without a token returns HTTP 401"
  - "A Viewer attempting to create a campaign receives HTTP 403"
  - "A Marketer can create contacts but cannot manage users"
  - "Using a refresh token twice (after rotation) returns HTTP 401 on the second attempt"
  - "Requests scoped to agency A never return data belonging to agency B"
  - "API key authentication allows access according to the permissions array"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Project Bootstrap and Environment Setup"
  integrates_with:
    - "Contact, Campaign, Automation, and Event Data Models"

handoff: |
  Exposes for downstream specs:
  - Authentication middleware for protecting endpoints
  - Authorization middleware with role checking (admin, marketer, viewer)
  - Agency-scoped query filter for multi-tenant data isolation
  - Authenticated user context (user_id, agency_id, role) available in every request
  - API key authentication middleware for external integrations
  - User model and registration/login endpoints

verification: "run project test suite"
```
