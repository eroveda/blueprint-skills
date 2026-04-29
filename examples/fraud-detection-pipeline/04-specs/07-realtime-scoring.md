# Spec: Real-time Scoring Service

```yaml
title: "Real-time Scoring Service"
purpose: "Implement sub-100ms model inference, explainability, multi-model A/B routing, and score persistence for audit trail"
category: "CONSTRUCTION"

input_context: |
  From "Transaction, Feature, Model, and Case Data Models":
  - Transaction table with status state machine (ingested to scored, scored to flagged)
  - Model table with status (production model is the active one for scoring)
  - ABTest table with champion/challenger model IDs and traffic split

  From "Feature Engineering Service":
  - Feature store with point-in-time feature values
  - Feature retrieval API for assembling feature vectors

  From "Authentication and Role-Based Access":
  - Service-to-service authentication for feature store reads
  - Audit log recording function

  From "Project Bootstrap and Environment Setup":
  - Model registry storage interface for loading model artifacts

instructions:
  - "Implement a scoring service that subscribes to 'transaction.ingested' events (or is called synchronously after ingestion), assembles the feature vector from the feature store, and runs model inference"
  - "Implement model loading: on startup and on model promotion events, load the current production model artifact from the model registry into memory; support hot-swapping models without service restart"
  - "Implement feature vector assembly: given a transaction, query the feature store for all active features for the cardholder and transaction entities; handle missing features gracefully (use default values or skip, log a warning)"
  - "Implement model inference with a strict latency budget: the entire scoring pipeline (feature retrieval + inference + persistence) must complete within 100ms at P99; instrument latency metrics at each stage"
  - "Implement score interpretation: apply the configurable scoring threshold to the model output; if the score exceeds the threshold, mark the transaction as flagged (is_flagged = true, status = 'flagged'); otherwise, transition status to 'scored'"
  - "Implement per-prediction explainability: compute feature importance for each scored transaction — rank the features by their contribution to the fraud score and store the top N (configurable, default 10) contributing features with their names, values, and importance weights"
  - "Implement multi-model routing for A/B testing: when an active ABTest exists, route a fraction of transactions (per the traffic_split configuration) to the challenger model; score using the assigned model and tag the result with the model ID for later analysis"
  - "Persist scoring results: update the transaction record with fraud_score, is_flagged, and the model version used; store the feature importance explanation as a JSON document linked to the transaction for audit trail"
  - "Implement scoring threshold configuration endpoint (Admin only): GET /scoring/config to view current threshold, PUT /scoring/config to update the threshold; record threshold changes in the audit log"
  - "Expose scoring metrics: requests per second, P50/P95/P99 latency, feature retrieval latency, inference latency, flag rate (percentage of transactions flagged)"
  - "Write unit tests for score interpretation at various threshold values, feature importance ranking, and A/B routing logic"
  - "Write integration tests for end-to-end scoring: publish a transaction event, verify the transaction is scored with the correct model, and verify the explanation is persisted"

constraints:
  - "End-to-end scoring latency (feature retrieval + inference + persistence) must be under 100ms at P99"
  - "Transactions scoring above the threshold must be held for analyst review — they must never be auto-rejected or auto-blocked"
  - "All predictions must include explainability data — a score without feature importance is incomplete and must not be persisted"
  - "Model hot-swap must not drop or delay any in-flight scoring requests"
  - "A/B test routing must be deterministic for a given transaction (same transaction always routes to the same model, based on a hash of the transaction ID)"
  - "Scoring threshold must be configurable at runtime without redeployment"

acceptance_criteria:
  - "A transaction event is scored and the result is persisted within 100ms at P99 under load"
  - "A transaction with fraud_score above the threshold is marked is_flagged = true and status = 'flagged'"
  - "A transaction with fraud_score below the threshold is marked is_flagged = false and status = 'scored'"
  - "Every scored transaction has an associated explanation with feature importance rankings"
  - "When an A/B test is active, the configured fraction of transactions are scored by the challenger model"
  - "PUT /scoring/config updates the threshold and creates an audit log entry"
  - "Model hot-swap loads the new model without dropping in-flight requests"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Transaction, Feature, Model, and Case Data Models"
    - "Feature Engineering Service"
  integrates_with:
    - "Transaction Ingestion Pipeline"
    - "Alert and Case Management"
    - "Model Monitoring and Drift Detection"
    - "Authentication and Role-Based Access"

handoff: |
  Exposes for downstream specs:
  - Scored transactions with fraud_score, is_flagged, and status updated
  - Per-prediction explanations with feature importance stored for audit trail
  - Flagged transactions available for Alert and Case Management
  - Scoring metrics (latency, flag rate) for Model Monitoring
  - Threshold configuration endpoint for Admin
  - A/B test scoring results tagged with model ID for Feedback Loop analysis

verification: "run project test suite"
```
