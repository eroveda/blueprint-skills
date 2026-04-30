# Architecture — Industrial Machine Monitoring System

## Overview

An IoT-based system for monitoring industrial machine health through vibration analysis. ESP32 sensors attached to machines sample accelerometer data, perform on-device signal processing (FFT, RMS), and publish results via MQTT. A backend service ingests the time-series data, runs statistical anomaly detection, and manages the alert lifecycle. Maintenance engineers interact with the system through a web dashboard displaying real-time vibration charts, fleet status, and actionable alerts.

**Primary Actors:**

| Actor | Role | Key Actions |
|---|---|---|
| ESP32 Device | Autonomous sensor node | Sample vibration, publish telemetry, receive config, apply OTA |
| Maintenance Engineer | Monitors machines, responds to anomalies | View charts, acknowledge/resolve alerts, annotate findings, configure thresholds |
| Admin | Manages the system | Register machines/sensors, manage users, push device config, trigger OTA |

**Scope:** Three-layer system — firmware (edge), backend (API + ingestion + anomaly detection), and frontend (web dashboard).

## Tech Stack

### Layer 1: ESP32 Sensor Firmware

| Component | Technology | Version | Purpose |
|---|---|---|---|
| Runtime | FreeRTOS (via ESP-IDF) | ESP-IDF 5.x | Real-time task scheduling, dual-core execution |
| Framework | ESP-IDF | 5.x | Peripheral drivers, networking, OTA, NVS |
| DSP | esp-dsp | latest | Optimized FFT (dsps_fft2r_fc32) |
| Sensor | ADXL345 (SPI) / MPU6050 (I2C) | — | 3-axis accelerometer, ±8g/±16g |
| Connectivity | esp_mqtt + esp_wifi | (bundled) | MQTT over TLS, WiFi STA |
| Storage | NVS Flash + SPIFFS | (bundled) | Config persistence, TLS certificates |
| JSON | cJSON | (bundled) | Message serialization |

### Layer 2: Backend API & Ingestion

| Component | Technology | Version | Purpose |
|---|---|---|---|
| Language/Runtime | Python 3.11+ / Go 1.21+ / Node 20+ | — | Backend application (choose one) |
| Framework | FastAPI / Gin / NestJS | latest | REST API, WebSocket, middleware |
| Time-Series DB | TimescaleDB | ≥ 2.11 | Vibration and health data storage, continuous aggregates |
| Relational DB | PostgreSQL | 15+ | Machines, sensors, users, alerts |
| Message Broker | Mosquitto / EMQX | latest | MQTT broker for device communication |
| Cache | Redis | 7+ | Optional: query caching, rate limiting |
| Auth | JWT (bcrypt) | — | Stateless authentication |
| Migrations | Alembic / golang-migrate / Knex | — | Schema versioning |
| Containerization | Docker + Docker Compose | — | Local dev and production deployment |

### Layer 3: Web Dashboard

| Component | Technology | Version | Purpose |
|---|---|---|---|
| Framework | React | 18+ | Component-based UI |
| Language | TypeScript | 5+ | Type safety |
| Build | Vite | 5+ | Dev server and production bundler |
| Styling | TailwindCSS | 3+ | Utility-first CSS |
| Charts | ECharts (echarts-for-react) | 5+ | Time-series and FFT visualization |
| Server State | TanStack React Query | 5+ | API data fetching and caching |
| Client State | Zustand | 4+ | Auth state, WebSocket events |
| HTTP | Axios | 1+ | API client with interceptors |
| Routing | React Router | 6+ | SPA routing |
| Hosting | Nginx (Alpine) | — | Static file serving with gzip |

## Component Architecture

### Layer 1 Modules (ESP32 Firmware)

#### Module: sensor_driver
- **Responsibility**: Initialize and read from accelerometer hardware at configurable sample rates
- **Public API**: `sensor_init(config)`, `sensor_read_sample(sample_t*)`, `sensor_start_continuous(callback)`, `sensor_stop()`
- **Dependencies**: ESP-IDF SPI/I2C drivers
- **Data**: Produces `sample_t {int16_t x, y, z; int64_t timestamp_us}` into a lock-free ring buffer (4096 samples, PSRAM)
- **Constraints**: Runs on Core 1, priority 20; jitter ≤ ±50µs at 1kHz

#### Module: signal_processing
- **Responsibility**: Transform raw vibration samples into frequency-domain features via FFT
- **Public API**: `dsp_init(config)`, `dsp_process_window(samples, count, result_t*)`
- **Dependencies**: sensor_driver (ring buffer consumer), esp-dsp (FFT)
- **Data**: Produces `vibration_result_t {rms_accel_g, peak_accel_g, rms_velocity_mm_s, dominant_freq_hz, fft_magnitudes[512], window_start_us, window_end_us}` into a FreeRTOS queue (depth 8)
- **Constraints**: 1024-point FFT, 50% window overlap, Hanning window, processing < 500ms/window

#### Module: mqtt_client (mqtt_manager)
- **Responsibility**: Manage WiFi and MQTT connectivity with auto-reconnect and message queuing
- **Public API**: `mqtt_manager_init(config)`, `mqtt_manager_publish(topic, payload, qos)`, `mqtt_manager_subscribe(topic, qos, callback)`, `mqtt_manager_is_connected()`
- **Dependencies**: mqtt_schema (topic strings, JSON serializers), esp_mqtt, esp_wifi
- **Data**: Publishes JSON to MQTT topics; receives config/OTA messages via callbacks
- **Constraints**: TLS required, 60s keep-alive, 32-message publish queue, exponential backoff reconnect

#### Module: mqtt_schema
- **Responsibility**: Define MQTT topic hierarchy and JSON serialization/deserialization for all message types
- **Public API**: Topic format strings (`TOPIC_VIBRATION`, `TOPIC_HEALTH`, etc.), `mqtt_json_serialize_vibration(result, buf)`, `mqtt_json_parse_config(json, config)`
- **Dependencies**: cJSON
- **Data**: Cross-layer contract defining the interface between firmware and backend

#### Module: data_buffer
- **Responsibility**: Buffer vibration results during connectivity loss, drain on reconnect
- **Public API**: `buffer_init(capacity)`, `buffer_push(result)`, `buffer_pop(result)`, `buffer_count()`, `buffer_is_full()`
- **Dependencies**: signal_processing (result queue), mqtt_client (connectivity status)
- **Data**: Persistent ring buffer in PSRAM, 1000 entries (~200KB)
- **Constraints**: Drain rate ≤ 10 msg/s, FIFO ordering, original timestamps preserved

#### Module: edge_detector
- **Responsibility**: Compare vibration results against thresholds, trigger local and remote alerts
- **Public API**: `edge_detector_init(thresholds)`, `edge_detector_evaluate(result, alert)`, `edge_detector_update_thresholds(thresholds)`
- **Dependencies**: signal_processing (results), mqtt_client (alert publishing)
- **Data**: Publishes alert JSON to `machines/{id}/alerts`; drives LED/buzzer GPIO
- **Constraints**: 60s cooldown per alert type, < 1ms evaluation time, runtime-updatable thresholds

#### Module: config_manager
- **Responsibility**: Receive and apply remote configuration, manage OTA firmware updates
- **Public API**: `config_init()`, `config_get(key, value)`, `config_set(key, value)`, `config_on_update(callback)`
- **Dependencies**: mqtt_client (config/OTA topic subscription), NVS (persistence)
- **Data**: Stores config in NVS, tracks config_version
- **Constraints**: Hot-reload (no reboot), monotonic version, SHA256-verified OTA, rollback on failed boot

#### Module: health_monitor
- **Responsibility**: Periodically publish device health telemetry
- **Public API**: `health_monitor_init(interval_s)`, `health_monitor_start()`, `health_monitor_stop()`
- **Dependencies**: mqtt_client, data_buffer (dropped_count)
- **Data**: Publishes health JSON every 60s (configurable 10–300s)

#### Module: power_manager
- **Responsibility**: Manage active/sleep duty cycle for battery-powered deployments
- **Public API**: `power_init(config)`, `power_enter_light_sleep(duration_ms)`, `power_get_wake_reason()`
- **Dependencies**: sensor_driver (start/stop), mqtt_client (drain queue before sleep)
- **Constraints**: Light sleep only (PSRAM retained), < 5ms wake latency, disableable for mains-powered

### Layer 2 Modules (Backend)

#### Module: ingestion
- **Responsibility**: Subscribe to MQTT topics, validate messages, batch-write to TimescaleDB
- **Public API**: Internal service; consumes MQTT, writes to DB, forwards alerts to anomaly engine
- **Dependencies**: MQTT broker, TimescaleDB (vibration_readings, device_health), sensors table
- **Database**: Writes to `vibration_readings`, `device_health` hypertables; updates `sensors.last_seen_at`
- **Constraints**: ≥ 100 msg/s sustained, batch inserts (100 rows or 1s window), never crash on invalid input

#### Module: anomaly
- **Responsibility**: Detect vibration anomalies using statistical and pattern-matching methods
- **Public API**: Internal; receives vibration data from ingestion, creates alerts, updates machine status
- **Dependencies**: ingestion (data feed), vibration_readings (historical baseline), alerts/machines tables
- **Database**: Reads `vibration_readings` (24h baseline), writes to `alerts`, updates `machines.status`
- **Algorithms**: Z-score against moving baseline (24h), bearing wear/imbalance/misalignment pattern matchers
- **Constraints**: < 100ms per reading, 5-minute dedup window, min 100 readings for statistical baseline

#### Module: alert_service
- **Responsibility**: Manage alert lifecycle, dispatch notifications
- **Public API**: REST endpoints: `GET /api/alerts`, `GET /api/alerts/{id}`, `POST /api/alerts/{id}/acknowledge`, `POST /api/alerts/{id}/resolve`, `PUT /api/alerts/{id}/notes`
- **Dependencies**: auth middleware, anomaly engine (creates alerts), WebSocket (dispatches events)
- **Database**: `alerts` table (CRUD + lifecycle transitions)
- **Constraints**: Status flow: open → acknowledged → resolved (no skip/revert)

#### Module: timeseries_api
- **Responsibility**: Efficient time-series query with automatic downsampling
- **Public API**: `GET /api/machines/{id}/vibration`, `GET /api/machines/{id}/vibration/fft`
- **Dependencies**: auth middleware, vibration_readings + continuous aggregates
- **Database**: Reads from `vibration_readings`, `vibration_readings_1m`, `vibration_readings_1h`, `vibration_readings_1d`
- **Constraints**: Max 5000 data points per response, 30-day max range, auto interval selection

#### Module: machine_api
- **Responsibility**: CRUD for machines and sensors, device auto-registration
- **Public API**: `GET/POST/PUT/DELETE /api/machines`, `GET/POST /api/machines/{id}/sensors`, `PUT/DELETE /api/sensors/{id}`
- **Dependencies**: auth middleware
- **Database**: `machines`, `sensors` tables
- **Constraints**: Soft delete, no cascade, auto-registration on unknown sensor_id

#### Module: device_config
- **Responsibility**: Push configuration and OTA commands to devices via MQTT
- **Public API**: `POST /api/sensors/{id}/config`, `GET /api/sensors/{id}/config`, `POST /api/sensors/{id}/ota`
- **Dependencies**: auth middleware, MQTT client, mqtt_schema
- **Database**: `device_configs` table (config history)
- **Constraints**: QoS 1 + retain for config, monotonic config_version, OTA admin-only

#### Module: websocket
- **Responsibility**: Push real-time events to connected dashboard clients
- **Public API**: WebSocket at `/ws`, client subscription model
- **Dependencies**: auth (JWT verification), ingestion (vibration events), alert_service (alert events)
- **Constraints**: Max 100 concurrent connections, 1/s vibration rate limit per machine per client, 30s heartbeat

#### Module: user_api
- **Responsibility**: User account and role management
- **Public API**: `GET/POST/PUT/DELETE /api/users`, `PUT /api/users/me/password`
- **Dependencies**: auth middleware
- **Database**: `users` table
- **Constraints**: Admin-only (except password change), cannot modify self, soft deactivation

#### Module: auth
- **Responsibility**: JWT authentication and role-based authorization middleware
- **Public API**: `POST /api/auth/login`, `POST /api/auth/refresh`; middleware: `authenticate()`, `require_role(min_role)`
- **Dependencies**: users table (credential verification)
- **Constraints**: bcrypt cost 12, access token 1h TTL, refresh token 7d TTL, stateless JWT

### Layer 3 Modules (Web Dashboard)

#### Module: api (src/api/)
- **Responsibility**: Axios client with JWT interceptors, API endpoint functions
- **Public API**: `client.ts` (axios instance), typed API functions per resource
- **Dependencies**: auth store (token), backend API

#### Module: stores (src/stores/)
- **Responsibility**: Client-side state management
- **Public API**: `authStore` (login/logout/token), `realtimeStore` (WebSocket events)
- **Dependencies**: Zustand

#### Module: hooks (src/hooks/)
- **Responsibility**: Custom React hooks for WebSocket and real-time data
- **Public API**: `useWebSocket()`, `useRealtimeVibration(machineId)`, `useRealtimeAlerts()`
- **Dependencies**: WebSocket server, React Query, auth store

#### Module: pages (src/pages/)
- **Responsibility**: Route-level page components
- **Pages**: LoginPage, DashboardPage (fleet overview), MachineDetailPage, AlertsPage, DevicesPage, UsersPage
- **Dependencies**: UI components, API hooks, auth guards

#### Module: components (src/components/)
- **Responsibility**: Reusable UI components
- **Components**: AppShell, StatusBadge, SeverityIcon, DataTable, TimeRangeSelector, VibrationChart, FFTChart, Card, ConfirmDialog, ProtectedRoute, RoleGuard

## Data Model

### Relational Entities (PostgreSQL)

```
┌──────────────┐       ┌──────────────┐       ┌──────────────────┐
│   machines   │       │   sensors    │       │  device_configs  │
├──────────────┤       ├──────────────┤       ├──────────────────┤
│ id (UUID PK) │◄──┐   │ id (UUID PK) │◄──┐   │ id (UUID PK)     │
│ name         │   │   │ machine_id FK│───┘   │ sensor_id FK     │───┐
│ location     │   │   │ position     │       │ config_version   │   │
│ status       │   │   │ type         │       │ config_json      │   │
│ created_at   │   │   │ firmware_ver │       │ pushed_at        │   │
│ updated_at   │   │   │ last_seen_at │       │ pushed_by FK     │   │
│ deleted_at   │   │   │ created_at   │       └──────────────────┘   │
└──────────────┘   │   │ deleted_at   │                              │
                   │   └──────────────┘                              │
                   │                                                 │
┌──────────────┐   │   ┌───────────────────┐                         │
│    users     │   │   │     alerts        │                         │
├──────────────┤   │   ├───────────────────┤                         │
│ id (UUID PK) │◄──┼───│ acknowledged_by FK│                         │
│ email        │   │   │ id (UUID PK)      │                         │
│ password_hash│   └───│ machine_id FK     │                         │
│ name         │       │ sensor_id FK      │─────────────────────────┘
│ role         │       │ severity          │
│ created_at   │       │ alert_type        │
│ updated_at   │       │ status            │
│ deactivated  │       │ value             │
└──────────────┘       │ threshold         │
                       │ detected_at       │
                       │ acknowledged_at   │
                       │ resolved_at       │
                       │ notes             │
                       └───────────────────┘
```

**Enumerations:**
- `machines.status`: `healthy`, `warning`, `critical`
- `users.role`: `admin`, `engineer`, `viewer`
- `alerts.severity`: `info`, `warning`, `critical`
- `alerts.status`: `open`, `acknowledged`, `resolved`

**Indexes:**
- `machines(status)` — fleet overview filter
- `sensors(machine_id)` — sensors per machine lookup
- `alerts(machine_id, status)` — active alerts per machine
- `alerts(detected_at)` — alert timeline
- `users(email)` — unique, login lookup

### Time-Series Entities (TimescaleDB Hypertables)

#### vibration_readings (chunk_interval: 1 day)
| Column | Type | Description |
|---|---|---|
| time | TIMESTAMPTZ NOT NULL | Measurement timestamp |
| machine_id | UUID | Machine reference |
| sensor_id | UUID | Sensor reference |
| rms_accel_g | FLOAT | RMS acceleration (g) |
| peak_accel_g | FLOAT | Peak acceleration (g) |
| rms_velocity_mm_s | FLOAT | RMS velocity (mm/s) |
| dominant_freq_hz | FLOAT | Dominant vibration frequency (Hz) |

**Index:** `(machine_id, time DESC)`

#### device_health (chunk_interval: 1 day)
| Column | Type | Description |
|---|---|---|
| time | TIMESTAMPTZ NOT NULL | Measurement timestamp |
| sensor_id | UUID | Sensor reference |
| uptime_s | INTEGER | Device uptime (seconds) |
| free_heap_bytes | INTEGER | Free heap memory |
| wifi_rssi_dbm | SMALLINT | WiFi signal strength |
| battery_voltage_mv | SMALLINT | Battery voltage (0 if mains) |
| firmware_version | VARCHAR(20) | Current firmware version |

**Index:** `(sensor_id, time DESC)`

### Continuous Aggregates

| Aggregate | Source | Bucket | Retention | Refresh |
|---|---|---|---|---|
| vibration_readings_1m | vibration_readings | 1 minute | 1 year | Every 1 min (lag 5 min) |
| vibration_readings_1h | vibration_readings_1m | 1 hour | 5 years | Every 1 hour (lag 1 hour) |
| vibration_readings_1d | vibration_readings_1h | 1 day | Forever | Every 1 day (lag 2 hours) |

**Retention Policies:**
- Raw vibration_readings: 30 days
- Raw device_health: 90 days

### Schema Migrations (Ordered)

1. Create `machines` table with status enum
2. Create `sensors` table with FK to machines
3. Create `users` table with role enum
4. Create `alerts` table with FKs to machines, sensors, users
5. Create `vibration_readings` hypertable (chunk_interval 1 day)
6. Create `device_health` hypertable (chunk_interval 1 day)
7. Create `device_configs` table with FK to sensors and users
8. Create indexes on all tables
9. Create continuous aggregates (1m, 1h, 1d)
10. Add retention policies and refresh policies

## API Contracts

### MQTT Topics (Device ↔ Backend)

| Topic | Direction | QoS | Retain | Payload |
|---|---|---|---|---|
| `machines/{machine_id}/vibration` | Device → Backend | 0 | No | Vibration summary JSON |
| `machines/{machine_id}/health` | Device → Backend | 1 | No | Device health JSON |
| `machines/{machine_id}/alerts` | Device → Backend | 1 | No | Edge-detected alert JSON |
| `devices/{sensor_id}/config` | Backend → Device | 1 | Yes | Configuration JSON |
| `devices/{sensor_id}/ota` | Backend → Device | 1 | No | OTA trigger JSON |

**Vibration Payload:**
```json
{
  "machine_id": "uuid",
  "sensor_id": "uuid",
  "timestamp": "2026-01-15T14:30:00.000Z",
  "rms_accel_g": 0.45,
  "peak_accel_g": 1.23,
  "rms_velocity_mm_s": 2.8,
  "dominant_freq_hz": 120.5,
  "fft_magnitudes": [0.01, 0.03, ...],
  "firmware_version": "1.2.0"
}
```

**Config Payload:**
```json
{
  "sampling_rate_hz": 1000,
  "reporting_interval_s": 60,
  "threshold_rms_warning": 2.0,
  "threshold_rms_critical": 5.0,
  "threshold_peak_warning": 4.0,
  "threshold_peak_critical": 10.0,
  "include_fft": false,
  "config_version": 42
}
```

### REST API Endpoints (Dashboard ↔ Backend)

All endpoints prefixed with `/api/`. Authentication via `Authorization: Bearer {JWT}` header.

**Standard Pagination:** `?page=1&per_page=20` → Response includes `{data: [...], pagination: {page, per_page, total, total_pages}}`

**Standard Error Response:** `{error: {code: "VALIDATION_ERROR", message: "...", details: {...}}}`

#### Authentication

| Method | Path | Auth | Request | Response |
|---|---|---|---|---|
| POST | /auth/login | None | `{email, password}` | `{access_token, refresh_token, expires_in}` |
| POST | /auth/refresh | None | `{refresh_token}` | `{access_token, expires_in}` |

#### Machines

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | /machines | viewer+ | List machines (filter: status, location; includes latest_reading, alert_count) |
| GET | /machines/{id} | viewer+ | Machine detail with sensors and latest readings |
| POST | /machines | admin | Create machine `{name, location}` → 201 |
| PUT | /machines/{id} | admin | Update machine `{name, location}` |
| DELETE | /machines/{id} | admin | Soft delete (rejects if active sensors) |

#### Sensors

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | /machines/{id}/sensors | viewer+ | List sensors for a machine |
| POST | /machines/{id}/sensors | admin | Register sensor `{id, position, type}` → 201 |
| PUT | /sensors/{id} | admin | Update sensor metadata |
| DELETE | /sensors/{id} | admin | Soft delete sensor |
| POST | /sensors/{id}/config | engineer+ | Push config to device via MQTT |
| GET | /sensors/{id}/config | viewer+ | Current config and version |
| POST | /sensors/{id}/ota | admin | Trigger OTA `{firmware_url, firmware_version, sha256}` |
| GET | /sensors/{id}/health | viewer+ | Latest health metrics |

#### Time-Series

| Method | Path | Auth | Query Params | Description |
|---|---|---|---|---|
| GET | /machines/{id}/vibration | viewer+ | start, end, interval (1s/1m/1h/1d), sensor_id | Vibration data with auto-downsampling |
| GET | /machines/{id}/vibration/fft | viewer+ | time, sensor_id | FFT spectrum at a specific timestamp |

**Auto-interval selection:** range < 1h → 1s, < 24h → 1m, < 7d → 1h, ≤ 30d → 1d. Max 5000 data points per response.

#### Alerts

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | /alerts | viewer+ | List alerts (filter: machine_id, severity, status, date range) |
| GET | /alerts/{id} | viewer+ | Alert detail with machine/sensor context |
| POST | /alerts/{id}/acknowledge | engineer+ | Set status=acknowledged |
| POST | /alerts/{id}/resolve | engineer+ | Set status=resolved (must be acknowledged first) |
| PUT | /alerts/{id}/notes | engineer+ | Update annotation notes |
| GET | /machines/{id}/alerts/summary | viewer+ | `{open, warning, critical}` counts |

#### Users

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | /users | admin | List users (paginated) |
| POST | /users | admin | Create user `{email, name, password, role}` → 201 |
| PUT | /users/{id} | admin | Update name, role (cannot change self) |
| DELETE | /users/{id} | admin | Soft deactivate (cannot deactivate self) |
| PUT | /users/me/password | any | Change own password `{current_password, new_password}` |

#### System

| Method | Path | Auth | Description |
|---|---|---|---|
| GET | /health | None | `{status: "ok", timestamp: "..."}` |
| GET | /ready | None | Checks DB + MQTT connectivity |

### WebSocket Events (Backend → Dashboard)

**Connection:** `ws://{host}/ws?token={JWT}`

**Client Subscription (sent after connect):**
```json
{"subscribe": ["fleet", "alerts", "machine:uuid-123"]}
```

**Server Events:**

| Event Type | Payload | Trigger |
|---|---|---|
| `vibration` | `{machine_id, data: {rms_accel_g, peak_accel_g, dominant_freq_hz, timestamp}}` | New vibration reading ingested |
| `alert` | `{id, machine_id, severity, alert_type, status, detected_at}` | Alert created or status changed |
| `machine_status` | `{machine_id, status}` | Machine status changed |
| `device_health` | `{sensor_id, data: {uptime_s, wifi_rssi_dbm, battery_voltage_mv}}` | Health telemetry received |

**Rate Limits:** Vibration events limited to 1/second/machine/client. Heartbeat ping/pong every 30s.

## Cross-Cutting Concerns

### Authentication & Authorization

**Mechanism:** Stateless JWT with bcrypt password hashing.

**Flow:**
1. Client sends `POST /api/auth/login` with email + password
2. Server verifies password against bcrypt hash (cost factor 12)
3. Server returns access token (1h TTL) and refresh token (7d TTL)
4. Client includes `Authorization: Bearer {access_token}` on all requests
5. On 401: client calls `POST /api/auth/refresh` with refresh token
6. On refresh failure: redirect to login

**JWT Claims:**
```json
{
  "sub": "user-uuid",
  "email": "engineer@example.com",
  "role": "engineer",
  "iat": 1706000000,
  "exp": 1706003600
}
```

**Role Hierarchy:** `viewer` < `engineer` < `admin` (higher includes all lower permissions)

| Action | Minimum Role |
|---|---|
| View machines, charts, alerts | viewer |
| Acknowledge/resolve alerts | engineer |
| Push device config | engineer |
| Configure thresholds | engineer |
| Create/delete machines, sensors | admin |
| Manage users | admin |
| Trigger OTA updates | admin |

### Device Security

- **Transport:** MQTT over TLS with CA certificate pinning
- **Identity:** Unique device UUID derived from ESP32 MAC address or efuse
- **Boot:** Secure boot and flash encryption documented (optional, irreversible efuse burn)
- **OTA:** SHA256 verification of firmware before flashing, rollback on failed boot within 60s
- **Certificates:** Stored in SPIFFS, rotatable via config without firmware update

### Logging & Observability

**Backend Structured Logging (JSON):**
```json
{
  "timestamp": "2026-01-15T14:30:00.000Z",
  "level": "info",
  "message": "Vibration data ingested",
  "correlation_id": "req-uuid",
  "service": "ingestion",
  "machine_id": "...",
  "batch_size": 50
}
```

- Correlation ID generated per HTTP request, propagated through all log entries
- Response header `X-Correlation-ID` returned to clients
- Request logging middleware: method, path, status_code, duration_ms

**Firmware Logging:**
- Serial output with component tags: `[SENSOR]`, `[DSP]`, `[MQTT]`, `[ALERT]`
- Core dumps stored in flash partition for post-mortem analysis
- Boot counter and crash reason tracked in NVS

**Ingestion Metrics:**
- messages_received, messages_valid, messages_invalid, inserts_per_second
- Exposed via health/ready endpoints or metrics endpoint

### Offline Resilience

- **ESP32:** Ring buffer in PSRAM (1000 entries) stores vibration results during connectivity loss. Drains at ≤ 10 msg/s on reconnect, preserving original timestamps. Dropped count reported in health telemetry.
- **Dashboard:** React Query caches recent data. WebSocket auto-reconnects with exponential backoff. Missing events recovered on next query refetch.

## Non-Functional Requirements

| NFR | Target | Validation |
|---|---|---|
| Ingestion throughput | ≥ 100 MQTT msg/s sustained | Load test with 50+ simulated devices |
| API response time (P95) | < 500ms for time-series queries | Load test with concurrent dashboard users |
| Time-series query max points | 5000 per response | Enforced in API, tested |
| WebSocket connections | 100 concurrent | Stress test |
| Firmware sampling jitter | ≤ ±50µs at 1kHz | Oscilloscope measurement |
| DSP processing time | < 500ms per 1024-sample window | Benchmark on ESP32 |
| Edge alert latency | < 1ms per evaluation | Microbenchmark |
| Offline buffer capacity | 1000 readings (~17 min at 1Hz) | Unit test |
| OTA firmware size | < 1.5MB | Build size check |
| Dashboard bundle size | < 2MB gzipped | Build output check |
| TSDB raw retention | 30 days | Retention policy |
| Anomaly detection latency | < 100ms per reading | Profiling |
| Device boot to first publish | < 10 seconds (WiFi connected) | Integration test |

## Architectural Decisions (ADRs)

### ADR-001: TimescaleDB over InfluxDB for Time-Series Storage
- **Status**: Accepted
- **Context**: Need a time-series database for high-throughput vibration data ingestion with SQL query capability. Options: InfluxDB (purpose-built TSDB), TimescaleDB (PostgreSQL extension), QuestDB.
- **Decision**: TimescaleDB — leverages PostgreSQL ecosystem (same DB for relational and time-series), supports continuous aggregates for automatic downsampling, standard SQL for complex queries including JOINs with relational tables.
- **Consequences**: Requires PostgreSQL expertise. Slightly lower raw ingestion throughput than InfluxDB, but sufficient for our scale (100 msg/s). Single database simplifies operations and removes cross-database JOIN complexity.

### ADR-002: Edge Processing on ESP32 Rather Than Cloud-Only
- **Status**: Accepted
- **Context**: Vibration analysis (FFT, RMS) could run entirely on the backend or be split between edge and cloud. Raw sample streaming at 1kHz × 3-axis × 50 devices would generate ~900KB/s sustained MQTT traffic.
- **Decision**: Perform FFT and feature extraction on the ESP32 (edge), transmit only summaries (~200 bytes/message at ~1Hz). Backend performs statistical anomaly detection on the summaries.
- **Consequences**: Dramatically reduces bandwidth (900KB/s → ~10KB/s). Enables offline alerting (edge threshold detection). Increases firmware complexity. Raw waveforms not available for backend analysis (acceptable trade-off — FFT magnitudes transmitted optionally).

### ADR-003: MQTT over HTTP for Device-to-Backend Communication
- **Status**: Accepted
- **Context**: Devices need to publish telemetry and receive configuration. Options: MQTT, HTTP polling, WebSocket, CoAP.
- **Decision**: MQTT with QoS levels — purpose-built for IoT telemetry, supports pub/sub for bidirectional communication (config push), low overhead, excellent ESP-IDF support, retained messages for config delivery on reconnect.
- **Consequences**: Requires MQTT broker infrastructure (Mosquitto/EMQX). Topic-based routing requires careful schema design. QoS 0 acceptable for vibration (high frequency, loss tolerable), QoS 1 required for alerts and config (at-least-once).

### ADR-004: Stateless JWT over Session-Based Auth
- **Status**: Accepted
- **Context**: Dashboard needs authentication for API and WebSocket. Internal tool with ~50 concurrent users.
- **Decision**: Stateless JWT with short-lived access tokens (1h) and longer refresh tokens (7d). No server-side session storage.
- **Consequences**: Simpler horizontal scaling (no shared session store). Token revocation requires waiting for expiry (acceptable for internal tool). Tokens stored in localStorage (acceptable risk for internal, non-public application).

### ADR-005: Hierarchical Continuous Aggregates for Time-Series Queries
- **Status**: Accepted
- **Context**: Dashboard queries spanning 7–30 days over millions of raw data points would be too slow. Need pre-aggregated data at multiple granularities.
- **Decision**: Hierarchical TimescaleDB continuous aggregates: raw → 1m → 1h → 1d. Each level aggregates from the previous (not raw), reducing computation. Real-time aggregation enabled for fresh data.
- **Consequences**: Additional storage (~5% overhead). Slightly delayed aggregates (5-minute lag for 1m). But queries over long ranges execute in < 500ms regardless of data volume.

### ADR-006: Dual OTA Partitions with Automatic Rollback
- **Status**: Accepted
- **Context**: Devices deployed in industrial environments may be difficult to access physically. A failed firmware update could brick devices.
- **Decision**: ESP32 dual-partition OTA scheme (ota_0/ota_1). New firmware flashed to inactive partition. If `esp_ota_mark_app_valid()` not called within 60 seconds of boot, automatic rollback to previous partition.
- **Consequences**: Firmware image limited to 1.5MB (half of available app flash). Boot counter in NVS triggers safe mode after 3 consecutive crash-reboots, still allowing MQTT+OTA for remote recovery.

## Standards Compliance

- **IEEE SWEBOK v4.0** — Used for work node categorization (Design, Construction, Security, Testing, Operations)
- **ISO/IEC/IEEE 12207:2017** — Software lifecycle processes followed for decomposition phases
- **ISO/IEC/IEEE 15289:2019** — This architecture document structure follows the Information Items standard
- **ISO 10816 / ISO 20816** — Vibration severity classification thresholds align with mechanical vibration standards for industrial machinery
- **MQTT v3.1.1 / v5.0** — Device communication follows MQTT specification
- **OpenAPI 3.0** — REST API documented per OpenAPI specification
- **OWASP Top 10** — Security considerations for web API (JWT handling, input validation, access control)

## Glossary

| Term | Definition |
|---|---|
| **RMS** | Root Mean Square — statistical measure of vibration amplitude over a time window |
| **FFT** | Fast Fourier Transform — converts time-domain vibration signal to frequency-domain spectrum |
| **Dominant Frequency** | The frequency with highest magnitude in the FFT spectrum |
| **BPFO/BPFI** | Ball Pass Frequency Outer/Inner — characteristic defect frequencies for rolling-element bearings |
| **Hypertable** | TimescaleDB abstraction that automatically partitions time-series data into chunks |
| **Continuous Aggregate** | TimescaleDB feature for automatic, incremental materialization of aggregate queries |
| **NVS** | Non-Volatile Storage — ESP32 key-value store in flash for persistent configuration |
| **OTA** | Over-The-Air — remote firmware update without physical access |
| **PSRAM** | Pseudo-Static RAM — external RAM on ESP32 modules (up to 8MB) |
| **QoS** | Quality of Service — MQTT delivery guarantee level (0=at-most-once, 1=at-least-once, 2=exactly-once) |
| **Retained Message** | MQTT message stored by broker and delivered to new subscribers immediately |
| **Z-score** | Number of standard deviations a value is from the mean — used for anomaly thresholds |
| **Light Sleep** | ESP32 power mode preserving RAM and WiFi state, ~1mA draw, < 5ms wake |
| **Core Dump** | Snapshot of task state stored to flash on crash, for post-mortem debugging |
| **Safe Mode** | Firmware fallback state after repeated crashes — only WiFi+MQTT, no sensor sampling |
