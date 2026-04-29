---
name: universal-framework
description: |
  Applies the INPUTS → PROCESOS → OUTPUTS → SOPORTE framework to ANY software type.
  Use when starting decomposition of unusual project types (IoT, ML, blockchain, simulation, etc.).
  Trigger with "apply universal framework" or "decompose using INPUTS-PROCESOS-OUTPUTS".
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
- INPUTS: HTTP requests, payloads, JWT tokens
- PROCESOS: business logic, validation, persistence
- OUTPUTS: HTTP responses, JSON payloads
- SOPORTE: auth, logging, monitoring, deployment

### Data Pipeline / ETL
- INPUTS: source files, APIs, database snapshots
- PROCESOS: cleaning, transformation, aggregation, anomaly detection
- OUTPUTS: data warehouse tables, reports, alerts
- SOPORTE: scheduler, retry logic, lineage tracking

### Mobile App
- INPUTS: user interactions, sensors, push notifications
- PROCESOS: state management, offline sync, calculations
- OUTPUTS: screens, local storage, server sync
- SOPORTE: auth, push registration, crash reporting

### IoT (Arduino/ESP32)
- INPUTS: sensor readings, user commands, OTA configurations
- PROCESOS: signal processing, rule evaluation, edge ML
- OUTPUTS: actuator commands, MQTT publishes, local display
- SOPORTE: WiFi config, deep sleep, OTA updates

### Blockchain (Smart Contracts)
- INPUTS: transactions, oracle data, governance votes
- PROCESOS: validation, state transitions, event emission
- OUTPUTS: blockchain state changes, events, return values
- SOPORTE: deployment, verification, monitoring, upgradability

### Machine Learning
- INPUTS: training datasets, hyperparameters, validation data
- PROCESOS: training, evaluation, inference, fine-tuning
- OUTPUTS: trained model, predictions, metrics
- SOPORTE: experiment tracking, model versioning, serving infra

### Simulation
- INPUTS: parameters, initial conditions, scenarios
- PROCESOS: model execution, iteration, intervention application
- OUTPUTS: visualizations, metrics, comparison reports
- SOPORTE: parameter management, reproducibility, export tools

### Research Project
- INPUTS: hypotheses, datasets, methodology
- PROCESOS: experiments, analysis, statistical tests
- OUTPUTS: tables, figures, paper draft, conclusions
- SOPORTE: environment setup, version control, reproducibility

## How to Apply

1. Read the project description carefully
2. List every INPUT mentioned (or implied)
3. List every PROCESS the system performs
4. List every OUTPUT the system produces
5. List every SUPPORT element (auth, deploy, monitoring)
6. Each PROCESS becomes a CONSTRUCTION node
7. INPUTS and OUTPUTS that need shared structure become DESIGN nodes
8. SUPPORT becomes SECURITY/OPERATIONS nodes
9. Generate the work breakdown using `swebok-decompose`

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

- ❌ Confusing INPUTS with PROCESOS ("ingest data" is a process, not an input)
- ❌ Forgetting OUTPUTS (some projects have implicit outputs like state changes)
- ❌ Mixing layers (don't put mobile UI in a backend Workpack)
- ❌ Treating SOPORTE as optional (auth/deploy are mandatory for production)
