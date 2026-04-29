# Spec: Project Bootstrap and Environment Setup

```yaml
title: "Project Bootstrap and Environment Setup"
purpose: "Initialize project structure, dependencies, database connections, background job runner, environment configuration, and CI/CD pipeline"
category: "OPERATIONS"

input_context: |
  Greenfield project — nothing exists yet.
  Requirements defined in the project input document:
  - SaaS web application for marketing automation
  - Three actor roles: Agency Admin, Marketer, Viewer/Client
  - Multi-tenant architecture with agency-level data isolation
  - Async email sending for large recipient lists
  - Near-real-time event ingestion with up to 5-minute delay tolerance

instructions:
  - "Create the top-level project directory with separate folders for source code, tests, configuration, database migrations, and documentation"
  - "Initialize package management and install core dependencies: web framework, ORM/database driver, authentication library, background job processor, and test framework"
  - "Configure the primary relational database connection with connection pooling and environment-based credentials"
  - "Configure a message broker or job queue backend for async task processing (campaign sends, event ingestion)"
  - "Create an environment configuration module that reads from environment variables with sensible defaults for development, and document all required variables"
  - "Set up the database migration tool and verify it can create and rollback an empty migration"
  - "Create a health-check endpoint at GET /health that verifies database connectivity and job queue availability, returning status and version"
  - "Configure structured logging with request correlation IDs and log levels configurable per environment"
  - "Set up the test runner with separate commands for unit tests, integration tests, and full suite execution"
  - "Create a seed script that populates development data: one agency, one admin user, one marketer user, one viewer user"
  - "Configure CI pipeline: lint, type-check (if applicable), run unit tests, run integration tests against a test database"
  - "Create a Dockerfile or equivalent containerization config for consistent local development and deployment"
  - "Document the project structure, setup instructions, and environment variables in ARCHITECTURE.md"

constraints:
  - "All timestamps must be stored and transmitted in UTC"
  - "Environment secrets must never be committed to version control — use environment variables or a secrets manager"
  - "The project must support running the full test suite in under 60 seconds for an empty project"
  - "Database migrations must be idempotent and reversible"
  - "The health-check endpoint must respond within 500ms"

acceptance_criteria:
  - "Project installs dependencies and starts the web server without errors"
  - "GET /health returns HTTP 200 with JSON containing status, database connectivity, and job queue status"
  - "Database migration tool creates and rolls back a test migration successfully"
  - "Background job processor starts and can enqueue and execute a no-op test job"
  - "CI pipeline runs lint, type-check, and test suite to completion"
  - "Seed script creates development data and the admin user can be queried from the database"
  - "All tests pass with project test suite"

dependencies:
  hard: []
  integrates_with: []

handoff: |
  Exposes for all downstream specs:
  - Project directory structure and module organization conventions
  - Database connection and migration tooling
  - Background job queue infrastructure for async task processing
  - Environment configuration module
  - Health-check endpoint pattern for extending with additional checks
  - Test runner configuration and conventions
  - CI pipeline for automated validation
  - Seed data: agency, admin, marketer, and viewer user records

verification: "run project test suite"
```
