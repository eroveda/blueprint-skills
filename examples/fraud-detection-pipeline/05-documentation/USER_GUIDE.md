# User Guide — Fraud Detection Pipeline

## Welcome

The Fraud Detection Pipeline helps your organization catch fraudulent transactions before they cause harm. The system ingests transactions in real time, scores them with machine learning, and routes suspicious activity to your fraud analysts for review. Over time, analyst decisions make the model smarter.

This guide covers everything you need to know, whether you are a Fraud Analyst reviewing cases, a Data Scientist managing models, a Data Engineer keeping the pipelines healthy, or an Admin running the show.

## Getting Started

### First Login

1. Your Admin will create your account and assign you a role.
2. Log in with your email and password.
3. You will land on a dashboard tailored to your role:
   - **Fraud Analysts** see the Alert Queue -- flagged transactions waiting for review.
   - **Data Scientists** see Model Health -- current production model status, drift indicators, and pending A/B tests.
   - **Data Engineers** see Pipeline Health -- ingestion throughput, consumer lag, and dead-letter queue status.
   - **Admins** see the System Overview -- user activity, flag rate trends, and operational status.
4. You are ready to start working.

## Daily Usage

### For Fraud Analysts

This section covers your primary workflow: reviewing flagged transactions and making fraud decisions.

#### Your Alert Queue

1. Open the Alert Queue from the main menu.
2. Flagged transactions appear sorted by priority -- the highest fraud scores are at the top.
3. Each alert shows a summary: transaction amount, merchant, cardholder ID, timestamp, and fraud score.
4. Click "Take Case" to assign the top alert to yourself. This locks it so no one else reviews the same transaction.

#### Reviewing a Case

When you take a case, the Case Review screen opens with everything you need:

- **Transaction Details**: The full transaction -- amount, merchant name and category, cardholder, location, channel (card-present, online, etc.), and timestamp.
- **Score Explanation**: The fraud score (e.g., 0.82 out of 1.0) and a breakdown of which factors contributed most. For example: "Unusual velocity -- 5 transactions in the last hour (normally 1)" or "Amount is 4.2 standard deviations above cardholder average."
- **Cardholder History**: The cardholder's last 10 transactions, so you can see whether this transaction fits their normal pattern.
- **Prior Alerts**: Any previous alerts for the same cardholder, with their outcomes.

#### Making a Decision

After reviewing the evidence, choose one of three options:

- **Confirm Fraud**: You believe this transaction is fraudulent.
- **Dismiss**: You believe this transaction is legitimate.
- **Escalate**: You are unsure and want a second opinion from a colleague or supervisor.

You must write a justification in the notes field (at least 10 characters). Good notes help your team learn and improve over time. Examples:
- "Confirmed: card-not-present transaction from unusual country, velocity spike, cardholder confirmed unauthorized."
- "Dismissed: cardholder is a frequent traveler, amount is within normal range for business purchases."
- "Escalated: pattern is unusual but not conclusive, need senior review."

Your decision is recorded permanently in the audit trail. It also becomes a labeled example that helps train future models -- so your expertise directly improves the system.

#### Your Dashboard

Open your personal dashboard to see:
- Cases reviewed today and this week
- Average review time
- Decision breakdown (how many confirmed, dismissed, escalated)
- Comparison against team averages

### For Data Scientists

This section covers model training, evaluation, and promotion.

#### Checking Model Health

1. Open the Monitoring dashboard.
2. You will see the current production model's health indicators:
   - **Prediction Drift**: Has the distribution of fraud scores changed? A high PSI (Population Stability Index) value means scores are shifting.
   - **Feature Drift**: Are any input features behaving differently than they did during training?
   - **Performance**: Rolling precision and recall over the last 7 and 30 days, computed from analyst-labeled outcomes.
   - **Alert Rate**: Is the flagging rate stable, spiking, or dropping?
   - **Model Age**: How old is the training data? Stale models may not reflect current fraud patterns.

#### Training a New Model

1. Go to Model Training and click "New Training Run."
2. Select which features to include (you can use the current feature set or add new ones).
3. Configure hyperparameters or use the recommended defaults.
4. Set the training data date range. The system will assemble the dataset from the feature store using point-in-time joins -- so you do not need to worry about data leakage.
5. Submit the training job. Training runs in the background.
6. When training finishes, you will see the evaluation report:
   - Precision, recall, F1, AUC-ROC, and false positive rate
   - Threshold analysis curves showing precision-recall trade-offs at different threshold values
   - Side-by-side comparison with the current production model

#### Promoting a Model

The promotion process has built-in safeguards to prevent deploying a bad model:

1. Review the evaluation metrics. If you are satisfied, click "Promote to Staging."
2. The system creates an A/B test automatically: 90% of traffic continues using the current model (champion), 10% goes to your new model (challenger).
3. The A/B test runs for at least 7 days. You cannot promote early -- the system enforces this minimum.
4. Check the A/B test dashboard daily to see how the challenger compares: precision, recall, false positive rate, and alert rate.
5. After 7 days, if the challenger performs well, click "Promote to Production."
6. The system swaps models, retires the old one, and records everything in the audit log.

If the challenger performs worse, click "Cancel Test." The champion continues serving all traffic.

#### Understanding Drift Alerts

When you receive a drift alert, it means something has changed:
- **Score drift**: The model is producing a different distribution of scores than expected. This could mean fraud patterns have shifted.
- **Feature drift**: An input feature is behaving differently. For example, a new merchant category with no history could cause the "merchant_risk" feature to behave unexpectedly.
- **Performance drop**: Precision or recall has fallen below the expected range based on recent analyst labels.

In any case, the recommended action is to retrain the model with recent data to capture the new patterns.

### For Data Engineers

This section covers pipeline operations and monitoring.

#### Monitoring Ingestion

1. Open the Ingestion Health dashboard.
2. Key metrics to watch:
   - **Throughput**: Transactions per second being consumed and processed. Normal range depends on your volume.
   - **Consumer Lag**: How far behind the consumers are. Should be under 30 seconds.
   - **Validation Failure Rate**: Percentage of messages failing schema validation. A sudden increase indicates a source change.
   - **Dead-Letter Queue Depth**: Messages that could not be processed. Zero is ideal.

#### Handling Dead-Letter Messages

1. Open the Dead-Letter Queue view.
2. Each message shows the original payload, the error that caused the failure, and the timestamp.
3. Common issues:
   - **Unknown currency code**: A source is sending a new currency not in the allowed list. Update the validation rules, then reprocess.
   - **Missing required field**: A source changed its format. Coordinate with the source provider to fix, or update the normalization logic.
   - **Timestamp parse error**: The source uses a different timestamp format. Add the format to the parser.
4. After fixing the root cause, reprocess the affected messages.

#### Managing Feature Definitions

1. Open the Feature Definitions page.
2. To add a new feature, click "New Definition" and specify the name, description, computation type (real-time or batch), and parameters.
3. To update a feature, edit the definition. The system creates a new version automatically -- old versions remain available for backward compatibility.
4. Monitor the next computation cycle to verify the feature is producing expected values.

### For Admins

#### Managing Users

1. Go to User Management.
2. To add a user, click "Create User" and enter their email, name, and role.
3. Available roles: Data Engineer, Data Scientist, Fraud Analyst, Admin.
4. To deactivate a user, click their entry and toggle the active status.
5. To change a role, click their entry and select a new role.

#### Adjusting the Scoring Threshold

The scoring threshold determines how many transactions get flagged for analyst review:

1. Go to Scoring Configuration.
2. You will see the current threshold, the current flag rate, and the estimated analyst workload.
3. **Lowering the threshold** (e.g., 0.7 to 0.6) catches more fraud but increases analyst workload.
4. **Raising the threshold** (e.g., 0.7 to 0.8) reduces workload but may miss some fraudulent transactions.
5. The system shows an estimated impact before you confirm the change.
6. Changes take effect immediately and are recorded in the audit log.

#### Reviewing the Audit Trail

1. Go to Audit Trail.
2. Filter by date range, action type, or actor.
3. Available action types: model promotions, threshold changes, case decisions, user management changes.
4. Export the filtered results for compliance reporting.

## Understanding the System

### How Fraud Scoring Works

1. **A transaction arrives** from a payment processor via streaming.
2. **Features are computed** in real time: How many transactions has this cardholder made in the last hour? How unusual is this amount? How far is this location from their last transaction?
3. **The model scores the transaction** using these features and produces a fraud score between 0.0 (very likely legitimate) and 1.0 (very likely fraud).
4. **If the score exceeds the threshold**, the transaction is flagged and an alert is created for analyst review. The transaction is held -- not blocked.
5. **An analyst reviews the alert**, examines the score explanation, and makes a decision.
6. **The decision becomes a training label** that improves future models.

### What the Fraud Score Means

- **0.0 - 0.3**: Low risk. The transaction matches the cardholder's normal patterns.
- **0.3 - 0.7**: Medium risk. Some features are unusual but not conclusive.
- **0.7 - 1.0**: High risk. Multiple features are abnormal.

The threshold (configurable by Admin) determines the cutoff. Only transactions above the threshold are flagged. Transactions below the threshold are resolved automatically.

### Feature Importance Explained

Every scored transaction comes with a feature importance breakdown. This tells you which factors contributed most to the fraud score. For example:

- "velocity_1h: 0.35" -- The number of transactions in the last hour was the biggest contributor.
- "amount_deviation: 0.28" -- The amount was unusually high compared to this cardholder's history.
- "geo_distance: 0.15" -- The location was far from the cardholder's last transaction.

These numbers help analysts understand why the model flagged a transaction, making reviews faster and more accurate.

### Permissions

- **Admins** can: manage users, configure thresholds, view audit trails, access all dashboards, and perform any action.
- **Data Scientists** can: train models, manage A/B tests, approve promotions, view monitoring dashboards, and manage feature definitions.
- **Data Engineers** can: monitor pipelines, manage ingestion configuration, handle dead-letter queues, and manage feature definitions.
- **Fraud Analysts** can: review alerts, make case decisions, and view their dashboards.
- **Data Engineers** cannot: review fraud cases or manage models.
- **Fraud Analysts** cannot: manage pipelines, train models, or configure thresholds.

## Common Scenarios

### Scenario: Investigating a High-Score Alert

You are a Fraud Analyst reviewing an alert with a score of 0.91:
1. Open the Case Review. The score explanation shows: velocity spike (8 transactions in 1 hour, normal is 2), unusual merchant category, and card-not-present transaction from a country the cardholder has never transacted in.
2. Check the cardholder's recent history -- the last 5 transactions were in a different country.
3. Note the amount is 3x the cardholder's average.
4. Confirm fraud with notes explaining the evidence.

### Scenario: Responding to a Model Drift Alert

You are a Data Scientist who received a drift alert:
1. Open the Monitoring dashboard. PSI for score distribution is 0.31 (above the 0.2 threshold).
2. Check feature drift -- "merchant_risk_90d" has PSI of 0.45. Many new merchants have entered the system.
3. Check rolling performance -- precision has dropped from 0.85 to 0.72 over the last 7 days.
4. Trigger a retraining run with the latest labeled data.
5. After training, evaluate the new model. It shows improved precision (0.83) on recent data.
6. Promote to staging and start a 7-day A/B test.

### Scenario: Adjusting Threshold After Model Change

You are an Admin noticing analyst workload has increased after a model promotion:
1. Open Scoring Configuration. The flag rate increased from 2% to 4% after the new model went live.
2. The analyst team is reviewing 200 cases per day instead of the normal 100.
3. Check with the Data Scientist team -- the new model has better recall (catches more fraud) but also a higher false positive rate.
4. Raise the threshold from 0.7 to 0.75 to bring the flag rate closer to the desired level.
5. Monitor for a few days to confirm the adjustment.

## Troubleshooting

### "My alert queue is empty but I expected flagged transactions"

The scoring threshold may be set too high, or the model may not be running. Check:
- Is the scoring service healthy? (Ask your Data Engineer to check the health endpoint.)
- What is the current threshold? (Ask your Admin to check the Scoring Configuration.)
- Are transactions being ingested? (Check the Ingestion Health dashboard.)

### "The score explanation does not make sense for this transaction"

Feature values may be stale. If a cardholder has not transacted in a long time, some features (like velocity) may be outdated. This is expected -- the model works with the best available information. If you notice a pattern of confusing explanations, let the Data Scientist team know so they can investigate the feature definitions.

### "I cannot promote my model -- the system says an A/B test is already active"

Only one A/B test can run at a time. Either wait for the current test to complete (check the A/B test dashboard for the remaining days) or cancel the current test if it is no longer needed.

### "Model training failed"

Common causes:
- The feature store had missing values for some features. Check feature freshness.
- The training dataset had too few labeled examples. Wait for more analyst decisions.
- A resource limit was exceeded (memory, disk). Contact your Admin or Data Engineer.

### "Consumer lag is increasing and will not come down"

The ingestion service may be overwhelmed. Check:
- Is throughput higher than usual? (A traffic spike may require scaling consumers.)
- Is the database responding slowly? (Check database health.)
- Is the feature store saturated? (Real-time feature computation depends on feature store writes.)

## FAQs

### Does the system automatically block fraudulent transactions?

No. Transactions scoring above the threshold are held for analyst review. The system never auto-blocks or auto-rejects. This is intentional -- model predictions are not perfect, and human review prevents legitimate transactions from being blocked.

### How does my review help the model improve?

When you confirm fraud or dismiss a case, your decision becomes a labeled training example. The next model retraining (daily) includes your labeled data. Over time, the model learns from thousands of your decisions and becomes more accurate.

### How long does a model promotion take?

At minimum, 7 days for the A/B test, plus time for training and evaluation. In practice, expect about 10 days from training start to production promotion.

### What happens if the model goes down?

If the scoring service becomes unavailable, transactions continue to be ingested but are not scored. When the service recovers, unscored transactions are processed in order. No transactions are lost.

### Why do some features show as "stale"?

Batch features are computed on a schedule (default nightly). If the batch job has not run or encountered an error, features may be staler than expected. The system alerts Data Engineers when feature freshness exceeds the configured threshold.

### Can two analysts review the same transaction?

No. When you assign an alert to yourself, it is locked to you. Other analysts see it as "assigned" and move on to the next alert. This prevents duplicate work.

## Getting Help

- **For pipeline issues**: Contact your Data Engineering team.
- **For model questions**: Contact your Data Science team.
- **For account or access issues**: Contact your Admin.
- **For system outages**: Follow your organization's incident response procedures.
