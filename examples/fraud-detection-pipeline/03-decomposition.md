# Fraud Detection Pipeline — SWEBOK Decomposition

```yaml
project:
  name: "Fraud Detection Pipeline"
  type: "Machine Learning / Real-time System"

actors:
  - name: "Data Engineer"
    operations:
      - "configure and monitor transaction ingestion pipelines"
      - "manage feature store schemas and data quality"
      - "monitor ingestion lag and throughput metrics"
      - "manage dead-letter queues and reprocess failed messages"
  - name: "Data Scientist"
    operations:
      - "define and version feature transformations"
      - "train and evaluate model candidates"
      - "register models in the model registry"
      - "configure and monitor A/B tests"
      - "approve model promotions to production"
      - "analyze model drift and performance reports"
  - name: "Fraud Analyst"
    operations:
      - "review flagged transactions in priority queue"
      - "view score explanations and feature breakdowns"
      - "confirm fraud, dismiss as legitimate, or escalate cases"
      - "add notes and evidence to cases"
      - "view related transactions for a cardholder"
  - name: "Admin"
    operations:
      - "manage user accounts and role assignments"
      - "configure scoring thresholds and alert rules"
      - "view system health and operational dashboards"
      - "export audit trail and compliance reports"

entities:
  - name: "Transaction"
    fields: ["id", "source", "amount", "currency", "merchant_id", "merchant_category", "cardholder_id", "timestamp", "location", "channel", "status", "fraud_score", "is_flagged", "created_at"]
  - name: "Feature"
    fields: ["id", "entity_type", "entity_id", "feature_name", "feature_value", "computed_at", "version"]
  - name: "FeatureDefinition"
    fields: ["id", "name", "description", "computation_type", "parameters", "version", "created_by", "created_at"]
  - name: "Model"
    fields: ["id", "name", "version", "status", "training_dataset_id", "metrics", "feature_set", "created_by", "trained_at", "promoted_at", "retired_at"]
  - name: "TrainingDataset"
    fields: ["id", "snapshot_date", "record_count", "label_distribution", "feature_version", "created_at"]
  - name: "ABTest"
    fields: ["id", "champion_model_id", "challenger_model_id", "traffic_split", "start_date", "min_duration_days", "status", "results", "created_by"]
  - name: "Alert"
    fields: ["id", "transaction_id", "fraud_score", "priority", "status", "assigned_to", "created_at"]
  - name: "Case"
    fields: ["id", "alert_id", "analyst_id", "decision", "notes", "evidence", "decided_at", "created_at"]
  - name: "AuditLog"
    fields: ["id", "actor_id", "action", "entity_type", "entity_id", "details", "timestamp"]
  - name: "MonitoringMetric"
    fields: ["id", "model_id", "metric_name", "metric_value", "window_start", "window_end", "created_at"]

nodes:
  - id: "1"
    title: "Project Bootstrap and Environment Setup"
    category: "OPERATIONS"
    purpose: "Initialize project structure with separate modules for ingestion, feature engineering, training, scoring, and web application; configure database connections, message broker, feature store, model registry, and CI/CD pipeline; set up structured logging with correlation IDs and health-check endpoints for each service"
    depends_on: []

  - id: "2"
    title: "Transaction, Feature, Model, and Case Data Models"
    category: "DESIGN"
    purpose: "Define all entity schemas (Transaction, Feature, FeatureDefinition, Model, TrainingDataset, ABTest, Alert, Case, AuditLog, MonitoringMetric), their relationships, database indexes optimized for high-volume writes and time-range queries, the transaction state machine (ingested, scored, flagged, reviewed, resolved), and the model lifecycle state machine (training, evaluating, staging, production, retired)"
    depends_on: ["1"]

  - id: "3"
    title: "Authentication and Role-Based Access"
    category: "SECURITY"
    purpose: "Implement authentication for the analyst web application with session management, enforce role-based access control for Data Engineer, Data Scientist, Fraud Analyst, and Admin roles, configure service-to-service authentication between ingestion, scoring, and feature store services, and implement audit logging for all model promotions, threshold changes, and analyst decisions"
    depends_on: ["1"]

  - id: "4"
    title: "Transaction Ingestion Pipeline"
    category: "CONSTRUCTION"
    purpose: "Implement streaming consumer that reads transaction events from payment processor topics, validates and normalizes transaction schema (currency, timestamps, merchant codes), deduplicates transactions across multiple source feeds, routes malformed messages to dead-letter queue with error metadata, handles backpressure during burst traffic exceeding 10,000 TPS, and persists validated transactions with status 'ingested'"
    depends_on: ["2", "3"]

  - id: "5"
    title: "Feature Engineering Service"
    category: "CONSTRUCTION"
    purpose: "Implement real-time feature computation triggered on transaction ingestion (velocity counts in sliding windows, amount deviation from cardholder mean, time and distance since last transaction), implement batch feature computation as a scheduled job (historical spending patterns, day-of-week profiles, merchant risk scores), store features in the feature store with point-in-time correctness to prevent data leakage, support feature versioning and backward-compatible schema evolution, and monitor feature freshness with staleness alerts"
    depends_on: ["2", "4"]

  - id: "6"
    title: "Model Training Pipeline"
    category: "CONSTRUCTION"
    purpose: "Assemble training datasets from the feature store using point-in-time joins against labeled transaction data, split data with temporal awareness (train on older periods, validate on newer), execute model training with configurable hyperparameters, evaluate models against precision, recall, F1, AUC-ROC, and false positive rate, register trained models in the model registry with version metadata and feature set references, and compare candidate models against the current production model"
    depends_on: ["2", "5"]

  - id: "7"
    title: "Real-time Scoring Service"
    category: "CONSTRUCTION"
    purpose: "Assemble feature vectors from the feature store for incoming transactions, execute model inference with sub-100ms P99 latency, compute feature importance per prediction for explainability, apply configurable scoring threshold to determine whether a transaction is flagged for review, support multi-model routing for A/B testing (split traffic between champion and challenger models), persist scores and explanations for audit trail, and transition transaction status from 'ingested' to 'scored' or 'flagged'"
    depends_on: ["2", "5"]

  - id: "8"
    title: "Alert and Case Management"
    category: "CONSTRUCTION"
    purpose: "Generate alerts for transactions scoring above the configurable threshold, manage a priority-ranked alert queue (higher fraud scores first), support analyst assignment and workload balancing, implement case review workflow (view transaction details, score explanation with feature importance, related transactions for the same cardholder), record analyst decisions (confirm fraud, dismiss as legitimate, escalate) with mandatory notes, enforce audit trail for every analyst action with timestamp and justification, and transition transaction status through the review lifecycle (flagged, reviewed, resolved)"
    depends_on: ["2", "3", "7"]

  - id: "9"
    title: "Model Monitoring and Drift Detection"
    category: "CONSTRUCTION"
    purpose: "Monitor prediction score distributions over time and detect drift (population stability index, histogram comparison), track feature distribution changes across the production feature set, compute ongoing model performance metrics against labeled outcomes (precision and recall over sliding windows), alert on sudden changes in flagged transaction volume (spike or drop), detect model staleness based on training data age, and generate monitoring dashboards and reports for Data Scientists"
    depends_on: ["2", "7"]

  - id: "10"
    title: "Analyst Feedback Loop"
    category: "CONSTRUCTION"
    purpose: "Map analyst decisions (confirm fraud, dismiss) back to training labels on the corresponding transactions, propagate updated labels to the feature store for inclusion in the next training cycle, trigger daily model retraining using the previous day's labeled data, enforce the model approval workflow (Data Scientist must approve before promotion to production), orchestrate A/B testing between the newly trained model and the current production model with a minimum 7-day evaluation period, and generate comparison reports to support promotion decisions"
    depends_on: ["2", "6", "8"]

validation:
  total_nodes: 10
  construction_count: 7
  construction_ratio: "7/10 (70%)"
  expected_operations:
    - "Data Engineer → Ingestion Pipelines → configure/monitor (covered by Node 4: transaction ingestion pipeline)"
    - "Data Engineer → Feature Store → manage schemas/quality (covered by Node 5: feature engineering service)"
    - "Data Engineer → Dead-Letter Queues → manage/reprocess (covered by Node 4: transaction ingestion pipeline)"
    - "Data Scientist → Features → define/version (covered by Node 5: feature engineering service)"
    - "Data Scientist → Models → train/evaluate (covered by Node 6: model training pipeline)"
    - "Data Scientist → Model Registry → register/compare (covered by Node 6: model training pipeline)"
    - "Data Scientist → A/B Tests → configure/monitor (covered by Node 10: analyst feedback loop)"
    - "Data Scientist → Model Promotions → approve (covered by Node 10: analyst feedback loop)"
    - "Data Scientist → Drift Reports → analyze (covered by Node 9: model monitoring and drift detection)"
    - "Fraud Analyst → Flagged Transactions → review (covered by Node 8: alert and case management)"
    - "Fraud Analyst → Score Explanations → view (covered by Node 8: alert and case management)"
    - "Fraud Analyst → Cases → decide/annotate (covered by Node 8: alert and case management)"
    - "Admin → Users → manage (covered by Node 3: authentication and role-based access)"
    - "Admin → Thresholds → configure (covered by Node 7: real-time scoring service)"
    - "Admin → System Health → view (covered by Node 1: bootstrap and environment setup)"
    - "Admin → Audit Trail → export (covered by Node 3: authentication and role-based access)"
  coverage: "16/16 operations covered"
  business_rules_coverage:
    - "Transactions above threshold held for review, not auto-rejected → Node 7 flags for review, Node 8 manages analyst workflow"
    - "Analyst decisions feed back into training data → Node 10 maps decisions to labels and triggers retraining"
    - "Model deployment requires Data Scientist approval → Node 10 enforces approval workflow"
    - "A/B testing runs minimum 7 days before promotion → Node 10 enforces minimum duration"
    - "Feature store maintains point-in-time correctness → Node 5 prevents data leakage"
    - "All predictions must be explainable → Node 7 computes feature importance per prediction"
    - "Audit trail for every analyst decision → Node 8 records timestamped justifications"
  constraints_coverage:
    - "Scoring latency < 100ms at P99 → Node 7 enforces sub-100ms inference"
    - "Transaction volume 10,000+ TPS at peak → Node 4 handles backpressure at 10k+ TPS"
    - "Daily retraining on previous day's labels → Node 10 triggers daily batch retraining"
  orphan_check: "No orphan dependencies — all depends_on references point to valid node IDs"
```
