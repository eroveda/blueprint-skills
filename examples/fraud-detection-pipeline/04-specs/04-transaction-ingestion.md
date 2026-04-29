# Spec: Transaction Ingestion Pipeline

```yaml
title: "Transaction Ingestion Pipeline"
purpose: "Implement streaming transaction consumption, validation, deduplication, dead-letter routing, and backpressure handling at 10,000+ TPS"
category: "CONSTRUCTION"

input_context: |
  From "Transaction, Feature, Model, and Case Data Models":
  - Transaction table and model class with fields: id, source, external_id, amount, currency, merchant_id, merchant_category, cardholder_id, timestamp, location, channel, status (enum: ingested, scored, flagged, reviewed, resolved), fraud_score, is_flagged, created_at
  - Transaction status state machine module with allowed transitions
  - Validation rules: amount > 0, currency is valid ISO 4217

  From "Authentication and Role-Based Access":
  - Service-to-service authentication for inter-service calls
  - Audit log recording function

  From "Project Bootstrap and Environment Setup":
  - Message broker connection and consumer group configuration
  - Structured logging with correlation IDs

instructions:
  - "Implement a streaming consumer service that subscribes to transaction topics from payment processor feeds, using consumer groups for horizontal scalability and offset tracking for at-least-once delivery"
  - "Define the canonical transaction message schema with required fields (external_id, amount, currency, merchant_id, cardholder_id, timestamp, channel) and optional fields (merchant_category, location); validate every incoming message against this schema"
  - "Implement transaction normalization: convert currencies to a canonical format (preserve original, add normalized fields), parse timestamps to UTC, normalize merchant category codes to a standard taxonomy"
  - "Implement deduplication using the composite key (source, external_id): check for existing transactions before insert; if a duplicate is found, skip processing and log a dedup event"
  - "Implement dead-letter queue routing: messages that fail schema validation or normalization are routed to a dead-letter topic with the original message, error details, and timestamp; provide a reprocessing endpoint for Data Engineers to retry dead-letter messages"
  - "Implement backpressure management: configure consumer batch sizes and processing concurrency to sustain 10,000+ transactions per second at peak; implement flow control that slows consumption when downstream systems (database, feature store) are saturated"
  - "Persist validated and normalized transactions to the database with status 'ingested' and publish a 'transaction.ingested' event to a downstream topic for the feature engineering and scoring services"
  - "Implement ingestion health monitoring: expose metrics for messages consumed per second, validation failure rate, deduplication rate, dead-letter queue depth, and consumer lag"
  - "Implement a Data Engineer management endpoint: GET /ingestion/health to view current throughput, lag, and error rates; POST /ingestion/dead-letter/{id}/retry to reprocess individual dead-letter messages"
  - "Apply service-to-service authentication to all inter-service communications"
  - "Write unit tests for schema validation, normalization logic, and deduplication checks"
  - "Write integration tests for end-to-end message consumption, dead-letter routing, and reprocessing"

constraints:
  - "Must sustain 10,000+ transactions per second at peak without message loss"
  - "At-least-once delivery semantics — deduplication prevents duplicate processing, but no messages are silently dropped"
  - "Dead-letter messages must preserve the original payload for debugging and reprocessing"
  - "Consumer lag must not exceed 30 seconds under normal load"
  - "All ingested transactions must have a correlation ID for end-to-end tracing through scoring and alerting"

acceptance_criteria:
  - "Consuming a valid transaction message persists it with status 'ingested' and publishes a downstream event"
  - "Consuming a message with missing required fields routes it to the dead-letter queue with an error description"
  - "Consuming a duplicate message (same source + external_id) skips processing without error"
  - "Throughput test sustains 10,000 messages per second with consumer lag under 30 seconds"
  - "GET /ingestion/health returns current throughput, lag, and error metrics"
  - "POST /ingestion/dead-letter/{id}/retry reprocesses the message and either persists it or returns it to dead-letter with updated error"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Transaction, Feature, Model, and Case Data Models"
    - "Authentication and Role-Based Access"
  integrates_with:
    - "Feature Engineering Service"
    - "Real-time Scoring Service"

handoff: |
  Exposes for downstream specs:
  - Validated and normalized transactions persisted in the database with status 'ingested'
  - 'transaction.ingested' events published to downstream topic for feature engineering and scoring
  - Ingestion health metrics for operational monitoring
  - Dead-letter queue management endpoints for Data Engineers
  - Correlation ID attached to every transaction for end-to-end tracing

verification: "run project test suite"
```
