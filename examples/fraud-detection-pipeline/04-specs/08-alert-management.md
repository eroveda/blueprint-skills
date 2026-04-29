# Spec: Alert and Case Management

```yaml
title: "Alert and Case Management"
purpose: "Implement alert generation, priority-ranked analyst queue, case review workflow, decision recording, and audit trail"
category: "CONSTRUCTION"

input_context: |
  From "Transaction, Feature, Model, and Case Data Models":
  - Transaction table with status state machine (flagged to reviewed, reviewed to resolved)
  - Alert table with transaction_id, fraud_score, priority, status (pending, assigned, reviewed), assigned_to
  - Case table with alert_id, analyst_id, decision (confirmed_fraud, dismissed, escalated), notes, evidence, decided_at
  - AuditLog table for recording analyst actions

  From "Real-time Scoring Service":
  - Flagged transactions with is_flagged = true, status = 'flagged', fraud_score, and feature importance explanation

  From "Authentication and Role-Based Access":
  - Authentication and authorization middleware
  - Fraud Analyst role for case operations
  - Admin role for configuration and reporting
  - Audit log recording function

instructions:
  - "Implement alert generation: when a transaction is scored and flagged (is_flagged = true), automatically create an Alert record with the transaction_id, fraud_score, a priority derived from the score (higher score = higher priority = lower priority number), and status 'pending'"
  - "Implement the alert queue endpoint for Fraud Analysts: GET /alerts — returns alerts sorted by priority (highest fraud score first), filterable by status (pending, assigned, reviewed), with pagination; include a summary of the transaction (amount, merchant, cardholder, timestamp) in each alert"
  - "Implement alert assignment: POST /alerts/{id}/assign — assigns the alert to the requesting analyst (self-assign) or to a specified analyst (Admin only); transitions alert status from 'pending' to 'assigned'; prevent double-assignment"
  - "Implement workload balancing endpoint: GET /alerts/workload — returns the count of assigned (not yet reviewed) alerts per analyst, sorted by count ascending, to help Admins distribute work"
  - "Implement case review view: GET /alerts/{id}/review — returns the full transaction details, the fraud score with feature importance explanation (which features contributed most to the score and by how much), the cardholder's recent transaction history (last 10 transactions), and any prior alerts for the same cardholder"
  - "Implement case decision endpoint: POST /alerts/{id}/decide — accepts decision (confirmed_fraud, dismissed, escalated), notes (required — minimum 10 characters), and optional evidence attachments; creates a Case record, transitions the alert status to 'reviewed', transitions the transaction status to 'reviewed', and writes an audit log entry with the analyst ID, decision, alert ID, and timestamp"
  - "Implement case resolution: when a case decision is recorded, transition the associated transaction status from 'reviewed' to 'resolved' (for confirmed_fraud and dismissed decisions); escalated cases remain in 'reviewed' until a follow-up decision is made"
  - "Implement case history endpoint: GET /cases — returns all cases for the requesting analyst (or all cases for Admin), with filtering by decision type and date range"
  - "Implement analyst dashboard: GET /dashboard/analyst — returns metrics for the requesting analyst: cases reviewed today, average review time, decision distribution (confirmed vs dismissed vs escalated)"
  - "Implement aggregate fraud metrics endpoint (Admin only): GET /dashboard/fraud — returns daily fraud rate, false positive rate (dismissed / total reviewed), analyst throughput (cases per analyst per day), average time to review"
  - "Apply authentication and authorization: Fraud Analysts can view and manage their own alerts and cases; Admins can view all alerts, cases, and dashboards"
  - "Write unit tests for priority ranking logic, alert status transitions, and case decision validation (notes minimum length)"
  - "Write integration tests for the full review workflow: alert created from flagged transaction, analyst assigns and reviews, records decision, transaction status updated, audit log written"

constraints:
  - "Analyst notes are required for every decision and must be at least 10 characters — a decision without justification is rejected"
  - "Alert priority must rank higher fraud scores above lower ones — analysts see the most suspicious transactions first"
  - "An alert can only be assigned to one analyst at a time — concurrent assignment attempts must be handled gracefully (return HTTP 409)"
  - "Every case decision must write an immutable audit log entry — no decision should exist without a corresponding audit trail"
  - "Escalated cases must remain actionable — they return to the queue for a second review by a different analyst or supervisor"
  - "Transactions are held for review, not auto-rejected — the system must never automatically block or reverse a transaction"

acceptance_criteria:
  - "A flagged transaction automatically generates an Alert with the correct priority"
  - "GET /alerts returns alerts sorted by priority with highest fraud scores first"
  - "POST /alerts/{id}/assign assigns the alert and transitions status to 'assigned'"
  - "Attempting to assign an already-assigned alert returns HTTP 409"
  - "GET /alerts/{id}/review returns transaction details, score explanation with feature importance, and cardholder history"
  - "POST /alerts/{id}/decide with notes under 10 characters is rejected with HTTP 422"
  - "POST /alerts/{id}/decide records the case, transitions alert and transaction statuses, and writes an audit log entry"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Transaction, Feature, Model, and Case Data Models"
    - "Authentication and Role-Based Access"
    - "Real-time Scoring Service"
  integrates_with:
    - "Analyst Feedback Loop"

handoff: |
  Exposes for downstream specs:
  - Case records with analyst decisions (confirmed_fraud, dismissed, escalated) for the Feedback Loop
  - Alert queue and case management endpoints for Fraud Analysts
  - Analyst dashboard and aggregate fraud metrics for Admin
  - Audit trail entries for every case decision
  - Transaction status transitions (flagged to reviewed to resolved) triggered by analyst workflow

verification: "run project test suite"
```
