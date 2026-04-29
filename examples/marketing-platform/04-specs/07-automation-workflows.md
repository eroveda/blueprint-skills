# Spec: Automation Workflows

```yaml
title: "Automation Workflows"
purpose: "Implement automation triggers, multi-step workflows, drip sequences, and pause/resume with state preservation"
category: "CONSTRUCTION"

input_context: |
  From "Contact, Campaign, Automation, and Event Data Models":
  - Automation table and model class with fields: id, name, trigger_type (enum: contact_added_to_list, tag_applied, date_field_reached, manual), trigger_config (JSON), status (enum: active, paused, draft), paused_at, agency_id
  - AutomationStep table with fields: id, automation_id, order, action_type (enum: send_email, wait, add_tag, remove_tag, move_to_list), action_config (JSON), delay_duration
  - Contact, List, and Template tables and model classes
  - Multi-tenant agency_id pattern

  From "Authentication and Role-Based Access":
  - Authentication and authorization middleware
  - Marketer role for write operations; Viewer for read-only
  - Agency-scoped query filter
  - Authenticated user context

  From "Contact and List Management":
  - Contact CRUD service for querying and updating contacts
  - List membership service for adding/removing contacts from lists
  - Tag management service for applying and removing tags

instructions:
  - "Implement automation CRUD endpoints: POST /automations (create with status 'draft'), GET /automations (list with pagination, filterable by status and trigger_type), GET /automations/:id (detail including steps), PUT /automations/:id (update — only when status is 'draft' or 'paused'), DELETE /automations/:id (only when status is 'draft')"
  - "Implement automation step management endpoints: POST /automations/:id/steps (add a step with order, action_type, action_config, delay_duration), PUT /automations/:id/steps/:step_id (update a step), DELETE /automations/:id/steps/:step_id (remove a step and reorder remaining), GET /automations/:id/steps (list steps in order)"
  - "Implement automation activation endpoint: POST /automations/:id/activate — validate the automation has at least one step, transition status from draft to active; validate trigger_config is complete for the trigger_type"
  - "Implement automation pause endpoint: POST /automations/:id/pause — transition status to paused, record paused_at timestamp; all in-progress contact enrollments retain their current step position"
  - "Implement automation resume endpoint: POST /automations/:id/resume — transition status back to active, clear paused_at; resume processing for all enrolled contacts from their paused step positions"
  - "Create an automation_enrollments tracking table (migration): fields id, automation_id, contact_id, current_step_id, status (enum: active, completed, paused, cancelled), enrolled_at (UTC), next_action_at (nullable, UTC), completed_at (nullable); with unique constraint on (automation_id, contact_id) for active enrollments"
  - "Implement trigger listener service: monitor for trigger events — when a contact is added to a list, tag is applied, or a date field is reached, check for active automations matching that trigger_type and trigger_config, then enroll the contact into the automation at step 1"
  - "Implement the automation execution engine as a background job: query automation_enrollments where status is 'active' and next_action_at is in the past, execute the current step's action (send_email: render template and send via campaign send interface; add_tag: apply tag to contact; remove_tag: remove tag; move_to_list: update list membership), advance to the next step, calculate next_action_at based on the next step's delay_duration"
  - "Implement the 'wait' step type: when the current step is a wait action, set next_action_at to now plus delay_duration without performing any other action"
  - "Implement enrollment completion: when a contact completes the final step, mark enrollment status as completed and record completed_at"
  - "Implement manual trigger endpoint: POST /automations/:id/trigger with contact_ids array — manually enroll specified contacts into the automation (only for automations with trigger_type 'manual')"
  - "Handle paused automation behavior: when an automation is paused, the execution engine skips all enrollments for that automation; when resumed, enrollments recalculate next_action_at relative to the resume time"
  - "Apply authentication, authorization, and agency-scoped filtering to all endpoints"
  - "Write unit tests for trigger matching logic, step execution for each action_type, enrollment state transitions, and pause/resume timing recalculation"
  - "Write integration tests for automation CRUD, step management, trigger-based enrollment, multi-step execution sequence, and pause/resume without state loss"

constraints:
  - "Automations can be paused and resumed without losing execution state — enrollments must retain their current step position"
  - "A contact can only be enrolled once in a given automation at a time — duplicate enrollment attempts are ignored"
  - "Automations can only be edited when in 'draft' or 'paused' status"
  - "Automations can only be deleted when in 'draft' status"
  - "The execution engine must process steps idempotently — if a step execution is retried, it must not duplicate the action"
  - "All timestamps in UTC"

acceptance_criteria:
  - "POST /automations creates an automation with status 'draft' and returns HTTP 201"
  - "POST /automations/:id/activate with no steps returns HTTP 422"
  - "Adding a contact to a list triggers enrollment in an active automation with trigger_type 'contact_added_to_list' matching that list"
  - "A three-step automation (send_email, wait 1 hour, add_tag) executes steps in order with the correct delay"
  - "POST /automations/:id/pause preserves each enrolled contact's current step position"
  - "POST /automations/:id/resume continues processing from the paused step — no steps are skipped or repeated"
  - "A contact already enrolled in an automation is not enrolled again"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Contact, Campaign, Automation, and Event Data Models"
    - "Authentication and Role-Based Access"
    - "Contact and List Management"
  integrates_with:
    - "Template Engine"
    - "Campaign Builder and Scheduling"
    - "Notification System"

handoff: |
  Exposes for downstream specs:
  - Automation enrollment events for Notification System triggers
  - Automation execution status for analytics dashboards
  - Trigger listener service interface for extending with new trigger types
  - automation_enrollments table for reporting on automation performance

verification: "run project test suite"
```
