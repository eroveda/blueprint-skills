# Output: `/blueprint-skills:swebok-decompose`

This is the literal output produced by the `swebok-decompose` skill, taking the [universal framework analysis](01-universal-framework-output.md) as its input.

The skill produced **36 work nodes** distributed across 3 layers, with explicit dependencies, SWEBOK categorization, validation against expected operations, and a dependency graph.

---

```yaml
project:
  name: "Industrial Machine Monitoring System"
  type: "Multi-layer (IoT + REST API/Data Pipeline + Web Frontend)"

actors:
  - name: "ESP32 Device"
    operations: ["sample vibration", "publish telemetry", "receive config", "apply OTA update"]
  - name: "Maintenance Engineer"
    operations: ["view machine status", "inspect vibration charts", "acknowledge alerts", "annotate anomalies", "configure thresholds"]
  - name: "Admin"
    operations: ["manage machines", "manage sensors", "manage users", "push device config", "trigger OTA", "view fleet health"]

entities:
  - name: "Machine"
    fields: ["id", "name", "location", "status", "created_at"]
  - name: "Sensor"
    fields: ["id", "machine_id", "position", "type", "firmware_version", "last_seen"]
  - name: "VibrationReading"
    fields: ["timestamp", "machine_id", "sensor_id", "rms", "peak", "dominant_freq", "raw_fft"]
  - name: "DeviceHealth"
    fields: ["timestamp", "sensor_id", "uptime", "free_heap", "wifi_rssi", "battery_voltage"]
  - name: "Alert"
    fields: ["id", "machine_id", "sensor_id", "severity", "type", "status", "detected_at", "acknowledged_by", "notes"]
  - name: "User"
    fields: ["id", "email", "role", "name", "created_at"]

# ═══════════════════════════════════════════════════════
# LAYER 1: ESP32 Sensor Firmware
# ═══════════════════════════════════════════════════════
layer_1:
  name: "ESP32 Sensor Firmware"
  type: "IoT"

  nodes:
    - id: "L1-01"
      title: "Firmware Project Bootstrap (ESP-IDF/PlatformIO)"
      category: "OPERATIONS"
      purpose: "Initialize firmware project structure, toolchain, build system, flash configuration, and partition table"
      depends_on: []

    - id: "L1-02"
      title: "Vibration Sensor Driver and Sampling"
      category: "CONSTRUCTION"
      purpose: "Initialize accelerometer via SPI/I2C, configure sampling rate (1kHz), read raw vibration data into DMA buffer"
      depends_on: ["L1-01"]

    - id: "L1-03"
      title: "Signal Processing Pipeline"
      category: "CONSTRUCTION"
      purpose: "Apply bandpass filter, compute FFT for frequency-domain features, calculate RMS velocity and peak acceleration from raw samples"
      depends_on: ["L1-02"]

    - id: "L1-04"
      title: "MQTT Message Schema Definition"
      category: "DESIGN"
      purpose: "Define MQTT topic hierarchy (machines/{id}/vibration, machines/{id}/health, devices/{id}/config) and JSON payload schemas"
      depends_on: ["L1-01"]

    - id: "L1-05"
      title: "WiFi and MQTT Connectivity"
      category: "CONSTRUCTION"
      purpose: "WiFi STA connection with exponential backoff reconnection, MQTT client with TLS, subscribe to config/command topics, publish vibration and health data"
      depends_on: ["L1-04"]

    - id: "L1-06"
      title: "Data Buffering for Offline Operation"
      category: "CONSTRUCTION"
      purpose: "Ring buffer in PSRAM to store vibration summaries during connectivity loss, drain buffer on reconnect preserving chronological order"
      depends_on: ["L1-03", "L1-05"]

    - id: "L1-07"
      title: "Edge Anomaly Detection and Local Alerting"
      category: "CONSTRUCTION"
      purpose: "Compare RMS/peak against configurable thresholds, publish immediate alert messages, drive LED/buzzer for on-site indication"
      depends_on: ["L1-03", "L1-05"]

    - id: "L1-08"
      title: "Remote Configuration and OTA Updates"
      category: "CONSTRUCTION"
      purpose: "Receive config changes (sampling rate, thresholds, reporting interval) via MQTT, apply OTA firmware from HTTPS endpoint, store config in NVS"
      depends_on: ["L1-05"]

    - id: "L1-09"
      title: "Device Health Telemetry"
      category: "CONSTRUCTION"
      purpose: "Periodically publish uptime, free heap, WiFi RSSI, battery voltage, and firmware version to health topic"
      depends_on: ["L1-05"]

    - id: "L1-10"
      title: "Power Management"
      category: "CONSTRUCTION"
      purpose: "Implement active/light-sleep cycle between sampling bursts, manage wake sources (timer, GPIO interrupt), optimize current draw"
      depends_on: ["L1-02", "L1-05"]

    - id: "L1-11"
      title: "Secure Communication"
      category: "SECURITY"
      purpose: "TLS certificate provisioning and pinning for MQTT broker, secure boot and flash encryption, unique device identity via hardware ID"
      depends_on: ["L1-05"]

    - id: "L1-12"
      title: "Watchdog and Crash Recovery"
      category: "OPERATIONS"
      purpose: "Task watchdog timer, panic handler with core dump to flash, automatic reboot and state recovery from NVS"
      depends_on: ["L1-01"]

  validation:
    total_nodes: 12
    construction_count: 8
    construction_ratio: "8/12 (67%)"

# ═══════════════════════════════════════════════════════
# LAYER 2: Backend API & Ingestion
# ═══════════════════════════════════════════════════════
layer_2:
  name: "Backend API & Ingestion"
  type: "REST API + Data Pipeline"

  nodes:
    - id: "L2-01"
      title: "Backend Project Bootstrap"
      category: "OPERATIONS"
      purpose: "Initialize backend project (Python/Go/Node), dependency management, Docker Compose with MQTT broker, TSDB, and relational DB"
      depends_on: []

    - id: "L2-02"
      title: "Data Models: Machine, Sensor, Alert, User"
      category: "DESIGN"
      purpose: "Define relational schema for machines, sensors, alerts, users with migrations; define TSDB schema for vibration_readings and device_health measurements"
      depends_on: ["L2-01"]

    - id: "L2-03"
      title: "API Contract Definition"
      category: "DESIGN"
      purpose: "Define REST API endpoints, request/response schemas, WebSocket event format, and error code catalog (OpenAPI spec)"
      depends_on: ["L2-02"]

    - id: "L2-04"
      title: "Authentication and Role-Based Access Control"
      category: "SECURITY"
      purpose: "JWT-based login/refresh flow, role enforcement (admin/engineer/viewer), endpoint-level permission middleware"
      depends_on: ["L2-02"]

    - id: "L2-05"
      title: "MQTT Ingestion Service"
      category: "CONSTRUCTION"
      purpose: "Subscribe to vibration and health topics, validate message schema, write time-series data to TSDB with proper tags (machine_id, sensor_id)"
      depends_on: ["L2-02", "L1-04"]

    - id: "L2-06"
      title: "Anomaly Detection Engine"
      category: "CONSTRUCTION"
      purpose: "Analyze incoming vibration data against moving baseline (z-score), match known failure signatures (bearing wear, imbalance, misalignment), classify severity, apply deduplication and cooldown"
      depends_on: ["L2-05"]

    - id: "L2-07"
      title: "Alert Management Service"
      category: "CONSTRUCTION"
      purpose: "Create alerts from anomaly detections, manage lifecycle (open → acknowledged → resolved), store engineer annotations, dispatch notifications via WebSocket/email/SMS"
      depends_on: ["L2-02", "L2-04", "L2-06"]

    - id: "L2-08"
      title: "Time-Series Query API"
      category: "CONSTRUCTION"
      purpose: "REST endpoints for vibration data with time-range selection, downsampling (1s/1m/1h aggregates), FFT spectrum retrieval for selected windows"
      depends_on: ["L2-03", "L2-04", "L2-05"]

    - id: "L2-09"
      title: "Machine and Sensor Management API"
      category: "CONSTRUCTION"
      purpose: "CRUD endpoints for machines and sensors, device registration on first boot, decommissioning, firmware version tracking"
      depends_on: ["L2-03", "L2-04"]

    - id: "L2-10"
      title: "Device Configuration Push"
      category: "CONSTRUCTION"
      purpose: "API endpoint to update device config (sampling rate, thresholds), publish config payload to MQTT devices/{id}/config topic, track config version"
      depends_on: ["L2-03", "L2-04", "L1-04"]

    - id: "L2-11"
      title: "WebSocket Real-Time Events"
      category: "CONSTRUCTION"
      purpose: "WebSocket server pushing live vibration updates, new alert notifications, and machine status changes to connected dashboard clients"
      depends_on: ["L2-04", "L2-05", "L2-07"]

    - id: "L2-12"
      title: "User Management API"
      category: "CONSTRUCTION"
      purpose: "CRUD for user accounts, role assignment (admin/engineer/viewer), password reset flow"
      depends_on: ["L2-03", "L2-04"]

    - id: "L2-13"
      title: "TSDB Retention and Downsampling Policies"
      category: "OPERATIONS"
      purpose: "Configure continuous aggregation queries (1m, 1h, 1d rollups), retention policies (raw: 30d, 1m: 1y, 1h: forever), automated cleanup"
      depends_on: ["L2-05"]

    - id: "L2-14"
      title: "Deployment Pipeline"
      category: "OPERATIONS"
      purpose: "Dockerfiles, docker-compose for local dev, CI/CD pipeline, health checks, readiness probes, structured logging with correlation IDs"
      depends_on: ["L2-01"]

  validation:
    total_nodes: 14
    construction_count: 8
    construction_ratio: "8/14 (57%)"

# ═══════════════════════════════════════════════════════
# LAYER 3: Web Dashboard
# ═══════════════════════════════════════════════════════
layer_3:
  name: "Web Dashboard"
  type: "Web Frontend"

  nodes:
    - id: "L3-01"
      title: "Frontend Project Bootstrap"
      category: "OPERATIONS"
      purpose: "Initialize React/Vue project with Vite, configure router, chart library (ECharts/Recharts), WebSocket client, and component structure"
      depends_on: []

    - id: "L3-02"
      title: "UI Component Library and Layout"
      category: "DESIGN"
      purpose: "Define dashboard layout (sidebar nav, header, content area), shared components (status badges, severity indicators, data tables), responsive breakpoints for control room displays"
      depends_on: ["L3-01"]

    - id: "L3-03"
      title: "Authentication Flow"
      category: "SECURITY"
      purpose: "Login page, JWT storage, token refresh interceptor, route guards for role-based page access, logout and session expiry handling"
      depends_on: ["L3-01", "L2-04"]

    - id: "L3-04"
      title: "Machine Fleet Overview Page"
      category: "CONSTRUCTION"
      purpose: "Grid/table view of all machines with real-time status indicators (green/yellow/red), last reading timestamp, active alert count, search and filter by location/status"
      depends_on: ["L3-02", "L3-03", "L2-09"]

    - id: "L3-05"
      title: "Machine Detail View with Vibration Charts"
      category: "CONSTRUCTION"
      purpose: "Time-series line charts for RMS, peak, dominant frequency over selectable time range; FFT spectrum chart for selected time window; sensor position selector; real-time data overlay via WebSocket"
      depends_on: ["L3-02", "L3-03", "L2-08", "L2-11"]

    - id: "L3-06"
      title: "Alert Dashboard and Management"
      category: "CONSTRUCTION"
      purpose: "Alert list with severity filters (info/warning/critical), status tabs (open/acknowledged/resolved), acknowledge button, annotation input, anomaly markers overlaid on vibration charts"
      depends_on: ["L3-02", "L3-03", "L2-07"]

    - id: "L3-07"
      title: "Device Management Page"
      category: "CONSTRUCTION"
      purpose: "Table of registered sensors with firmware version, last seen, WiFi RSSI, battery; config editor (sampling rate, thresholds) with push-to-device action; OTA trigger button"
      depends_on: ["L3-02", "L3-03", "L2-09", "L2-10"]

    - id: "L3-08"
      title: "User Administration Page"
      category: "CONSTRUCTION"
      purpose: "User list with role badges, create/edit/deactivate users, role assignment dropdown (admin-only page)"
      depends_on: ["L3-02", "L3-03", "L2-12"]

    - id: "L3-09"
      title: "Real-Time WebSocket Integration"
      category: "CONSTRUCTION"
      purpose: "WebSocket client with auto-reconnect, merge live events into chart state and fleet overview, browser notifications for critical alerts"
      depends_on: ["L3-01", "L2-11"]

    - id: "L3-10"
      title: "Frontend Build and Deployment"
      category: "OPERATIONS"
      purpose: "Production build config, static hosting setup (S3/CDN/Nginx), environment variable injection, CI pipeline for lint/build/deploy"
      depends_on: ["L3-01"]

  validation:
    total_nodes: 10
    construction_count: 6
    construction_ratio: "6/10 (60%)"

# ═══════════════════════════════════════════════════════
# CROSS-PROJECT VALIDATION
# ═══════════════════════════════════════════════════════
validation:
  total_nodes: 36
  construction_count: 22
  construction_ratio: "22/36 (61%)"

  expected_operations:
    - "ESP32 → Vibration → sample and process"
    - "ESP32 → Telemetry → publish health"
    - "ESP32 → Config → receive and apply"
    - "ESP32 → Firmware → OTA update"
    - "ESP32 → Anomaly → edge detect and alert"
    - "Engineer → Machine status → view fleet"
    - "Engineer → Vibration charts → inspect detail"
    - "Engineer → Alerts → view, acknowledge, annotate"
    - "Engineer → Thresholds → configure"
    - "Admin → Machines → CRUD"
    - "Admin → Sensors → manage, push config, OTA"
    - "Admin → Users → CRUD and role assignment"
  coverage: "12/12 operations covered"

  cross_layer_contracts:
    - contract: "MQTT vibration payload schema"
      producer: "L1-04, L1-03"
      consumer: "L2-05"
    - contract: "MQTT config topic schema"
      producer: "L2-10"
      consumer: "L1-08"
    - contract: "REST API endpoints"
      producer: "L2-03, L2-08, L2-09, L2-07, L2-12"
      consumer: "L3-04, L3-05, L3-06, L3-07, L3-08"
    - contract: "WebSocket event format"
      producer: "L2-11"
      consumer: "L3-09, L3-05, L3-06"
```

---

## Dependency Graph (Simplified)

The skill rendered this dependency map showing how the three layers connect:

```
LAYER 1 (ESP32)          LAYER 2 (Backend)           LAYER 3 (Dashboard)
───────────────          ─────────────────           ───────────────────
L1-01 Bootstrap          L2-01 Bootstrap              L3-01 Bootstrap
  │                        │                            │
  ├─ L1-02 Sensor Drv      ├─ L2-02 Data Models        ├─ L3-02 UI Components
  │   │                    │   │                        │
  │   └─ L1-03 DSP ──┐     ├─ L2-03 API Contract       ├─ L3-03 Auth ◄──── L2-04
  │                   │    │   │                        │
  ├─ L1-04 MQTT Sch ──┼────┼─ L2-04 Auth/RBAC          ├─ L3-04 Fleet ◄─── L2-09
  │   │               │    │   │                        │
  │   └─ L1-05 MQTT ──┤    ├─ L2-05 Ingestion ◄────────┤  L3-05 Detail ◄── L2-08
  │       │           │    │   │                        │
  │       ├─ L1-06 Buf│    │   └─ L2-06 Anomaly Det    ├─ L3-06 Alerts ◄── L2-07
  │       ├─ L1-07 Edg│    │       │                    │
  │       ├─ L1-08 Cfg◄────┼── L2-07 Alert Mgmt        ├─ L3-07 Devices ◄─ L2-10
  │       ├─ L1-09 Hlth│   │                            │
  │       └─ L1-10 Pwr │   ├─ L2-08 TS Query API       ├─ L3-08 Users ◄── L2-12
  │                    │   ├─ L2-09 Machine API         │
  ├─ L1-11 TLS         │   ├─ L2-10 Cfg Push ──────────┤  L3-09 WebSocket ◄ L2-11
  └─ L1-12 Watchdog    │   ├─ L2-11 WebSocket ─────────┘
                       │   ├─ L2-12 User API
                       │   ├─ L2-13 TSDB Retention
                       │   └─ L2-14 Deploy Pipeline
```

---

## What the Skill Did

**36 nodes total, 22 CONSTRUCTION (61%).** Each layer independently exceeds the 40% construction threshold required by the skill.

The skill enforced:

1. **Bootstrap mandatory in every layer**: L1-01, L2-01, L3-01 are all OPERATIONS nodes for project setup
2. **CONSTRUCTION dominance**: 22 out of 36 nodes (61%) are CONSTRUCTION — well above the 40% threshold
3. **No invented features**: every node traces to inputs/processes/outputs identified in the previous step
4. **Coverage validation**: 12 expected actor-entity-operation pairs were extracted from the brief; all 12 are covered by at least one CONSTRUCTION node
5. **Cross-layer contracts**: 4 contracts identified and mapped to producer/consumer specs

---

**Next step**: The skill explicitly suggested running `/blueprint-skills:swebok-generate-spec` to convert each node into an executable specification. See [`03-SPECS.yaml`](03-SPECS.yaml).
