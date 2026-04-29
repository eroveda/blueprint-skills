# Spec: Feature Engineering Service

```yaml
title: "Feature Engineering Service"
purpose: "Implement real-time and batch feature computation, feature store management with point-in-time correctness, and feature versioning"
category: "CONSTRUCTION"

input_context: |
  From "Transaction, Feature, Model, and Case Data Models":
  - Transaction table with cardholder_id, amount, timestamp, merchant_category, location, channel
  - Feature table with entity_type, entity_id, feature_name, feature_value, computed_at, version
  - FeatureDefinition table with name, computation_type, parameters, version

  From "Transaction Ingestion Pipeline":
  - 'transaction.ingested' events published to downstream topic
  - Validated and normalized transactions in the database

  From "Project Bootstrap and Environment Setup":
  - Feature store connection and read/write interface

instructions:
  - "Implement a real-time feature computation service that subscribes to 'transaction.ingested' events and computes features immediately for each incoming transaction"
  - "Implement real-time features: transaction velocity (count of transactions per cardholder in configurable sliding windows — e.g. 1 hour, 24 hours, 7 days), amount deviation (z-score of current amount vs cardholder's historical mean and standard deviation), time since last transaction (seconds since cardholder's previous transaction), geographic distance from last transaction (haversine distance between current and previous location)"
  - "Implement batch feature computation as a scheduled job that runs periodically (configurable, default nightly): historical spending pattern by merchant category (average amount per category over 30/60/90 days), day-of-week and time-of-day spending profile (average transaction count and amount by day and hour), merchant risk score (fraud rate observed at each merchant over trailing 90 days), account-level aggregate features (total transaction count, average amount, distinct merchants over 30 days)"
  - "Store all computed features in the feature store with point-in-time correctness: each feature value is stored with a computed_at timestamp; feature lookups for training must return the value that was current as of the transaction's timestamp, not the latest value — this prevents data leakage"
  - "Implement a feature retrieval API that assembles a complete feature vector for a given entity (cardholder or transaction) at a given point in time, returning all active features with their values"
  - "Implement feature versioning: when a FeatureDefinition is updated (e.g. window size changed), increment the version; new feature values are stored under the new version while old values remain queryable for backward compatibility"
  - "Implement feature freshness monitoring: track the timestamp of the latest computed value for each active feature; alert when any feature is staler than a configurable threshold (default: 2x the expected computation interval)"
  - "Implement FeatureDefinition management endpoints for Data Scientists and Data Engineers: CRUD operations on feature definitions with validation of parameters, list active features, view computation history"
  - "Write unit tests for each real-time feature computation (velocity, deviation, time gap, distance) with known input/output pairs"
  - "Write unit tests for point-in-time retrieval: verify that querying features at time T returns values computed before T, not after"
  - "Write integration tests for end-to-end feature computation triggered by a transaction event"

constraints:
  - "Point-in-time correctness is mandatory — feature lookups for model training must never include features computed after the transaction's timestamp"
  - "Real-time features must be computed and stored within 500ms of transaction ingestion to be available for scoring"
  - "Feature versioning must be backward-compatible — old feature versions remain queryable even after new versions are created"
  - "Batch feature jobs must be idempotent — running the same job twice for the same date produces identical results"
  - "Feature freshness alerts must fire within 5 minutes of a feature exceeding its staleness threshold"

acceptance_criteria:
  - "A 'transaction.ingested' event triggers real-time feature computation and stores results in the feature store within 500ms"
  - "Transaction velocity feature correctly counts transactions in the configured sliding window"
  - "Amount deviation feature computes the correct z-score against historical cardholder data"
  - "Feature retrieval at time T returns feature values with computed_at <= T, ignoring values computed after T"
  - "Batch feature job computes and stores all batch features for the configured time range"
  - "Running the batch job twice for the same date produces identical feature values"
  - "Feature freshness alert fires when a feature exceeds the staleness threshold"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Transaction, Feature, Model, and Case Data Models"
    - "Transaction Ingestion Pipeline"
  integrates_with:
    - "Real-time Scoring Service"
    - "Model Training Pipeline"

handoff: |
  Exposes for downstream specs:
  - Feature store populated with real-time and batch features, queryable with point-in-time correctness
  - Feature retrieval API for assembling feature vectors (used by Scoring and Training)
  - FeatureDefinition management endpoints for Data Scientists and Data Engineers
  - Feature freshness monitoring and alerting
  - Feature versioning with backward-compatible lookups

verification: "run project test suite"
```
