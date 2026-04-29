---
name: universal-framework
description: |
  Applies the INPUTS → PROCESOS → OUTPUTS → SOPORTE framework to ANY software type.
  Use as the FIRST step when decomposing any project, especially non-standard types (IoT, ML, blockchain, simulation, etc.).
  Do NOT use for projects already decomposed or when the user only needs documentation.
  Trigger with "apply universal framework", "decompose using INPUTS-PROCESOS-OUTPUTS", or "what are the dimensions of this project".
---

# Universal Decomposition Framework

You apply a universal four-dimension framework to decompose any software project, regardless of type.

## The Framework

```
INPUTS    → What enters the system?
PROCESOS  → What does the system do with the inputs?
OUTPUTS   → What does the system produce/expose?
SOPORTE   → What sustains the system in production?
```

## Applied to Different Project Types

### REST API
- INPUTS: HTTP requests (REST/GraphQL), request payloads (JSON/XML/form-data), authentication tokens (JWT/OAuth/API keys), webhook callbacks from external services, file uploads
- PROCESOS: request validation and sanitization, business logic execution, database queries and persistence, cache management (read-through/write-behind), event publishing to message queues, response serialization and formatting
- OUTPUTS: HTTP responses with status codes, JSON/XML response payloads, pagination metadata (cursors/offsets), error responses with structured error codes, webhook deliveries to subscribers, generated files (reports/exports)
- SOPORTE: authentication and authorization middleware, rate limiting and throttling, API versioning strategy, logging and request tracing (correlation IDs), health checks and readiness probes, deployment pipeline (CI/CD), API documentation (OpenAPI/Swagger), database migrations

### Data Pipeline / ETL
- INPUTS: source files (CSV/Parquet/JSON/XML), API responses from upstream services, database snapshots and CDC streams, message queue events (Kafka/RabbitMQ), scheduled triggers (cron/orchestrator), configuration and schema definitions
- PROCESOS: data extraction and ingestion, schema validation and type coercion, cleaning (dedup, null handling, normalization), transformation and enrichment, aggregation and windowing, anomaly and quality checks, partitioned loading into target stores
- OUTPUTS: data warehouse tables (fact/dimension), materialized views and data marts, reports and dashboards, alerts on quality violations, data lineage metadata, exported files for downstream consumers
- SOPORTE: orchestration and scheduling (Airflow/Dagster), retry logic and dead-letter queues, data lineage and catalog tracking, monitoring and SLA alerting, secret management for source credentials, backfill and replay tooling

### Mobile App
- INPUTS: user interactions (touch/gesture/voice), device sensors (GPS, camera, accelerometer), push notifications (APNs/FCM), deep links and universal links, local storage and cached data, API responses from backend services
- PROCESOS: state management (Redux/Bloc/MVI), offline-first sync and conflict resolution, media processing (image resize, video compression), navigation and routing logic, background task execution, permission handling and capability checks
- OUTPUTS: rendered screens and UI components, local database writes (SQLite/Realm), API requests to backend, analytics events and crash reports, files saved to device storage, share intents and inter-app communication
- SOPORTE: authentication and session management, push notification registration and handling, crash reporting and diagnostics (Sentry/Crashlytics), app store build and distribution pipeline, feature flags and remote config, accessibility compliance, certificate pinning and secure storage

### IoT (Arduino/ESP32)
- INPUTS: sensor readings (temperature, humidity, motion, voltage), user commands via physical controls or companion app, OTA configuration updates, MQTT subscriptions from cloud broker, BLE/WiFi provisioning data, interrupt signals from hardware peripherals
- PROCESOS: signal filtering and calibration (moving average, Kalman), threshold evaluation and rule engine, edge ML inference (TFLite Micro), actuator control logic (PID loops, state machines), local data buffering for intermittent connectivity, power mode management (active/sleep/deep-sleep)
- OUTPUTS: actuator commands (relays, motors, LEDs, displays), MQTT publishes to cloud topics, local display updates (OLED/LCD), BLE advertisements and notifications, serial debug output, local SD card logging
- SOPORTE: WiFi/BLE provisioning and reconnection, OTA firmware update mechanism, watchdog timer and crash recovery, power management and battery monitoring, secure boot and flash encryption, device registration and fleet management

### Blockchain (Smart Contracts)
- INPUTS: signed transactions from wallets, oracle data feeds (Chainlink/Band), governance proposals and votes, cross-chain bridge messages, constructor arguments and initialization parameters, off-chain metadata (IPFS/Arweave URIs)
- PROCESOS: input validation and access control (modifiers/guards), state transitions and storage updates, token minting/burning/transfer logic, event emission for indexers, delegate calls and proxy patterns, fee calculation and distribution
- OUTPUTS: on-chain state changes (storage slots), emitted events (indexed/non-indexed), return values to calling contracts, token balance updates, NFT metadata resolution, transaction receipts and logs
- SOPORTE: deployment scripts and verification (Hardhat/Foundry), proxy upgrade mechanism (UUPS/Transparent), contract monitoring and alerting, gas optimization and profiling, audit tooling and formal verification, testnet staging and migration strategy

### Machine Learning
- INPUTS: training datasets (tabular/image/text/audio), validation and test splits, hyperparameter configurations, pre-trained model weights (transfer learning), feature definitions and transformation pipelines, labeling and annotation metadata
- PROCESOS: data preprocessing and augmentation, feature engineering and selection, model training (distributed/single-node), evaluation against metrics (accuracy/F1/AUC/BLEU), hyperparameter tuning (grid/random/Bayesian), inference pipeline (batch/real-time), model explainability analysis (SHAP/LIME)
- OUTPUTS: trained model artifacts (weights/checkpoints), prediction results (scores/labels/embeddings), evaluation reports and metric dashboards, feature importance rankings, model cards and documentation, exported models (ONNX/TFLite/CoreML)
- SOPORTE: experiment tracking (MLflow/W&B), model registry and versioning, serving infrastructure (TF Serving/Triton/BentoML), data versioning (DVC), monitoring for drift and degradation, A/B testing and canary rollout, GPU/TPU resource management

### Simulation
- INPUTS: model parameters and constants, initial conditions and boundary values, scenario definitions and parameter sweeps, random seeds for reproducibility, external data for calibration, intervention schedules and triggers
- PROCESOS: model initialization and warm-up, time-step iteration (fixed/adaptive), agent interaction and state updates (agent-based), Monte Carlo sampling and replication, intervention application at scheduled points, convergence checking and early stopping, sensitivity analysis across parameter ranges
- OUTPUTS: time-series data and state trajectories, aggregate statistics and confidence intervals, visualizations (plots/animations/heatmaps), comparison reports across scenarios, exported datasets for further analysis, parameter sensitivity rankings
- SOPORTE: parameter management and configuration files, reproducibility guarantees (seed control, environment locking), checkpoint and restart mechanism, parallel execution orchestration, export and archival tooling, validation against empirical data

### Research Project
- INPUTS: research hypotheses and questions, raw datasets (experimental/observational/survey), methodology and protocol definitions, literature references and prior results, ethical approval and data use agreements, configuration for statistical tools and environments
- PROCESOS: data cleaning and exploratory analysis, experiment execution and data collection, statistical tests (t-test, ANOVA, regression, Bayesian), model fitting and cross-validation, result interpretation and effect size calculation, peer review iteration and revision, sensitivity and robustness checks
- OUTPUTS: statistical tables and result summaries, figures and visualizations (publication-quality), paper draft sections (intro/methods/results/discussion), supplementary materials and appendices, reproducible analysis notebooks, presentation materials (slides/posters)
- SOPORTE: computational environment setup (conda/Docker), version control for code and manuscripts, data management and backup strategy, reproducibility tooling (Makefile/Snakemake), collaboration platform (Overleaf/Git), archival and DOI registration

## Signals to Detect Project Type

When the user doesn't explicitly state the project type, look for these keywords:

| Signal Keywords | Likely Type |
|---|---|
| endpoint, route, controller, API, REST, GraphQL, CRUD | REST API / Web Service |
| extract, transform, load, pipeline, warehouse, batch, ingest | Data Pipeline / ETL |
| screen, navigation, offline, push notification, app store | Mobile App |
| sensor, actuator, MQTT, firmware, OTA, edge, device | IoT |
| contract, wallet, token, mint, stake, chain, gas | Blockchain |
| train, model, predict, feature, dataset, inference, epoch | Machine Learning |
| simulate, iterate, parameter sweep, agent-based, Monte Carlo | Simulation |
| hypothesis, experiment, p-value, dataset, methodology | Research Project |

If keywords from multiple types appear, the project is likely **multi-layer** — apply the framework to each layer independently.

## Output Format

Produce a structured analysis before passing to `swebok-decompose`:

```yaml
project:
  name: "..."
  type: "REST API | Mobile | ETL | IoT | Blockchain | ML | Simulation | Research"
  layers:
    - name: "..."  # only if multi-layer
      type: "..."

dimensions:
  inputs:
    - category: "..."
      items:
        - "specific input 1"
        - "specific input 2"
  processes:
    - category: "..."
      items:
        - "specific process 1"
        - "specific process 2"
  outputs:
    - category: "..."
      items:
        - "specific output 1"
        - "specific output 2"
  support:
    - category: "..."
      items:
        - "specific support element 1"
        - "specific support element 2"

mapping_to_swebok:
  design_candidates: ["items from inputs/outputs that need shared structure"]
  construction_candidates: ["items from processes — these become work nodes"]
  security_candidates: ["items from support related to auth/access"]
  operations_candidates: ["items from support related to deploy/config/monitoring"]

next_step: "Pass this analysis to swebok-decompose to generate the work breakdown"
```

## Multi-Layer Projects

Some projects have multiple layers (e.g., mobile + backend + IoT). Each layer is its own Workpack:

```
Layer 1: Backend API
  INPUTS → PROCESOS → OUTPUTS → SOPORTE

Layer 2: Mobile App
  INPUTS → PROCESOS → OUTPUTS → SOPORTE
  (consumes Layer 1's outputs as inputs)

Layer 3: IoT Device
  INPUTS → PROCESOS → OUTPUTS → SOPORTE
  (publishes to Layer 1)
```

Each layer is decomposed independently, then linked via the contracts (API endpoints, MQTT topics, etc.).

## Anti-Patterns

- Do not confuse INPUTS with PROCESOS ("ingest data" is a process, not an input)
- Do not forget OUTPUTS (some projects have implicit outputs like state changes)
- Do not mix layers (don't put mobile UI in a backend Workpack)
- Do not treat SOPORTE as optional (auth/deploy are mandatory for production)
