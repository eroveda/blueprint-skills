# Interface Catalog — Fraud Detection Pipeline

## Overview

The Fraud Detection Pipeline exposes three categories of interfaces:

1. **REST APIs** — used by the analyst web application and management tools
2. **Streaming Interfaces** — used for transaction ingestion and inter-service events
3. **CLI Tools** — used by Data Engineers and Data Scientists for pipeline operations

## REST APIs

### Base URL

- Production: `https://api.fraud-platform.internal/v1`
- Staging: `https://api.staging.fraud-platform.internal/v1`
- Local: `http://localhost:8080/v1`

### Authentication

#### How to Authenticate

1. Obtain a session token via the login endpoint.
2. Include in every request: `Authorization: Bearer {session_token}`
3. Session tokens expire after 8 hours (configurable). Re-authenticate when expired.

#### Login — `POST /auth/login`

```bash
curl -X POST https://api.fraud-platform.internal/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"analyst@company.com","password":"secureP@ss123"}'
```

**Response 200**:
```json
{"session_token":"tok_a1b2c3d4...","expires_in":28800,"user":{"id":"usr_x1y2z3","email":"analyst@company.com","role":"fraud_analyst"}}
```

#### Logout — `POST /auth/logout`

**Response 204**: No content. Session token is revoked.

### Common Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | `Bearer {session_token}` |
| `Content-Type` | Yes (POST/PUT) | `application/json` |
| `X-Correlation-ID` | Optional | UUID for request tracing across services |

### Common Errors

| Status | Meaning | Example Body |
|---|---|---|
| 400 | Validation error | `{"error": "amount must be greater than 0"}` |
| 401 | Missing/invalid token | `{"error": "Unauthorized"}` |
| 403 | Insufficient permissions | `{"error": "Data Scientist role required"}` |
| 404 | Resource not found | `{"error": "Alert not found"}` |
| 409 | Conflict | `{"error": "Alert already assigned to another analyst"}` |
| 422 | Business rule violation | `{"error": "A/B test must run for at least 7 days before promotion. 3 days remaining."}` |
| 429 | Rate limit exceeded | `{"error": "Too many requests", "retry_after": 30}` |

---

### Alerts

#### List Alerts — `GET /alerts?status={status}&page=1&size=20`

```bash
curl https://api.fraud-platform.internal/v1/alerts?status=pending&page=1&size=20 \
  -H "Authorization: Bearer tok_..."
```

**Response 200**:
```json
{"data":[{"id":"alt_1a2b3c","transaction_id":"txn_4d5e6f","fraud_score":0.91,"priority":1,"status":"pending","transaction_summary":{"amount":2450.00,"currency":"USD","merchant":"Electronics Store","cardholder_id":"ch_789","timestamp":"2026-04-29T14:30:00Z","channel":"card_not_present"},"created_at":"2026-04-29T14:30:02Z"}],"pagination":{"page":1,"size":20,"total_items":47,"total_pages":3}}
```

**Permissions**: Fraud Analyst (own and unassigned), Admin (all)

#### Assign Alert — `POST /alerts/{id}/assign`

```bash
curl -X POST https://api.fraud-platform.internal/v1/alerts/alt_1a2b3c/assign \
  -H "Authorization: Bearer tok_..."
```

**Response 200**: `{"id":"alt_1a2b3c","status":"assigned","assigned_to":"usr_x1y2z3"}`

**Error 409**: `{"error":"Alert already assigned to another analyst"}`

**Permissions**: Fraud Analyst (self-assign), Admin (assign to any analyst)

#### Review Alert — `GET /alerts/{id}/review`

**Response 200**:
```json
{
  "alert":{"id":"alt_1a2b3c","fraud_score":0.91,"priority":1,"status":"assigned"},
  "transaction":{"id":"txn_4d5e6f","amount":2450.00,"currency":"USD","merchant_id":"mer_abc","merchant_category":"5732","cardholder_id":"ch_789","timestamp":"2026-04-29T14:30:00Z","location":{"country":"RO","city":"Bucharest"},"channel":"card_not_present"},
  "explanation":{"top_features":[{"name":"velocity_1h","value":8,"importance":0.35,"description":"8 transactions in last hour (cardholder average: 1.2)"},{"name":"amount_deviation","value":4.2,"importance":0.28,"description":"Amount is 4.2 std deviations above cardholder mean"},{"name":"geo_distance_km","value":8500,"importance":0.18,"description":"8,500 km from last transaction location"}]},
  "cardholder_history":[{"id":"txn_prev1","amount":45.00,"merchant":"Coffee Shop","timestamp":"2026-04-29T08:15:00Z","status":"resolved"},{"id":"txn_prev2","amount":120.00,"merchant":"Grocery Store","timestamp":"2026-04-28T17:30:00Z","status":"resolved"}],
  "prior_alerts":[{"id":"alt_old1","fraud_score":0.73,"decision":"dismissed","decided_at":"2026-04-15T10:00:00Z"}]
}
```

**Permissions**: Fraud Analyst (assigned alerts), Admin (all)

#### Decide Alert — `POST /alerts/{id}/decide`

```bash
curl -X POST https://api.fraud-platform.internal/v1/alerts/alt_1a2b3c/decide \
  -H "Authorization: Bearer tok_..." -H "Content-Type: application/json" \
  -d '{"decision":"confirmed_fraud","notes":"Card-not-present from unusual country, velocity spike, amount 4x average. Cardholder confirmed unauthorized."}'
```

**Response 200**:
```json
{"case":{"id":"cas_7g8h9i","alert_id":"alt_1a2b3c","analyst_id":"usr_x1y2z3","decision":"confirmed_fraud","decided_at":"2026-04-29T15:10:00Z"},"transaction_status":"reviewed","audit_log_id":"aud_j1k2l3"}
```

**Validation**: `decision` must be one of `confirmed_fraud`, `dismissed`, `escalated`. `notes` required, minimum 10 characters.

**Error 422**: `{"error":"Notes must be at least 10 characters"}`

**Permissions**: Fraud Analyst (own assigned alerts), Admin (all)

---

### Cases

#### List Cases — `GET /cases?decision={decision}&from={date}&to={date}&page=1&size=20`

**Response 200**:
```json
{"data":[{"id":"cas_7g8h9i","alert_id":"alt_1a2b3c","analyst_id":"usr_x1y2z3","decision":"confirmed_fraud","notes":"Card-not-present from unusual country...","decided_at":"2026-04-29T15:10:00Z"}],"pagination":{"page":1,"size":20,"total_items":312,"total_pages":16}}
```

**Permissions**: Fraud Analyst (own cases), Admin (all cases)

---

### Models

#### List Models — `GET /models?status={status}&page=1&size=20`

**Response 200**:
```json
{"data":[{"id":"mod_1a2b3c","name":"fraud_classifier","version":12,"status":"production","metrics":{"precision":0.87,"recall":0.82,"f1":0.845,"auc_roc":0.94,"false_positive_rate":0.03},"trained_at":"2026-04-22T02:00:00Z","promoted_at":"2026-04-25T14:00:00Z"}],"pagination":{"page":1,"size":20,"total_items":15,"total_pages":1}}
```

**Permissions**: Data Scientist, Admin

#### Get Model Details — `GET /models/{id}`

**Response 200**:
```json
{
  "id":"mod_1a2b3c","name":"fraud_classifier","version":12,"status":"production",
  "metrics":{"precision":0.87,"recall":0.82,"f1":0.845,"auc_roc":0.94,"false_positive_rate":0.03},
  "feature_set":["velocity_1h","velocity_24h","amount_deviation","geo_distance_km","time_since_last","merchant_risk_90d","spending_profile_dow"],
  "training_dataset":{"id":"ds_4d5e6f","snapshot_date":"2026-04-21","record_count":245000,"label_distribution":{"fraud":4200,"legitimate":240800}},
  "threshold_analysis":[{"threshold":0.5,"precision":0.72,"recall":0.93},{"threshold":0.7,"precision":0.87,"recall":0.82},{"threshold":0.9,"precision":0.95,"recall":0.61}],
  "created_by":"usr_ds_abc","trained_at":"2026-04-22T02:00:00Z"
}
```

#### Train Model — `POST /models/train`

```bash
curl -X POST https://api.fraud-platform.internal/v1/models/train \
  -H "Authorization: Bearer tok_..." -H "Content-Type: application/json" \
  -d '{"name":"fraud_classifier","feature_set":["velocity_1h","velocity_24h","amount_deviation","geo_distance_km","time_since_last","merchant_risk_90d"],"hyperparameters":{"learning_rate":0.01,"max_depth":8},"date_range":{"start":"2026-01-01","end":"2026-04-21"}}'
```

**Response 202**:
```json
{"model_id":"mod_new_xyz","status":"training","message":"Training job started. Poll GET /models/mod_new_xyz for status updates."}
```

**Permissions**: Data Scientist

#### Compare Models — `GET /models/compare?model_a={id}&model_b={id}`

**Response 200**:
```json
{
  "model_a":{"id":"mod_1a2b3c","version":12,"metrics":{"precision":0.87,"recall":0.82,"f1":0.845,"auc_roc":0.94,"false_positive_rate":0.03}},
  "model_b":{"id":"mod_new_xyz","version":13,"metrics":{"precision":0.89,"recall":0.84,"f1":0.865,"auc_roc":0.95,"false_positive_rate":0.028}},
  "comparison":{"precision_delta":"+0.02","recall_delta":"+0.02","fpr_delta":"-0.002","recommendation":"Challenger shows improvement across all metrics. Consider promoting to staging."}
}
```

#### Promote to Staging — `POST /models/{id}/promote-to-staging`

**Response 200**: `{"model_id":"mod_new_xyz","status":"staging","ab_test_id":"abt_9a8b7c","message":"Model promoted to staging. A/B test created with 10% traffic to challenger."}`

**Error 422**: `{"error":"Model must be in evaluating status to promote to staging"}`

#### Reject Model — `POST /models/{id}/reject`

**Response 200**: `{"model_id":"mod_new_xyz","status":"retired","message":"Model rejected and retired."}`

**Permissions** (all model actions): Data Scientist

---

### A/B Tests

#### List A/B Tests — `GET /ab-tests?status={status}`

**Response 200**:
```json
{"data":[{"id":"abt_9a8b7c","champion":{"id":"mod_1a2b3c","version":12},"challenger":{"id":"mod_new_xyz","version":13},"traffic_split":0.1,"start_date":"2026-04-25","min_duration_days":7,"days_elapsed":4,"status":"active"}]}
```

#### Get A/B Test Results — `GET /ab-tests/{id}`

**Response 200**:
```json
{
  "id":"abt_9a8b7c","status":"active","days_elapsed":4,"min_duration_days":7,"days_remaining":3,
  "champion":{"model_id":"mod_1a2b3c","transactions_scored":18200,"metrics":{"precision":0.86,"recall":0.81,"false_positive_rate":0.032,"alert_rate":0.021}},
  "challenger":{"model_id":"mod_new_xyz","transactions_scored":2020,"metrics":{"precision":0.88,"recall":0.83,"false_positive_rate":0.029,"alert_rate":0.023}}
}
```

#### Promote Challenger — `POST /ab-tests/{id}/promote`

**Response 200**: `{"ab_test_id":"abt_9a8b7c","status":"completed","promoted_model":"mod_new_xyz","retired_model":"mod_1a2b3c","message":"Challenger promoted to production. Old champion retired."}`

**Error 422**: `{"error":"A/B test must run for at least 7 days before promotion. 3 days remaining."}`

#### Cancel A/B Test — `POST /ab-tests/{id}/cancel`

**Response 200**: `{"ab_test_id":"abt_9a8b7c","status":"cancelled","message":"A/B test cancelled. Champion continues serving all traffic."}`

**Permissions** (all A/B test actions): Data Scientist

---

### Scoring Configuration

#### Get Scoring Config — `GET /scoring/config`

**Response 200**:
```json
{"threshold":0.7,"flag_rate_current":0.021,"flag_rate_estimated_at_0_65":0.035,"flag_rate_estimated_at_0_75":0.014}
```

#### Update Scoring Config — `PUT /scoring/config`

```bash
curl -X PUT https://api.fraud-platform.internal/v1/scoring/config \
  -H "Authorization: Bearer tok_..." -H "Content-Type: application/json" \
  -d '{"threshold":0.75}'
```

**Response 200**: `{"threshold":0.75,"message":"Threshold updated. Change recorded in audit log.","audit_log_id":"aud_m1n2o3"}`

**Permissions**: Admin

---

### Monitoring

#### Model Drift — `GET /monitoring/models/{id}/drift`

**Response 200**:
```json
{
  "model_id":"mod_1a2b3c",
  "prediction_drift":{"psi":0.08,"status":"stable","reference_period":"2026-04-01 to 2026-04-14","current_period":"2026-04-15 to 2026-04-29"},
  "feature_drift":[
    {"feature":"velocity_1h","psi":0.05,"status":"stable"},
    {"feature":"merchant_risk_90d","psi":0.31,"status":"drifted"},
    {"feature":"amount_deviation","psi":0.07,"status":"stable"}
  ]
}
```

#### Model Performance — `GET /monitoring/models/{id}/performance`

**Response 200**:
```json
{
  "model_id":"mod_1a2b3c",
  "rolling_7d":{"precision":0.85,"recall":0.80,"false_positive_rate":0.035,"labeled_count":420},
  "rolling_30d":{"precision":0.87,"recall":0.82,"false_positive_rate":0.030,"labeled_count":1850},
  "evaluation_baseline":{"precision":0.87,"recall":0.82,"false_positive_rate":0.030}
}
```

#### Monitoring Summary — `GET /monitoring/summary`

**Response 200**:
```json
{
  "production_models":[
    {"model_id":"mod_1a2b3c","name":"fraud_classifier","version":12,"drift_status":"stable","performance_status":"healthy","training_data_age_days":8,"alert_rate_anomaly":false}
  ]
}
```

**Permissions** (monitoring): Data Scientist, Admin

---

### Dashboards

#### Analyst Dashboard — `GET /dashboard/analyst`

**Response 200**:
```json
{"cases_today":12,"cases_this_week":47,"avg_review_time_minutes":4.2,"decisions":{"confirmed_fraud":5,"dismissed":6,"escalated":1},"team_average_cases_today":10}
```

**Permissions**: Fraud Analyst (own), Admin (any analyst)

#### Fraud Dashboard — `GET /dashboard/fraud`

**Response 200**:
```json
{"period":"2026-04-29","daily_fraud_rate":0.0031,"false_positive_rate":0.42,"analyst_throughput":{"avg_cases_per_analyst_per_day":11.5},"avg_review_time_minutes":4.8,"total_alerts_pending":23}
```

**Permissions**: Admin

---

### Ingestion Health

#### Get Ingestion Health — `GET /ingestion/health`

**Response 200**:
```json
{"throughput_tps":8542,"consumer_lag_seconds":3,"validation_failure_rate":0.002,"dead_letter_queue_depth":7,"uptime_hours":432}
```

#### Retry Dead-Letter Message — `POST /ingestion/dead-letter/{id}/retry`

**Response 200**: `{"id":"dl_abc123","status":"reprocessed","result":"success","transaction_id":"txn_new_456"}`

**Error 422**: `{"id":"dl_abc123","status":"reprocessed","result":"failed","error":"Currency code XYZ still not recognized"}`

**Permissions** (ingestion): Data Engineer, Admin

---

### Feature Definitions

#### List Feature Definitions — `GET /features?computation_type={type}&is_active={bool}`

**Response 200**:
```json
{"data":[{"id":"fd_1a2b3c","name":"velocity_1h","description":"Transaction count per cardholder in 1-hour window","computation_type":"realtime","parameters":{"window_seconds":3600,"aggregation":"count"},"version":2,"is_active":true}]}
```

#### Create Feature Definition — `POST /features`

**Response 201**:
```json
{"id":"fd_new_xyz","name":"merchant_category_risk","computation_type":"batch","parameters":{"window_days":90,"aggregation":"fraud_rate"},"version":1,"is_active":true}
```

**Permissions** (features): Data Scientist, Data Engineer

---

### Users

#### List Users — `GET /users`

**Response 200**:
```json
{"data":[{"id":"usr_x1y2z3","email":"analyst@company.com","first_name":"Maria","last_name":"Rodriguez","role":"fraud_analyst","is_active":true}]}
```

#### Create User — `POST /users`

**Response 201**: `{"id":"usr_new_abc","email":"newuser@company.com","role":"data_scientist","is_active":true}`

**Permissions** (users): Admin

---

### Audit Trail

#### List Audit Logs — `GET /audit-logs?action={action}&from={date}&to={date}&actor_id={id}&page=1&size=20`

**Response 200**:
```json
{"data":[{"id":"aud_j1k2l3","actor_id":"usr_x1y2z3","action":"case_decided","entity_type":"alert","entity_id":"alt_1a2b3c","details":{"decision":"confirmed_fraud"},"timestamp":"2026-04-29T15:10:00Z"}],"pagination":{"page":1,"size":20,"total_items":1240,"total_pages":62}}
```

**Permissions**: Admin

---

### Labels

#### Label Statistics — `GET /labels/stats`

**Response 200**:
```json
{"since_last_training":"2026-04-22T02:00:00Z","new_labels":{"total":312,"fraud":52,"legitimate":260},"by_date":[{"date":"2026-04-28","fraud":8,"legitimate":42},{"date":"2026-04-29","fraud":5,"legitimate":17}]}
```

**Permissions**: Data Scientist

---

## Streaming Interfaces

### Topics

| Topic | Producer | Consumer(s) | Message Format |
|---|---|---|---|
| `transactions.raw.{source}` | Payment processor / Bank feed | Ingestion Service | Source-specific (validated by ingestion) |
| `transactions.ingested` | Ingestion Service | Feature Engineering, Scoring Service | Canonical transaction format (see below) |
| `transactions.dead-letter` | Ingestion Service | Data Engineer (manual reprocessing) | Original message + error metadata |

### Canonical Transaction Message

Published to `transactions.ingested` after validation and normalization:

```json
{
  "transaction_id": "txn_4d5e6f",
  "source": "payment_processor",
  "external_id": "pp_98765",
  "amount": 2450.00,
  "currency": "USD",
  "merchant_id": "mer_abc",
  "merchant_category": "5732",
  "cardholder_id": "ch_789",
  "timestamp": "2026-04-29T14:30:00Z",
  "location": {"country": "RO", "city": "Bucharest", "lat": 44.4268, "lon": 26.1025},
  "channel": "card_not_present",
  "correlation_id": "corr_a1b2c3d4"
}
```

### Dead-Letter Message

Published to `transactions.dead-letter` when validation fails:

```json
{
  "original_message": "{raw message bytes}",
  "source_topic": "transactions.raw.payment_processor_a",
  "error": "Missing required field: cardholder_id",
  "error_code": "VALIDATION_MISSING_FIELD",
  "failed_at": "2026-04-29T14:30:01Z",
  "correlation_id": "corr_x1y2z3"
}
```

### Consumer Group Configuration

| Consumer Group | Topic | Behavior |
|---|---|---|
| `ingestion-service` | `transactions.raw.*` | Parallel consumption with offset tracking; at-least-once delivery |
| `feature-engine` | `transactions.ingested` | Real-time feature computation triggered per message |
| `scoring-service` | `transactions.ingested` | Real-time scoring triggered per message |

### Backpressure

- Consumer batch size and concurrency are configurable per consumer group.
- When downstream systems (database, feature store) saturate, consumers slow their consumption rate.
- Consumer lag is monitored and alerts fire when lag exceeds 30 seconds.

---

## CLI Tools

### Model Training CLI

Used by Data Scientists for batch operations and automation:

```bash
# Trigger a training run
fraud-cli models train \
  --name fraud_classifier \
  --features velocity_1h,velocity_24h,amount_deviation,geo_distance_km \
  --date-range 2026-01-01:2026-04-21 \
  --hyperparams '{"learning_rate":0.01,"max_depth":8}'

# List models
fraud-cli models list --status evaluating

# Compare two models
fraud-cli models compare --model-a mod_1a2b3c --model-b mod_new_xyz

# Promote model to staging
fraud-cli models promote --id mod_new_xyz --to staging
```

### Feature Management CLI

Used by Data Engineers and Data Scientists:

```bash
# List active feature definitions
fraud-cli features list --active

# Create a new feature definition
fraud-cli features create \
  --name merchant_category_risk \
  --type batch \
  --params '{"window_days":90,"aggregation":"fraud_rate"}'

# Trigger batch feature computation
fraud-cli features compute-batch --date 2026-04-28

# Check feature freshness
fraud-cli features freshness --alert-threshold 2h
```

### Ingestion CLI

Used by Data Engineers:

```bash
# Check ingestion health
fraud-cli ingestion health

# List dead-letter messages
fraud-cli ingestion dead-letter list --limit 10

# Retry a dead-letter message
fraud-cli ingestion dead-letter retry --id dl_abc123

# Retry all dead-letter messages from the last hour
fraud-cli ingestion dead-letter retry-all --since 1h
```

---

## Pagination

All list endpoints support pagination:

| Parameter | Default | Description |
|---|---|---|
| `page` | 1 | Page number (1-based) |
| `size` | 20 | Items per page (max 100) |

Response includes: `{"pagination":{"page":1,"size":20,"total_items":1432,"total_pages":72}}`

## Rate Limiting

| Endpoint Group | Limit |
|---|---|
| Authentication (login) | 10 requests/minute |
| Standard API endpoints | 100 requests/minute per user |
| Monitoring endpoints | 30 requests/minute per user |

**Response headers**: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (unix timestamp).

Exceeding the limit returns `429 Too Many Requests` with `retry_after` in the body.

## Service-to-Service Authentication

Internal services authenticate using pre-shared credentials:

| Header | Description |
|---|---|
| `X-Service-Auth` | Hashed credential identifying the calling service |
| `X-Service-Name` | Name of the calling service (for logging) |

Permissions are defined per service credential:
- Scoring service: `read_features`, `write_scores`
- Ingestion service: `write_transactions`, `publish_events`
- Feature engine: `read_transactions`, `write_features`
- Training service: `read_features`, `read_transactions`, `write_models`

## Versioning

- **Current version**: v1 (all endpoints prefixed with `/v1/`)
- **Deprecation policy**: 6 months of support after successor release, communicated via `Sunset` response header.
