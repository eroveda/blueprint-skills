# Spec: Model Training Pipeline

```yaml
title: "Model Training Pipeline"
purpose: "Implement training dataset assembly, model training, evaluation, registry management, and candidate comparison"
category: "CONSTRUCTION"

input_context: |
  From "Transaction, Feature, Model, and Case Data Models":
  - Model table with name, version, status (training, evaluating, staging, production, retired), metrics, feature_set, artifact_path
  - Model lifecycle state machine module
  - TrainingDataset table with snapshot_date, record_count, label_distribution, feature_version

  From "Feature Engineering Service":
  - Feature store with point-in-time correct feature values
  - Feature retrieval API for assembling feature vectors at a given timestamp
  - FeatureDefinition management with versioning

  From "Project Bootstrap and Environment Setup":
  - Model registry storage interface for versioned model artifacts

instructions:
  - "Implement training dataset assembly: query labeled transactions (those with analyst decisions from the feedback loop or historical labels), join with point-in-time feature vectors from the feature store using each transaction's timestamp to prevent data leakage, and produce a tabular dataset with features and binary label (fraud / legitimate)"
  - "Implement temporal data splitting: split the assembled dataset so that training data comes from older time periods and validation/test data from more recent periods; never use random splitting, which would leak future information into training"
  - "Create a TrainingDataset record for each assembled dataset with snapshot_date, date ranges, record count, and label distribution (fraud count vs legitimate count)"
  - "Implement model training execution: accept a training configuration (hyperparameters, feature set, training dataset ID), train a model using the assembled dataset, and store the trained artifact in the model registry"
  - "Implement model evaluation: compute metrics on the held-out test set — precision, recall, F1 score, AUC-ROC, and false positive rate at the configured scoring threshold; store metrics in the Model entity"
  - "Implement threshold analysis: for each trained model, compute precision and recall at multiple threshold values (e.g. 0.3 to 0.9 in steps of 0.05) to help Data Scientists choose the optimal operating point"
  - "Register trained models in the model registry with: model name, auto-incremented version, artifact path, feature set reference (which features and versions were used), training dataset reference, evaluation metrics, and the creating user"
  - "Implement model comparison endpoint: given two model IDs (typically current production vs candidate), display side-by-side metrics, threshold analysis curves, and a recommendation based on configurable criteria (e.g. improve recall without increasing false positive rate by more than 5%)"
  - "Implement model listing and detail endpoints for Data Scientists: list models with filtering by status and name, view model details including metrics, feature set, training data summary, and threshold analysis"
  - "Create a model training trigger endpoint: POST /models/train that accepts training configuration and enqueues a training job; return the job ID and model ID for polling status"
  - "Apply role-based access: only Data Scientists can trigger training, view models, and compare candidates"
  - "Write unit tests for temporal data splitting (verify no future data in training set), metric computation, and threshold analysis"
  - "Write integration tests for end-to-end training: assemble dataset, train model, evaluate, and register in the model registry"

constraints:
  - "Training data assembly must use point-in-time feature joins — using features computed after a transaction's timestamp is a critical data leakage bug"
  - "Data splitting must be temporal, never random — the validation and test sets must contain only transactions that occurred after all training transactions"
  - "Model training must be stack-agnostic — the pipeline accepts any model artifact format and does not prescribe a specific ML framework"
  - "Model evaluation must compute all five metrics (precision, recall, F1, AUC-ROC, false positive rate) — partial evaluation is not acceptable"
  - "Model artifacts must be stored with sufficient metadata to reproduce the training run (hyperparameters, feature set version, training dataset ID)"

acceptance_criteria:
  - "Training dataset assembly joins features at each transaction's timestamp, not the latest feature values"
  - "Temporal split places all training transactions before all validation/test transactions chronologically"
  - "Training a model produces an artifact stored in the model registry with status 'evaluating'"
  - "Model evaluation computes and stores precision, recall, F1, AUC-ROC, and false positive rate"
  - "Threshold analysis returns precision-recall pairs at multiple thresholds"
  - "Model comparison endpoint displays side-by-side metrics for two models"
  - "POST /models/train returns a job ID and model ID; the model transitions from 'training' to 'evaluating' on completion"
  - "All tests pass with project test suite"

dependencies:
  hard:
    - "Transaction, Feature, Model, and Case Data Models"
    - "Feature Engineering Service"
  integrates_with:
    - "Real-time Scoring Service"
    - "Analyst Feedback Loop"

handoff: |
  Exposes for downstream specs:
  - Trained model artifacts in the model registry, ready for deployment to the scoring service
  - Model evaluation reports and threshold analysis for Data Scientists
  - Model comparison endpoint for promotion decisions
  - TrainingDataset records tracking data provenance
  - Training trigger endpoint for automated retraining from the feedback loop

verification: "run project test suite"
```
