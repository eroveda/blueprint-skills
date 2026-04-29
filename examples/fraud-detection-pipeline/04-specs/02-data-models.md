# Spec: Transaction, Feature, Model, and Case Data Models

```yaml
title: "Transaction, Feature, Model, and Case Data Models"
purpose: "Define all entity schemas, relationships, indexes, and state machines for transactions and models"
category: "DESIGN"

input_context: |
  From "Project Bootstrap and Environment Setup":
  - Project directory structure and module organization conventions
  - Database connection and migration tooling
  - Feature store connection and read/write interface
  - Model registry storage interface
  - Test runner configuration and conventions

instructions:
  - "Create a migration for the Transaction entity with fields: id (primary key), source (enum: payment_processor, bank_feed, manual), external_id (string, for deduplication), amount (decimal), currency (string, ISO 4217), merchant_id (string), merchant_category (string, MCC code), cardholder_id (string), timestamp (UTC), location (JSON — lat/lon or country/city), channel (enum: card_present, card_not_present, ach, wire), status (enum: ingested, scored, flagged, reviewed, resolved), fraud_score (nullable decimal, 0.0 to 1.0), is_flagged (boolean, default false), created_at (UTC)"
  - "Create a migration for the Feature entity in the feature store schema with fields: id, entity_type (enum: cardholder, merchant, transaction), entity_id (string), feature_name (string), feature_value (numeric or JSON), computed_at (UTC timestamp for point-in-time correctness), version (integer); with a composite index on (entity_type, entity_id, feature_name, computed_at) for point-in-time lookups"
  - "Create a migration for the FeatureDefinition entity with fields: id, name (unique), description, computation_type (enum: realtime, batch), parameters (JSON — window sizes, aggregation types, thresholds), version (integer), created_by (foreign key to User), is_active (boolean), created_at (UTC)"
  - "Create a migration for the Model entity with fields: id, name (string), version (integer, auto-incremented per name), status (enum: training, evaluating, staging, production, retired), training_dataset_id (foreign key), metrics (JSON — precision, recall, f1, auc_roc, false_positive_rate), feature_set (JSON — list of feature names and versions used), artifact_path (string — reference to model registry), created_by (foreign key to User), trained_at (UTC), promoted_at (nullable, UTC), retired_at (nullable, UTC)"
  - "Create a migration for the TrainingDataset entity with fields: id, snapshot_date (date), date_range_start (date), date_range_end (date), record_count (integer), label_distribution (JSON — counts of fraud vs legitimate), feature_version (integer), created_at (UTC)"
  - "Create a migration for the ABTest entity with fields: id, champion_model_id (foreign key to Model), challenger_model_id (foreign key to Model), traffic_split (decimal — fraction routed to challenger, e.g. 0.1 for 10%), start_date (date), min_duration_days (integer, default 7), end_date (nullable date), status (enum: active, completed, cancelled), results (nullable JSON — comparative metrics), created_by (foreign key to User), created_at (UTC)"
  - "Create a migration for the Alert entity with fields: id, transaction_id (foreign key, unique), fraud_score (decimal), priority (integer — derived from score), status (enum: pending, assigned, reviewed), assigned_to (nullable foreign key to User), created_at (UTC)"
  - "Create a migration for the Case entity with fields: id, alert_id (foreign key, unique), analyst_id (foreign key to User), decision (enum: confirmed_fraud, dismissed, escalated), notes (text), evidence (JSON — list of attachment references), decided_at (UTC), created_at (UTC)"
  - "Create a migration for the AuditLog entity with fields: id, actor_id (foreign key to User), action (string — e.g. 'model_promoted', 'case_decided', 'threshold_changed'), entity_type (string), entity_id (string), details (JSON — before/after values or context), timestamp (UTC); partitioned or indexed by timestamp"
  - "Create a migration for the MonitoringMetric entity with fields: id, model_id (foreign key to Model), metric_name (string — e.g. 'score_mean', 'feature_drift_psi', 'precision_7d'), metric_value (decimal), window_start (UTC), window_end (UTC), created_at (UTC)"
  - "Add database indexes: Transaction.external_id + source (unique, for deduplication), Transaction.cardholder_id + timestamp, Transaction.status, Transaction.is_flagged + status, Alert.status + priority, AuditLog.timestamp, AuditLog.entity_type + entity_id, MonitoringMetric.model_id + metric_name + window_end"
  - "Implement the transaction status state machine as a validation module: define allowed transitions (ingested to scored, scored to flagged, flagged to reviewed, reviewed to resolved, scored to resolved for non-flagged transactions), reject any transition not in the allowed map"
  - "Implement the model lifecycle state machine: define allowed transitions (training to evaluating, evaluating to staging, staging to production, production to retired, evaluating to retired for rejected models), reject invalid transitions"
  - "Create model/entity classes for each table with basic validation rules (amount > 0, fraud_score between 0.0 and 1.0, currency is valid ISO 4217)"
  - "Write unit tests for both state machines covering all valid transitions and all invalid transition attempts"

constraints:
  - "All timestamps stored in UTC with timezone-aware column types"
  - "Transaction status transitions must follow the defined state machine — no skipping allowed"
  - "Model status transitions must follow the defined lifecycle — evaluating models cannot jump directly to production"
  - "Feature entity must support point-in-time queries: given an entity_id and a timestamp, return the feature value that was current at that time"
  - "Transaction table must be optimized for high-volume inserts (10,000+ TPS) and time-range queries"
  - "AuditLog table must be append-only — no updates or deletes allowed"

acceptance_criteria:
  - "All migrations run successfully and create the expected tables with correct column types"
  - "All migrations can be rolled back without errors"
  - "Transaction state machine allows ingested to scored transition"
  - "Transaction state machine rejects ingested to flagged transition (must go through scored)"
  - "Transaction state machine allows scored to resolved for non-flagged transactions"
  - "Model state machine allows evaluating to staging transition"
  - "Model state machine rejects evaluating to production transition (must go through staging)"
  - "Feature table supports point-in-time queries returning the correct value for a given timestamp"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Project Bootstrap and Environment Setup"
  integrates_with: []

handoff: |
  Exposes for downstream specs:
  - Transaction table with status state machine module
  - Feature and FeatureDefinition tables with point-in-time query support
  - Model table with lifecycle state machine module
  - TrainingDataset table for training pipeline
  - ABTest table for A/B test orchestration
  - Alert and Case tables for case management
  - AuditLog table for audit trail recording
  - MonitoringMetric table for drift detection storage
  - All entity validation rules and enum definitions

verification: "run project test suite"
```
