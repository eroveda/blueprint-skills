# Spec: Authentication and Role-Based Access

```yaml
title: "Authentication and Role-Based Access"
purpose: "Implement authentication, role-based access control for four actor types, service-to-service authentication, and audit logging"
category: "SECURITY"

input_context: |
  From "Project Bootstrap and Environment Setup":
  - Project directory structure and module organization conventions
  - Database connection and migration tooling
  - Test runner configuration and conventions
  - Seed data with admin, data engineer, data scientist, and fraud analyst user records

instructions:
  - "Create a migration for the User entity with fields: id, email (unique), password_hash, first_name, last_name, role (enum: admin, data_engineer, data_scientist, fraud_analyst), is_active (boolean), created_at (UTC), updated_at (UTC)"
  - "Create a migration for the Session entity with fields: id, user_id (foreign key), token_hash, expires_at (UTC), revoked_at (nullable, UTC), created_at (UTC)"
  - "Create a migration for the ServiceCredential entity with fields: id, service_name (unique), credential_hash, permissions (JSON array — e.g. 'read_features', 'write_scores', 'read_models'), is_active (boolean), created_at (UTC)"
  - "Implement user login endpoint: validate credentials, generate session token, return token with expiry"
  - "Implement logout endpoint: revoke the current session token"
  - "Create authentication middleware for the web application that extracts and validates the session token, attaches the authenticated user context (user_id, role) to the request"
  - "Create authorization middleware that checks the user role against endpoint requirements; define the permission matrix — Admin: full access including user management and threshold configuration; Data Engineer: read/write ingestion config, feature definitions, pipeline monitoring; Data Scientist: read/write models, training datasets, A/B tests, feature definitions; Fraud Analyst: read/write alerts, cases, view transaction details and score explanations"
  - "Implement user management endpoints (Admin only): create user, list users, update user role, deactivate user"
  - "Implement service-to-service authentication: services authenticate using pre-shared credentials via a header (e.g. X-Service-Auth); the scoring service authenticates to the feature store, the ingestion service authenticates to the scoring service"
  - "Implement audit log recording: every model promotion, threshold change, user role change, and analyst case decision writes an entry to the AuditLog table with actor_id, action, entity details, and timestamp"
  - "Implement audit log query endpoint (Admin only): list audit logs with filtering by action, entity_type, actor_id, and date range"
  - "Write unit tests for password hashing and verification"
  - "Write integration tests for login, session management, role-based endpoint access (each role accessing permitted and forbidden endpoints), and audit log recording"

constraints:
  - "Session tokens must have a configurable maximum lifetime (default 8 hours for analyst sessions)"
  - "Passwords must be hashed with a computationally expensive algorithm — never stored in plaintext"
  - "Failed login attempts must return a generic error message that does not reveal whether the email exists"
  - "Service-to-service credentials must never be stored in plaintext — store only the hash"
  - "Audit log entries must be immutable — no update or delete operations permitted on the AuditLog table"

acceptance_criteria:
  - "Login with valid credentials returns a session token"
  - "Login with invalid credentials returns HTTP 401 with a generic error message"
  - "Accessing a protected endpoint without a token returns HTTP 401"
  - "A Fraud Analyst attempting to promote a model receives HTTP 403"
  - "A Data Scientist can access model training but cannot manage users"
  - "A Data Engineer can access pipeline configuration but cannot review fraud cases"
  - "Service-to-service authentication allows the scoring service to read from the feature store"
  - "Promoting a model writes an audit log entry with the actor, model ID, and timestamp"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Project Bootstrap and Environment Setup"
  integrates_with:
    - "Transaction, Feature, Model, and Case Data Models"

handoff: |
  Exposes for downstream specs:
  - Authentication middleware for protecting web application endpoints
  - Authorization middleware with role checking (admin, data_engineer, data_scientist, fraud_analyst)
  - Authenticated user context (user_id, role) available in every request
  - Service-to-service authentication middleware for inter-service calls
  - User management endpoints for Admin
  - Audit log recording function for use by any spec that needs audit trail
  - Audit log query endpoint for Admin

verification: "run project test suite"
```
