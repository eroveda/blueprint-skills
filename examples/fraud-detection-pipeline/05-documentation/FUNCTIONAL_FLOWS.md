# Functional Flows — Fraud Detection Pipeline

## Actors

| Actor | Role | Permissions |
|---|---|---|
| Data Engineer | Manages data pipelines, monitors ingestion health, configures feature stores | Read/write ingestion config, feature definitions, pipeline monitoring; cannot review cases or manage models |
| Data Scientist | Trains and evaluates models, manages feature definitions, approves model promotions | Read/write models, training datasets, A/B tests, feature definitions; cannot review cases or manage users |
| Fraud Analyst | Reviews flagged transactions, makes fraud decisions, provides labeled data | Read/write alerts and cases; view transaction details and score explanations; cannot manage models or pipelines |
| Admin | Manages users, configures thresholds, views system health | Full access to all features; user management; threshold configuration; audit trail exports |

## Flows by Actor

### Data Engineer

#### Flow: Monitor Ingestion Pipeline Health

**Goal**: Verify that transaction ingestion is running normally and identify any issues.

**Steps**:
1. The Data Engineer opens the Ingestion Health dashboard.
2. The system displays current throughput (transactions per second), consumer lag, validation failure rate, and dead-letter queue depth.
3. The Data Engineer checks that throughput matches expected levels and consumer lag is under 30 seconds.
4. If the dead-letter queue depth is non-zero, the Data Engineer clicks into the dead-letter view.

**Outcome**: The Data Engineer has confirmed pipeline health or identified issues requiring attention.

**Possible errors**:
- Consumer lag is increasing -- indicates the ingestion service is falling behind. The Data Engineer checks for downstream bottlenecks (database write latency, feature store saturation) and considers scaling consumers.
- Dead-letter queue is growing -- indicates a new source is sending malformed data. The Data Engineer examines sample messages, corrects the schema mapping, and reprocesses.

#### Flow: Reprocess Dead-Letter Messages

**Goal**: Fix and reprocess transactions that failed validation during ingestion.

**Steps**:
1. The Data Engineer views the dead-letter queue with error details for each message.
2. The Data Engineer identifies the root cause (e.g., new currency code not in the allowed list, unexpected field format).
3. The Data Engineer fixes the configuration or validation rules.
4. The Data Engineer triggers reprocessing for individual messages or in bulk.
5. The system reprocesses each message through the validation pipeline.
6. Successfully processed messages enter the normal pipeline; still-failing messages return to the dead-letter queue with updated error details.

**Outcome**: Recoverable transactions are processed; persistent failures are documented for source correction.

#### Flow: Manage Feature Definitions

**Goal**: Add or update a feature definition used in the scoring pipeline.

**Steps**:
1. The Data Engineer opens the Feature Definitions management page.
2. The Data Engineer creates a new feature definition (e.g., "merchant_risk_90d" with computation type "batch", parameters including 90-day window and fraud rate aggregation).
3. The system validates the parameters and saves the definition.
4. The Data Engineer monitors the next batch computation run to confirm the feature is computed correctly.

**Outcome**: The new feature is available for inclusion in model training.

### Data Scientist

#### Flow: Train a New Model

**Goal**: Train a fraud detection model using the latest labeled data and features.

**Steps**:
1. The Data Scientist opens the Model Training section.
2. The Data Scientist configures a training run: selects the feature set (which feature definitions to include), sets hyperparameters, and specifies the training data date range.
3. The Data Scientist submits the training request.
4. The system assembles the training dataset from the feature store using point-in-time joins, splits the data temporally, trains the model, and evaluates it.
5. When training completes, the Data Scientist views the evaluation report: precision, recall, F1, AUC-ROC, false positive rate, and threshold analysis curves.
6. The Data Scientist compares the new model against the current production model side-by-side.

**Outcome**: A new model candidate is available in the registry with evaluation metrics.

**Possible errors**:
- Insufficient labeled data -- the system warns that the dataset has fewer than the minimum labeled transactions. The Data Scientist waits for more analyst decisions to accumulate.
- Feature store query fails for point-in-time join -- indicates a feature has missing historical values. The Data Scientist checks feature freshness and backfills if needed.

#### Flow: Promote a Model Through A/B Testing

**Goal**: Move a trained model into production after validating it against live traffic.

**Steps**:
1. The Data Scientist reviews the evaluation metrics of a candidate model in "evaluating" status.
2. If the metrics look promising, the Data Scientist promotes the model to "staging."
3. The system automatically creates an A/B test: the current production model is the champion, the staged model is the challenger, and 10% of traffic is routed to the challenger.
4. The A/B test runs for at least 7 days. The Data Scientist monitors comparative metrics daily.
5. After 7 days, the Data Scientist reviews the final A/B test results: precision, recall, false positive rate, and alert rate for both models.
6. If the challenger performs better (or at least as well), the Data Scientist promotes it to production.
7. The system swaps the production model, retires the old champion, records the promotion in the audit log, and completes the A/B test.

**Outcome**: The new model is serving production traffic and the old model is retired.

**Possible errors**:
- Attempting to promote before 7 days -- the system rejects the request and shows how many days remain.
- The challenger performs worse -- the Data Scientist cancels the A/B test and the champion continues serving all traffic.
- Only one A/B test can be active at a time -- attempting to start a second returns an error.

#### Flow: Investigate Model Drift

**Goal**: Understand why model performance may be degrading and decide whether to retrain.

**Steps**:
1. The Data Scientist receives a drift alert or opens the Monitoring dashboard.
2. The system shows prediction distribution drift (PSI), feature distribution drift for each feature, and rolling performance metrics (precision, recall over 7 and 30-day windows).
3. The Data Scientist identifies which features have drifted most.
4. The Data Scientist checks the alert rate trend for anomalies.
5. Based on the analysis, the Data Scientist decides to trigger a retraining run with updated data.

**Outcome**: The Data Scientist has a clear picture of model health and has initiated corrective action if needed.

### Fraud Analyst

#### Flow: Review Flagged Transactions

**Goal**: Examine transactions flagged by the model and decide whether they are fraudulent.

**Steps**:
1. The Fraud Analyst opens the Alert Queue.
2. The system displays flagged transactions sorted by priority (highest fraud score first), showing amount, merchant, cardholder, timestamp, and fraud score for each.
3. The Analyst selects the top-priority alert and assigns it to themselves.
4. The system opens the Case Review view showing:
   - Full transaction details (amount, merchant, category, location, channel)
   - Fraud score with feature importance breakdown (e.g., "velocity_1h contributed 0.35, amount_deviation contributed 0.28")
   - The cardholder's last 10 transactions for context
   - Any prior alerts for the same cardholder
5. The Analyst examines the evidence and makes a decision: Confirm Fraud, Dismiss, or Escalate.
6. The Analyst writes a justification in the notes field (at least 10 characters).
7. The Analyst submits the decision.
8. The system records the case, transitions the alert status, and updates the transaction status.

**Outcome**: The transaction is reviewed and the decision is recorded in the audit trail. The decision becomes a training label for future model improvement.

**Possible errors**:
- Notes are too short -- the system rejects the decision and asks for a more detailed justification.
- The alert was already assigned to another analyst -- the system shows a conflict message. The Analyst picks a different alert.

#### Flow: View Personal Dashboard

**Goal**: Check daily progress and performance metrics.

**Steps**:
1. The Fraud Analyst opens their dashboard.
2. The system shows: cases reviewed today, average review time, decision distribution (confirmed vs dismissed vs escalated), and comparison against team averages.
3. The Analyst identifies whether they are on pace for their daily target.

**Outcome**: The Analyst has visibility into their own performance and workload.

### Admin

#### Flow: Configure Scoring Threshold

**Goal**: Adjust the fraud score threshold that determines which transactions are flagged for review.

**Steps**:
1. The Admin opens the Scoring Configuration page.
2. The system shows the current threshold value, the current flag rate (percentage of transactions being flagged), and the analyst workload (cases per analyst per day).
3. The Admin adjusts the threshold (e.g., from 0.7 to 0.65 to catch more fraud, or to 0.75 to reduce analyst workload).
4. The system confirms the change and records it in the audit log.
5. The new threshold takes effect immediately for all subsequent scored transactions.

**Outcome**: The flagging behavior is adjusted. More transactions are flagged (lower threshold) or fewer (higher threshold).

**Possible errors**:
- Setting the threshold too low would overwhelm analysts -- the system shows a warning with the estimated impact on daily alert volume based on recent score distributions.

#### Flow: Manage Users

**Goal**: Add or modify user accounts and roles.

**Steps**:
1. The Admin opens the User Management section.
2. The Admin sees all users with their roles and active status.
3. To add a user, the Admin clicks "Create User," enters email, name, and selects a role (Data Engineer, Data Scientist, Fraud Analyst, or Admin).
4. To change a role or deactivate a user, the Admin selects the user and modifies accordingly.
5. All changes are recorded in the audit log.

**Outcome**: Users have the correct access levels.

#### Flow: Export Audit Trail

**Goal**: Generate a compliance report of all actions taken in the system.

**Steps**:
1. The Admin opens the Audit Trail section.
2. The Admin filters by date range, action type (model promotions, case decisions, threshold changes, user changes), or specific actors.
3. The Admin reviews the filtered entries.
4. The Admin exports the results for compliance documentation.

**Outcome**: A complete audit trail is available for regulatory review.

## Cross-Actor Flows

### Flow: End-to-End Fraud Detection (All Actors)

1. A payment processor sends a transaction event. The ingestion service validates, normalizes, and persists it.
2. The feature engineering service computes real-time features (velocity, amount deviation, distance).
3. The scoring service assembles the feature vector, runs inference, and produces a fraud score of 0.82.
4. The score exceeds the threshold (0.7). The transaction is flagged and an alert is created.
5. The **Fraud Analyst** sees the alert at the top of the queue, assigns it, reviews the score explanation and cardholder history, and confirms fraud.
6. The decision is recorded as a training label. The **Data Scientist** sees that sufficient new labels have accumulated.
7. The daily retraining job assembles a new dataset and trains a candidate model.
8. The **Data Scientist** reviews the evaluation, promotes to staging, and an A/B test begins.
9. After 7 days, the challenger outperforms. The **Data Scientist** promotes it to production.
10. The **Admin** reviews the promotion in the audit trail and monitors the updated flag rate.

### Flow: Model Drift Response (Data Scientist + Admin)

1. The monitoring service detects that the prediction score distribution has shifted (PSI > 0.2).
2. The **Data Scientist** receives a drift alert and opens the monitoring dashboard.
3. The **Data Scientist** identifies that the feature "merchant_risk_90d" has drifted significantly — new merchants with no history are entering the system.
4. The **Data Scientist** triggers a retraining run with updated data.
5. The new model shows improved metrics. The **Data Scientist** promotes it through the A/B test process.
6. The **Admin** adjusts the threshold slightly to account for the new model's scoring distribution.

### Flow: Analyst Feedback Improves Model (Fraud Analyst + Data Scientist)

1. Over several weeks, **Fraud Analysts** review hundreds of flagged transactions, confirming some as fraud and dismissing others.
2. Each decision becomes a labeled training example.
3. When sufficient labels accumulate, the daily retraining job triggers automatically.
4. The new model benefits from the analyst-provided ground truth, reducing false positives.
5. The **Data Scientist** monitors the improvement via A/B testing before promoting.
6. **Fraud Analysts** notice fewer false alarms in their queue over time.

## State Machines

### Transaction State Machine

- **Initial state**: Ingested
- **Transitions**:
  - From **Ingested** to **Scored** -- triggered when the scoring service computes a fraud score below the threshold
  - From **Ingested** to **Scored** then immediately to **Flagged** -- triggered when the fraud score exceeds the threshold (is_flagged = true)
  - From **Scored** to **Resolved** -- for non-flagged transactions, resolved automatically (no review needed)
  - From **Flagged** to **Reviewed** -- triggered when an analyst submits a case decision (confirm, dismiss, or escalate)
  - From **Reviewed** to **Resolved** -- triggered when the case has a final decision (confirmed_fraud or dismissed); escalated cases remain in Reviewed until a follow-up decision
- **Forbidden transitions**:
  - Ingested cannot go directly to Flagged -- must be scored first
  - Flagged cannot go back to Ingested or Scored -- it must move forward through the review process
  - Resolved is final -- no transitions out of Resolved

### Model Lifecycle State Machine

- **Initial state**: Training
- **Transitions**:
  - From **Training** to **Evaluating** -- triggered when model training completes and evaluation begins
  - From **Evaluating** to **Staging** -- triggered when a Data Scientist approves the model for A/B testing
  - From **Evaluating** to **Retired** -- triggered when a Data Scientist rejects the model
  - From **Staging** to **Production** -- triggered when a Data Scientist promotes the A/B test challenger after the 7-day minimum
  - From **Production** to **Retired** -- triggered when a new model is promoted to production, replacing this one
- **Forbidden transitions**:
  - Training cannot go directly to Staging or Production -- must be evaluated first
  - Evaluating cannot go directly to Production -- must go through Staging (A/B testing)
  - Retired is final -- no transitions out of Retired
- **Special rules**: Only one model can be in Production status at a time. Only one A/B test can be active at a time. Staging requires an active A/B test.

## Business Rules

1. Transactions scoring above the configured threshold are held for analyst review -- they are never auto-rejected or auto-blocked.
2. Every analyst decision must include written justification (minimum 10 characters in the notes field).
3. Analyst decisions (confirm fraud, dismiss) feed back into the training dataset as labeled examples for the next model version.
4. Escalated cases do not receive training labels until a final decision is made.
5. Model deployment to production requires explicit approval from at least one Data Scientist.
6. A/B testing must run for a minimum of 7 days before a challenger model can be promoted.
7. Only one A/B test can be active at a time.
8. The feature store maintains point-in-time correctness -- training data must never include features computed after the transaction's timestamp.
9. All predictions must include explainability data (feature importance per decision).
10. Model retraining runs daily on the previous day's labeled data combined with all historical labels.
11. Scoring latency must remain under 100ms at P99.
12. Transaction ingestion must sustain 10,000+ transactions per second at peak.
13. Every model promotion, threshold change, and analyst decision is recorded in the audit trail.
14. The scoring threshold is configurable at runtime by Admins without requiring redeployment.
15. Dead-letter messages preserve the original payload for debugging and can be reprocessed after the root cause is fixed.
