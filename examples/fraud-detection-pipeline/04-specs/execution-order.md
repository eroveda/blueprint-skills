# Spec Execution Order — Fraud Detection Pipeline

```yaml
execution_order:
  - phase: 1
    specs: ["Project Bootstrap and Environment Setup"]
    reason: "No dependencies — must run first"
  - phase: 2
    specs: ["Transaction, Feature, Model, and Case Data Models", "Authentication and Role-Based Access"]
    reason: "Both depend only on Bootstrap — can run in parallel"
  - phase: 3
    specs: ["Transaction Ingestion Pipeline"]
    reason: "Depends on Data Models and Auth"
  - phase: 4
    specs: ["Feature Engineering Service", "Real-time Scoring Service"]
    reason: "Both depend on Data Models and Ingestion — can run in parallel (Scoring also uses Feature Engineering at runtime but can be built independently with mocked features)"
  - phase: 5
    specs: ["Model Training Pipeline", "Alert and Case Management", "Model Monitoring and Drift Detection"]
    reason: "Training depends on Feature Engineering; Alert Management depends on Scoring; Monitoring depends on Scoring — all can run in parallel"
  - phase: 6
    specs: ["Analyst Feedback Loop"]
    reason: "Depends on Training Pipeline and Alert Management — must run last"
```
