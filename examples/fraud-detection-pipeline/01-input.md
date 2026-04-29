# Fraud Detection Pipeline — Input

## The Idea

A real-time fraud detection system that ingests financial transactions, extracts features, trains ML models, scores transactions in real-time, and triggers alerts for suspicious activity.

## Actors

- **Data Engineer** — manages data pipelines, monitors ingestion health, configures feature stores
- **Data Scientist** — trains and evaluates models, experiments with features, deploys model versions
- **Fraud Analyst** — reviews flagged transactions, confirms/dismisses fraud cases, provides labeled data
- **Admin** — manages users, configures alert thresholds, views system health

## Key Features

- Transaction ingestion from multiple sources (payment processors, bank feeds) via streaming
- Feature engineering: real-time features (velocity, amount deviation) and batch features (historical patterns)
- Model training pipeline: data splitting, training, evaluation, model registry
- Real-time scoring: sub-100ms latency per transaction
- Alert management: flagged transactions queue, analyst review workflow, feedback loop
- Model monitoring: drift detection, performance degradation alerts, A/B testing

## Business Rules

- Transactions scoring above threshold are held for review (not auto-rejected)
- Analyst decisions feed back into training data for next model version
- Model deployment requires approval from at least one Data Scientist
- A/B testing runs for minimum 7 days before a model can be promoted
- Feature store maintains point-in-time correctness (no data leakage)

## Constraints

- Scoring latency must be < 100ms at P99
- Transaction volume: 10,000+ per second at peak
- Model retraining runs daily on previous day's labeled data
- All predictions must be explainable (feature importance per decision)
- Audit trail required for every analyst decision
