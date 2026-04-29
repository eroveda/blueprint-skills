# Spec: Analyst Feedback Loop

```yaml
title: "Analyst Feedback Loop"
purpose: "Implement label propagation from analyst decisions, automated retraining triggers, model approval workflow, and A/B test orchestration"
category: "CONSTRUCTION"

input_context: |
  From "Transaction, Feature, Model, and Case Data Models":
  - Transaction table with fraud_score and status
  - Model table with lifecycle state machine (evaluating to staging to production to retired)
  - ABTest table with champion/challenger model IDs, traffic_split, min_duration_days, status
  - AuditLog table for recording promotions

  From "Alert and Case Management":
  - Case records with analyst decisions (confirmed_fraud, dismissed, escalated)
  - Transaction status transitions (flagged to reviewed to resolved)

  From "Model Training Pipeline":
  - Training dataset assembly from feature store with point-in-time joins
  - Model training trigger endpoint (POST /models/train)
  - Model comparison endpoint

  From "Authentication and Role-Based Access":
  - Data Scientist role for model approval and A/B test management
  - Audit log recording function

instructions:
  - "Implement label propagation: when an analyst records a case decision (confirmed_fraud or dismissed), update the transaction record with a training label (fraud or legitimate); escalated cases do not receive a label until a final decision is made"
  - "Implement label statistics endpoint: GET /labels/stats — returns the count of newly labeled transactions since the last training run, broken down by label (fraud vs legitimate), and by date"
  - "Implement automated retraining trigger: a daily scheduled job checks for newly labeled transactions since the last training run; if the count exceeds a configurable minimum (default 50), automatically trigger model training via the training pipeline endpoint using the updated labeled data"
  - "Implement model approval workflow: when a newly trained model completes evaluation, it enters 'evaluating' status; a Data Scientist reviews the evaluation metrics and either promotes it to 'staging' (POST /models/{id}/promote-to-staging) or rejects it (POST /models/{id}/reject, which transitions to 'retired'); promotion to staging requires explicit Data Scientist approval"
  - "Implement A/B test initiation: when a model is promoted to 'staging', automatically create an ABTest record with the current production model as champion and the staged model as challenger; configure the traffic split (default 10% to challenger) and set the minimum duration to 7 days"
  - "Implement A/B test monitoring endpoint: GET /ab-tests/{id} — returns the current test status, days elapsed, and comparative metrics (precision, recall, false positive rate, alert rate) for champion vs challenger computed from scored transactions tagged with each model ID"
  - "Implement A/B test completion: after the minimum 7-day duration has elapsed, compute final comparative metrics; a Data Scientist reviews the results and either promotes the challenger to production (POST /ab-tests/{id}/promote) or cancels the test (POST /ab-tests/{id}/cancel)"
  - "Implement model promotion to production: when the challenger is promoted, transition its status from 'staging' to 'production', transition the old champion from 'production' to 'retired', update the scoring service to use the new model, complete the ABTest record with results, and write an audit log entry"
  - "Implement A/B test listing for Data Scientists: GET /ab-tests — returns all A/B tests with filtering by status (active, completed, cancelled)"
  - "Enforce the 7-day minimum: attempting to promote a challenger before 7 days have elapsed returns HTTP 422 with a message indicating the remaining days"
  - "Write unit tests for label propagation logic, retraining trigger threshold, and 7-day minimum enforcement"
  - "Write integration tests for the full feedback cycle: analyst decision creates label, daily job triggers retraining, model is evaluated, Data Scientist promotes to staging, A/B test runs, and challenger is promoted to production"

constraints:
  - "Model deployment to production requires explicit approval from at least one Data Scientist — automated promotion is not allowed"
  - "A/B testing must run for a minimum of 7 days before a model can be promoted — no early promotion is permitted"
  - "Label propagation must only set labels for confirmed_fraud and dismissed decisions — escalated cases must not receive training labels"
  - "Daily retraining uses the previous day's labeled data combined with all historical labeled data — it does not discard older labels"
  - "Only one A/B test can be active at a time — attempting to create a second concurrent test returns HTTP 409"
  - "Every model promotion must be recorded in the audit log with the approving Data Scientist, the model IDs, and the A/B test results"

acceptance_criteria:
  - "An analyst confirming fraud on a case sets the transaction's training label to 'fraud'"
  - "An analyst dismissing a case sets the transaction's training label to 'legitimate'"
  - "An escalated case does not receive a training label"
  - "The daily retraining job triggers model training when newly labeled transactions exceed the minimum threshold"
  - "POST /models/{id}/promote-to-staging transitions the model and creates an A/B test"
  - "GET /ab-tests/{id} returns comparative metrics for champion vs challenger"
  - "Attempting to promote a challenger before 7 days returns HTTP 422"
  - "POST /ab-tests/{id}/promote transitions the challenger to production, retires the champion, and writes an audit log"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Transaction, Feature, Model, and Case Data Models"
    - "Model Training Pipeline"
    - "Alert and Case Management"
  integrates_with:
    - "Real-time Scoring Service"
    - "Model Monitoring and Drift Detection"
    - "Authentication and Role-Based Access"

handoff: |
  Exposes for downstream specs:
  - Training labels propagated from analyst decisions for continuous model improvement
  - Automated daily retraining pipeline
  - Model approval workflow with Data Scientist gate
  - A/B test orchestration with 7-day minimum duration
  - Model promotion pipeline updating the scoring service
  - Complete audit trail for all model lifecycle events

verification: "run project test suite"
```
