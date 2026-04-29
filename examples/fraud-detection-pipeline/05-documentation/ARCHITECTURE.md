# Architecture — Fraud Detection Pipeline

## Overview

The Fraud Detection Pipeline is a multi-service system that ingests financial transactions in real time, computes fraud risk features, scores transactions using machine learning models, and routes suspicious activity to human analysts for review. The system serves four actor types: Data Engineers (pipeline operations), Data Scientists (model lifecycle), Fraud Analysts (case review), and Admins (configuration and oversight).

The platform handles streaming ingestion at 10,000+ transactions per second, real-time model inference with sub-100ms P99 latency, and a continuous feedback loop where analyst decisions improve future models.

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Runtime | Chosen runtime | Service execution for all microservices |
| Web framework | Chosen web framework | HTTP routing for analyst application and management APIs |
| Database (primary) | Chosen relational database | Case management, user accounts, audit logs, model metadata |
| Message broker | Chosen streaming platform | Transaction ingestion, inter-service event communication |
| Feature store | Chosen low-latency store | Point-in-time feature storage and retrieval for scoring and training |
| Model registry | Chosen artifact store | Versioned model artifact storage with metadata |
| Model serving | Chosen serving runtime | Model inference execution within the scoring service |
| Background jobs | Chosen job runner | Batch feature computation, model training, monitoring |
| Object storage | Chosen object store | Training datasets, model artifacts, evidence attachments |

## Component Architecture

### Service: Transaction Ingestion

- **Responsibility**: Consume streaming transaction events from payment processor topics, validate schemas, normalize data (currency, timestamps, merchant codes), deduplicate across sources, route malformed messages to dead-letter queue, publish validated transactions for downstream processing
- **Public API**: Consumer group subscription, `GET /ingestion/health`, `POST /ingestion/dead-letter/{id}/retry`
- **Dependencies**: Message broker, primary database
- **Data stores**: Writes to `transactions` table, publishes to `transaction.ingested` topic

### Service: Feature Engineering

- **Responsibility**: Compute real-time features on transaction events (velocity, amount deviation, time gap, geographic distance) and batch features on schedule (spending patterns, merchant risk, account aggregates); store in feature store with point-in-time correctness; manage feature definitions and versioning
- **Public API**: Feature retrieval API (internal), `FeatureDefinitionController`
- **Dependencies**: Transaction Ingestion (events), primary database, feature store
- **Data stores**: Writes to feature store, reads from `transactions` table

### Service: Model Training

- **Responsibility**: Assemble training datasets from feature store with point-in-time joins, train models with configurable hyperparameters, evaluate against five metrics (precision, recall, F1, AUC-ROC, FPR), register artifacts in model registry, compare candidates for promotion decisions
- **Public API**: `POST /models/train`, `GET /models`, `GET /models/{id}`, `GET /models/compare`
- **Dependencies**: Feature store, model registry, primary database
- **Data stores**: Reads from feature store, writes to `models` and `training_datasets` tables, stores artifacts in model registry

### Service: Real-time Scoring

- **Responsibility**: Assemble feature vectors, execute model inference within 100ms P99, compute per-prediction feature importance, apply scoring threshold, route flagged transactions to alerts, support multi-model A/B testing
- **Public API**: Scoring pipeline (event-driven), `GET /scoring/config`, `PUT /scoring/config`
- **Dependencies**: Feature store, model registry, primary database
- **Data stores**: Updates `transactions` table with scores, writes explanations

### Service: Analyst Web Application

- **Responsibility**: Alert queue management, case review workflow, analyst dashboards, user management, audit trail, model approval and A/B test management
- **Public API**: REST endpoints for alerts, cases, dashboards, users, models, A/B tests, monitoring
- **Dependencies**: Primary database, scoring service (reads scored transactions)
- **Data stores**: Reads/writes `alerts`, `cases`, `audit_logs`, `users`, `ab_tests`

### Shared: Authentication & Access Control

- **Responsibility**: User authentication with session management, role-based access control (Admin, Data Engineer, Data Scientist, Fraud Analyst), service-to-service authentication between internal services, audit log recording
- **Public API**: `AuthController`, `RoleGuard` middleware, `ServiceAuthMiddleware`
- **Dependencies**: Primary database
- **Data stores**: `users`, `sessions`, `service_credentials`, `audit_logs`

## Data Model

### Entities

#### Transaction
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| source | Enum | `payment_processor`, `bank_feed`, `manual` |
| external_id | String | Unique per source (dedup key) |
| amount | Decimal | > 0 |
| currency | String | ISO 4217 |
| merchant_id | String | Required |
| merchant_category | String | MCC code |
| cardholder_id | String | Required |
| timestamp | Timestamp | UTC |
| location | JSON | Lat/lon or country/city |
| channel | Enum | `card_present`, `card_not_present`, `ach`, `wire` |
| status | Enum | `ingested`, `scored`, `flagged`, `reviewed`, `resolved` |
| fraud_score | Decimal | Nullable, 0.0 to 1.0 |
| is_flagged | Boolean | Default false |
| created_at | Timestamp | UTC, auto-set |

**Indexes**: `(source, external_id)` unique, `(cardholder_id, timestamp)`, `status`, `(is_flagged, status)`

#### Feature
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| entity_type | Enum | `cardholder`, `merchant`, `transaction` |
| entity_id | String | Reference to entity |
| feature_name | String | Must match active FeatureDefinition |
| feature_value | Numeric/JSON | Computed value |
| computed_at | Timestamp | UTC, point-in-time key |
| version | Integer | Feature definition version |

**Indexes**: `(entity_type, entity_id, feature_name, computed_at)` for point-in-time lookups

#### Model
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| name | String | Required |
| version | Integer | Auto-incremented per name |
| status | Enum | `training`, `evaluating`, `staging`, `production`, `retired` |
| training_dataset_id | UUID | FK to TrainingDataset |
| metrics | JSON | Precision, recall, F1, AUC-ROC, FPR |
| feature_set | JSON | Feature names and versions used |
| artifact_path | String | Reference to model registry |
| created_by | UUID | FK to User |
| trained_at | Timestamp | UTC |
| promoted_at | Timestamp | Nullable, UTC |
| retired_at | Timestamp | Nullable, UTC |

**Indexes**: `status`, `(name, version)` unique

#### Alert
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| transaction_id | UUID | FK to Transaction, unique |
| fraud_score | Decimal | 0.0 to 1.0 |
| priority | Integer | Derived from score |
| status | Enum | `pending`, `assigned`, `reviewed` |
| assigned_to | UUID | Nullable FK to User |
| created_at | Timestamp | UTC |

**Indexes**: `(status, priority)`, `assigned_to`

#### Case
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| alert_id | UUID | FK to Alert, unique |
| analyst_id | UUID | FK to User |
| decision | Enum | `confirmed_fraud`, `dismissed`, `escalated` |
| notes | Text | Required, min 10 chars |
| evidence | JSON | Attachment references |
| decided_at | Timestamp | UTC |
| created_at | Timestamp | UTC |

#### AuditLog
| Field | Type | Constraints |
|---|---|---|
| id | UUID | Primary key |
| actor_id | UUID | FK to User |
| action | String | e.g. `model_promoted`, `case_decided`, `threshold_changed` |
| entity_type | String | e.g. `model`, `alert`, `scoring_config` |
| entity_id | String | Reference |
| details | JSON | Before/after values, context |
| timestamp | Timestamp | UTC |

**Indexes**: `timestamp`, `(entity_type, entity_id)`

**Constraint**: Append-only — no updates or deletes

## Cross-Cutting Concerns

### Authentication & Authorization

- **Mechanism**: Session-based authentication for the analyst web application with configurable session lifetime (default 8 hours)
- **RBAC enforcement**: Middleware checks role against endpoint requirements
- **Roles**: Admin (full access), Data Engineer (pipeline ops), Data Scientist (model lifecycle), Fraud Analyst (case review)
- **Service-to-service**: Pre-shared credentials via header for internal service communication

### Observability

- Structured JSON logging with correlation IDs propagated across service boundaries (ingestion through scoring through alerting)
- End-to-end latency tracing: ingestion timestamp, scoring timestamp, alert creation timestamp
- Metrics: ingestion throughput (TPS), consumer lag, scoring latency (P50/P95/P99), feature retrieval latency, flag rate, analyst throughput
- Health endpoints at `/health` for each service with dependency checks

### Rate Limiting

- Ingestion: governed by consumer group configuration and backpressure, not HTTP rate limiting
- Web application: 100 requests per minute per user for standard endpoints
- Scoring service: internal only, no HTTP rate limiting

## Non-Functional Requirements

| NFR | Target | Validation |
|---|---|---|
| Scoring latency (P99) | < 100ms | Load testing with realistic feature vectors |
| Ingestion throughput | 10,000+ TPS | Stream processing stress test |
| Consumer lag | < 30 seconds | Monitoring under sustained load |
| Feature computation latency | < 500ms after ingestion | End-to-end timing test |
| Web application response (P95) | < 200ms | Load testing analyst workflows |
| Model training | Completes within 4 hours for 1M transactions | Training benchmark |
| System availability | 99.9% | Uptime monitoring |

## Architectural Decisions (ADRs)

### ADR-001: Model Versioning Strategy

- **Status**: Accepted
- **Context**: The system continuously trains new models as analyst labels accumulate. Multiple model versions must coexist during A/B testing, and the system needs to track which model scored each transaction for auditability and feedback loop correctness.
- **Decision**: Models are versioned with auto-incrementing version numbers per model name. The model registry stores complete artifacts with metadata (feature set, training dataset, metrics). Each scored transaction records which model version produced the score. A/B tests reference specific model versions as champion and challenger.
- **Consequences**: Clear lineage from score to model to training data. Model registry may grow large over time — implement retention policy to archive old artifacts. Scoring service must support loading two model versions simultaneously during A/B tests.

### ADR-002: Feature Store Design with Point-in-Time Correctness

- **Status**: Accepted
- **Context**: Using the latest feature values for model training would introduce data leakage — the model would learn from information that was not available at the time the transaction occurred. This is a critical correctness issue in fraud detection ML systems.
- **Decision**: Every feature value is stored with a `computed_at` timestamp. Feature lookups for training use point-in-time joins: given a transaction at time T, only feature values with `computed_at <= T` are used. The feature store maintains historical values rather than overwriting them. A composite index on `(entity_type, entity_id, feature_name, computed_at)` supports efficient point-in-time queries.
- **Consequences**: Storage grows over time (historical values are retained). Query complexity increases compared to simple latest-value lookups. A data retention and compaction strategy is needed for features older than the training window. Point-in-time correctness must be verified by automated tests in the training pipeline.

### ADR-003: Scoring Latency Budget

- **Status**: Accepted
- **Context**: Fraud scoring must happen fast enough to hold suspicious transactions before they clear. The 100ms P99 budget covers the entire scoring pipeline: feature retrieval, model inference, and result persistence.
- **Decision**: The scoring service pre-loads model artifacts into memory (no per-request artifact fetching). Feature retrieval uses a low-latency key-value store optimized for single-key lookups. Score persistence is done asynchronously after the score is computed and returned. Model hot-swap uses atomic reference replacement to avoid blocking in-flight requests.
- **Consequences**: Memory requirements scale with model size. Feature store must meet low-latency SLAs. Async persistence introduces a small window where a score exists in memory but not in the database — mitigated by at-least-once persistence with retry.

### ADR-004: Hold-for-Review, Never Auto-Reject

- **Status**: Accepted
- **Context**: Auto-rejecting transactions based on ML scores would block legitimate transactions at an unacceptable rate given current false positive rates. Regulatory and business requirements mandate human review.
- **Decision**: Transactions scoring above the threshold are flagged and held for analyst review. They are never automatically blocked, reversed, or rejected by the system. The threshold is configurable by Admin at runtime. Analysts make the final decision (confirm fraud, dismiss, escalate).
- **Consequences**: Requires staffing analysts to review flagged transactions in a timely manner. Alert queue management and workload balancing become critical. The threshold must be tuned to balance analyst workload against fraud detection coverage.

### ADR-005: 7-Day Minimum A/B Test Duration

- **Status**: Accepted
- **Context**: Fraud patterns vary by day of week and time of day. A short A/B test might show misleading results due to temporal bias (e.g., a model tested only on weekdays misses weekend fraud patterns).
- **Decision**: A/B tests must run for a minimum of 7 days before a challenger model can be promoted. This ensures the test covers a full weekly cycle. The system enforces this minimum — early promotion is rejected. Data Scientists can extend the test beyond 7 days if they need more data.
- **Consequences**: Model promotion is never instantaneous — a minimum 7-day wait applies. Emergency model changes (e.g., a clearly broken model) must be handled by reverting to a previously proven model rather than fast-tracking a new one.

## Standards Compliance

- IEEE SWEBOK v4.0 -- used for module categorization (Construction, Design, Security, Operations)
- ISO/IEC/IEEE 12207 -- software lifecycle processes
- ISO/IEC/IEEE 15289 -- documentation standards applied to this document
- PCI-DSS considerations -- transaction data handling, access controls, audit trails
- Model governance -- explainability, approval gates, audit trail for all model lifecycle events

## Glossary

| Term | Definition |
|---|---|
| **Transaction** | A financial transaction event ingested from a payment processor or bank feed |
| **Feature** | A computed attribute of a cardholder, merchant, or transaction used as input to the fraud model |
| **Feature store** | A data system optimized for storing and retrieving feature values with point-in-time correctness |
| **Point-in-time correctness** | Ensuring that feature values used for training reflect only information available at the time of the transaction, preventing data leakage |
| **Data leakage** | The use of future information during model training, leading to artificially inflated evaluation metrics that do not reflect real-world performance |
| **Fraud score** | A decimal value between 0.0 and 1.0 indicating the model's estimated probability that a transaction is fraudulent |
| **Threshold** | The fraud score cutoff above which a transaction is flagged for analyst review |
| **Feature importance** | A ranking of which features contributed most to a specific prediction, used for explainability |
| **Model registry** | A versioned artifact store that tracks trained model files along with their metadata (metrics, feature set, training data) |
| **A/B test** | A controlled experiment where a fraction of transactions are scored by a challenger model while the rest use the current champion model |
| **Champion model** | The current production model used for scoring the majority of transactions |
| **Challenger model** | A candidate model being evaluated via A/B test against the champion |
| **PSI (Population Stability Index)** | A statistical measure of how much a distribution has shifted relative to a reference distribution; used for drift detection |
| **Drift** | A change in the distribution of input features or model predictions over time, which may indicate degraded model performance |
| **Dead-letter queue** | A queue where messages that cannot be processed (validation failures, parsing errors) are stored for debugging and reprocessing |
| **Alert** | A notification generated when a transaction's fraud score exceeds the configured threshold, queued for analyst review |
| **Case** | An analyst's review record for a flagged transaction, containing the decision, notes, and evidence |
