# Spec: Template Engine

```yaml
title: "Template Engine"
purpose: "Implement reusable email template management with merge tag substitution, preview, and thumbnail generation"
category: "CONSTRUCTION"

input_context: |
  From "Contact, Campaign, Automation, and Event Data Models":
  - Template table and model class with fields: id, name, html_content (text), merge_tags (array/JSON), thumbnail_url, agency_id, created_at, updated_at
  - Contact model with fields available as merge tag variables: first_name, last_name, email, custom_fields
  - Multi-tenant agency_id pattern

  From "Project Bootstrap and Environment Setup":
  - Project directory structure and module conventions
  - Test runner configuration

instructions:
  - "Implement template CRUD endpoints: POST /templates (create), GET /templates (list with pagination, searchable by name), GET /templates/:id (detail), PUT /templates/:id (update), DELETE /templates/:id (delete — only if not referenced by active campaigns)"
  - "Implement template input validation: name required (max 200 characters), html_content required (non-empty), merge_tags auto-extracted from html_content during save"
  - "Implement merge tag extraction: scan html_content for placeholder patterns (e.g., double-brace syntax like {{first_name}}, {{last_name}}, {{email}}, {{custom.field_name}}), extract all unique tags, and persist them in the merge_tags field"
  - "Implement merge tag substitution service: accept a template and a contact record, replace all merge tag placeholders with the corresponding contact field values, handle missing values with a configurable fallback (empty string or a default like 'Valued Customer')"
  - "Implement template preview endpoint: POST /templates/:id/preview — accept optional sample contact data (or use default sample data), render the template with merge tags substituted, return the rendered HTML"
  - "Implement template duplication endpoint: POST /templates/:id/duplicate — create a copy of the template with a modified name (e.g., 'Original Name (Copy)'), useful for creating variations"
  - "Implement template thumbnail generation: when a template is created or updated, enqueue a background job to generate a thumbnail image of the rendered HTML and store the URL in thumbnail_url"
  - "Implement template versioning: track template content changes by storing a version counter and last_modified_by; when a template is updated, increment the version; provide GET /templates/:id/versions to list version history"
  - "Create a template version table (migration): fields id, template_id (foreign key), version_number, html_content, merge_tags, created_by, created_at"
  - "Validate merge tags against known contact fields: warn (but do not reject) when a merge tag references a field not in the standard Contact schema — it may reference a custom field"
  - "Apply authentication and agency-scoped filtering to all endpoints"
  - "Write unit tests for merge tag extraction (standard tags, custom field tags, no tags, malformed tags)"
  - "Write unit tests for merge tag substitution (all fields present, missing fields with fallback, custom fields)"
  - "Write integration tests for template CRUD, preview, duplication, and version history"

constraints:
  - "Templates cannot be deleted if they are referenced by a campaign with status other than 'draft'"
  - "Merge tag syntax must be consistent and documented — use double-brace notation"
  - "Template html_content has no maximum size but must be valid HTML"
  - "Thumbnail generation is asynchronous and must not block the create/update response"
  - "All timestamps in UTC"

acceptance_criteria:
  - "POST /templates with valid data returns HTTP 201 and auto-populates merge_tags from html_content"
  - "A template with '{{first_name}}' and '{{custom.company}}' in html_content has merge_tags ['first_name', 'custom.company']"
  - "POST /templates/:id/preview with sample data returns HTML with all merge tags replaced"
  - "Merge tag substitution with missing contact field uses the fallback value instead of leaving the placeholder"
  - "DELETE /templates/:id for a template used by a 'scheduled' campaign returns HTTP 422"
  - "POST /templates/:id/duplicate creates a new template with name containing '(Copy)'"
  - "GET /templates/:id/versions returns the version history ordered by version_number"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Contact, Campaign, Automation, and Event Data Models"
  integrates_with:
    - "Campaign Builder and Scheduling"
    - "Automation Workflows"

handoff: |
  Exposes for downstream specs:
  - Template CRUD service/module for campaign and automation template selection
  - Merge tag substitution service for rendering emails with contact data
  - Template preview service for campaign preview functionality
  - Template model with version history for auditing

verification: "run project test suite"
```
