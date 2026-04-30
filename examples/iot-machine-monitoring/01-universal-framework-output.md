# Output: `/blueprint-skills:universal-framework`

This is the literal output produced by the `universal-framework` skill when given the [original input](00-input.md).

The skill identified the project as **multi-layer (IoT + Backend + Frontend)** and applied the INPUTS → PROCESOS → OUTPUTS → SOPORTE framework to each layer independently, then mapped each dimension to candidate SWEBOK categories.

---

```yaml
project:
  name: "Industrial Machine Monitoring System"
  type: "Multi-layer (IoT + REST API + Web Dashboard)"
  layers:
    - name: "ESP32 Sensor Firmware"
      type: "IoT"
    - name: "Backend API & Ingestion"
      type: "REST API + Data Pipeline"
    - name: "Web Dashboard"
      type: "Web Frontend"

dimensions:
  # ═══════════════════════════════════════════════
  # LAYER 1: ESP32 Sensor Firmware
  # ═══════════════════════════════════════════════
  inputs:
    - category: "Sensor Data (Layer 1)"
      items:
        - "Accelerometer/vibration sensor readings (ADXL345, MPU6050, or industrial-grade ICP)"
        - "Machine ID and sensor position from NVS/EEPROM configuration"
        - "Sampling rate and threshold parameters via MQTT config topic"
        - "NTP time sync for accurate timestamps"
        - "WiFi/network credentials from provisioning (BLE or hardcoded)"

    - category: "Backend Commands (Layer 1)"
      items:
        - "OTA firmware update payloads"
        - "Remote configuration changes (sampling rate, reporting interval)"
        - "Heartbeat/ping requests from backend"

  processes:
    - category: "Signal Processing (Layer 1)"
      items:
        - "Vibration sampling at fixed interval (e.g., 1kHz)"
        - "Signal filtering (moving average, bandpass) to remove noise"
        - "FFT computation for frequency-domain features (dominant frequency, harmonics)"
        - "RMS velocity and peak acceleration calculation"
        - "Local threshold check for immediate alerts (edge detection)"
        - "Data buffering during connectivity loss (ring buffer in PSRAM)"

    - category: "Communication (Layer 1)"
      items:
        - "MQTT publish of vibration summaries to broker topic (e.g., machines/{id}/vibration)"
        - "MQTT subscribe to config and command topics"
        - "WiFi reconnection with exponential backoff"
        - "Power mode management (active sampling vs. light sleep between bursts)"

  outputs:
    - category: "MQTT Messages (Layer 1)"
      items:
        - "Vibration data payload: {machine_id, sensor_pos, rms, peak, dominant_freq, timestamp}"
        - "Device health telemetry: {uptime, free_heap, wifi_rssi, battery_voltage}"
        - "Local anomaly alerts: {machine_id, alert_type, severity, raw_values}"
        - "OTA status acknowledgments"

    - category: "Local (Layer 1)"
      items:
        - "LED/buzzer indication for connectivity and alert status"
        - "Serial debug output for field diagnostics"

  support:
    - category: "Device Lifecycle (Layer 1)"
      items:
        - "WiFi/BLE provisioning flow for factory setup"
        - "OTA firmware update mechanism (HTTPS or MQTT-based)"
        - "Watchdog timer and crash recovery (automatic reboot)"
        - "NVS storage for persistent configuration across reboots"
        - "Secure MQTT connection (TLS with certificate pinning)"
        - "Device registration: each ESP32 registers with unique machine_id on first boot"

  # ═══════════════════════════════════════════════
  # LAYER 2: Backend API & Ingestion
  # ═══════════════════════════════════════════════
  inputs:
    - category: "MQTT Ingestion (Layer 2)"
      items:
        - "Vibration data messages from ESP32 fleet (consumes Layer 1 outputs)"
        - "Device health telemetry messages"
        - "Local anomaly alert messages from edge devices"

    - category: "HTTP Requests (Layer 2)"
      items:
        - "Dashboard API requests (machine list, time-series queries, anomaly history)"
        - "Engineer authentication requests (login/token refresh)"
        - "Device management requests (register, decommission, update config)"
        - "Alert acknowledgment and annotation requests"

  processes:
    - category: "Data Ingestion (Layer 2)"
      items:
        - "MQTT subscriber consuming vibration topic with fan-out"
        - "Message validation and schema enforcement"
        - "Time-series write to TSDB (InfluxDB/TimescaleDB/QuestDB)"
        - "Device health metric aggregation"

    - category: "Anomaly Detection (Layer 2)"
      items:
        - "Statistical anomaly detection on incoming vibration data (z-score, moving baseline)"
        - "Pattern matching against known failure signatures (bearing wear, imbalance, misalignment)"
        - "Severity classification (info/warning/critical)"
        - "Alert deduplication and cooldown logic"
        - "Alert dispatch (WebSocket push to dashboard, optional email/SMS)"

    - category: "API Business Logic (Layer 2)"
      items:
        - "Time-series query with downsampling for dashboard charts"
        - "Machine and sensor CRUD operations"
        - "Alert lifecycle management (open → acknowledged → resolved)"
        - "User authentication and role-based access (admin vs. viewer)"

  outputs:
    - category: "API Responses (Layer 2)"
      items:
        - "Time-series data (JSON arrays with timestamps and values)"
        - "Machine registry with current status (healthy/warning/critical)"
        - "Anomaly/alert list with severity, timestamp, and machine context"
        - "Device configuration payloads pushed back via MQTT"

    - category: "Real-time (Layer 2)"
      items:
        - "WebSocket events for live dashboard updates"
        - "Alert notifications (email/SMS/webhook for critical anomalies)"

  support:
    - category: "Infrastructure (Layer 2)"
      items:
        - "MQTT broker deployment and clustering (Mosquitto/EMQX/HiveMQ)"
        - "Time-series database provisioning and retention policies"
        - "API deployment pipeline (Docker/K8s)"
        - "Database migrations for relational data (machines, users, alerts)"
        - "Logging with correlation IDs (request → MQTT message → alert)"
        - "Health checks and readiness probes"
        - "Backup strategy for TSDB and relational data"
        - "JWT-based authentication middleware"

  # ═══════════════════════════════════════════════
  # LAYER 3: Web Dashboard
  # ═══════════════════════════════════════════════
  inputs:
    - category: "API Data (Layer 3)"
      items:
        - "REST API responses from backend (consumes Layer 2 outputs)"
        - "WebSocket events for real-time vibration and alert updates"
        - "Engineer login credentials"

    - category: "User Interactions (Layer 3)"
      items:
        - "Machine selection and time-range navigation"
        - "Alert acknowledgment and annotation by engineers"
        - "Threshold configuration for anomaly sensitivity"
        - "Dashboard layout preferences"

  processes:
    - category: "Visualization (Layer 3)"
      items:
        - "Time-series chart rendering (vibration RMS, peak, frequency over time)"
        - "FFT spectrum visualization for selected time windows"
        - "Machine fleet overview with status indicators (green/yellow/red)"
        - "Alert timeline and history display"
        - "Real-time data merging (WebSocket events into chart state)"

    - category: "State Management (Layer 3)"
      items:
        - "Authentication state and token refresh"
        - "Selected machine/time-range/view state"
        - "WebSocket connection management and reconnection"
        - "Client-side caching for recently viewed data"

  outputs:
    - category: "Rendered UI (Layer 3)"
      items:
        - "Machine overview dashboard with health status grid"
        - "Detail view with vibration charts and anomaly markers"
        - "Alert panel with severity indicators and action buttons"
        - "Device management table (firmware version, last seen, config)"

    - category: "User Actions (Layer 3)"
      items:
        - "API requests triggered by user interactions (acknowledge alert, update config)"
        - "Browser notifications for critical alerts"

  support:
    - category: "Frontend Infrastructure (Layer 3)"
      items:
        - "Build pipeline and static hosting (Vite + CDN/S3)"
        - "Responsive layout for control room displays and tablets"
        - "Accessibility compliance for industrial environment"
        - "Error boundary and offline fallback"

mapping_to_swebok:
  design_candidates:
    - "MQTT topic structure and message schemas (contract between Layer 1 ↔ Layer 2)"
    - "REST API contract (contract between Layer 2 ↔ Layer 3)"
    - "WebSocket event format"
    - "Time-series data model (measurements, tags, retention)"
    - "Machine/sensor/alert relational data model"
    - "Anomaly detection algorithm selection"

  construction_candidates:
    - "ESP32 firmware: sampling, FFT, MQTT publish, OTA"
    - "MQTT ingestion service: subscribe, validate, write to TSDB"
    - "Anomaly detection engine: statistical analysis, alert dispatch"
    - "REST API: machine CRUD, time-series queries, alert management"
    - "Dashboard: charts, fleet overview, alert panel, device management"

  security_candidates:
    - "TLS for MQTT connections (device ↔ broker)"
    - "JWT authentication for API"
    - "Role-based access control (admin/engineer/viewer)"
    - "Device certificate management"
    - "Secure OTA update verification"

  operations_candidates:
    - "MQTT broker deployment and monitoring"
    - "TSDB retention policies and capacity planning"
    - "Device fleet management and provisioning"
    - "Alerting pipeline monitoring (ensuring alerts are not dropped)"
    - "Dashboard deployment pipeline"

next_step: "Pass this analysis to swebok-decompose to generate the work breakdown"
```

---

## Layer Dependency Map

The skill also produced this visualization of the contracts between layers:

```
┌─────────────────┐     MQTT      ┌─────────────────────┐    REST/WS    ┌──────────────┐
│  ESP32 Firmware │─────────────▶│  Backend API &       │◀────────────▶│  Web          │
│  (Layer 1)      │              │  Ingestion (Layer 2) │              │  Dashboard    │
│                 │              │                      │              │  (Layer 3)    │
│  Vibration data │◀─────────────│  TSDB + Anomaly Det. │              │  Charts/Alerts│
└─────────────────┘  Config/OTA  └─────────────────────┘              └──────────────┘
```

**Key contracts between layers:**
- L1 ↔ L2: MQTT topics (`machines/{id}/vibration`, `machines/{id}/health`, `devices/{id}/config`)
- L2 ↔ L3: REST endpoints + WebSocket channel for real-time updates

---

## What the Skill Did

1. Recognized the input describes **three architectural layers**, not one application
2. Applied INPUTS → PROCESOS → OUTPUTS → SOPORTE to each layer independently
3. Identified **cross-layer contracts** (MQTT, REST, WebSocket) as candidates for DESIGN nodes
4. Pre-categorized everything into SWEBOK buckets to set up the next skill (`swebok-decompose`)

The user's 3-sentence input had no mention of:
- TLS, JWT, RBAC → skill inferred them as security candidates
- Continuous aggregation, retention policies → skill inferred for time-series ops
- WebSocket vs polling → skill chose WebSocket for real-time
- Edge processing vs cloud-only → skill identified the trade-off (will become ADR-002 in architecture doc)

These inferences come from the universal framework + IoT pattern knowledge embedded in the skill, not from the user's prompt.

---

**Next step**: The skill explicitly suggested running `/blueprint-skills:swebok-decompose` next. See [`02-decomposition-output.md`](02-decomposition-output.md).
