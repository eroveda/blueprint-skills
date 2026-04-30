# Original Input

This is the exact prompt provided to `/blueprint-skills:universal-framework` to start the case study.

---

## Context

The skill was invoked in an empty directory:

```bash
mkdir -p ~/iot-blueprint-test
cd ~/iot-blueprint-test
claude
```

Then in the Claude Code session:

```
/blueprint-skills:universal-framework
```

The skill detected the empty directory and prompted for a project description. The exact response was:

---

## The Prompt

> An IoT system with ESP32 sensors monitoring industrial machines.
>
> Sensors send vibration data via MQTT. Backend stores time series.
>
> Engineers see anomalies on a web dashboard.

---

## What Was NOT Provided

To highlight what the skills inferred vs. what was given:

| Provided in prompt | Not provided (inferred by skills) |
|---|---|
| 3 layers (sensors, backend, dashboard) | Specific tech stack (ESP-IDF, Python/Go/Node, React, TimescaleDB) |
| Communication protocol (MQTT) | Authentication strategy (JWT) |
| Generic actor (engineers) | Specific roles (Engineer, Admin, Viewer + Sensor Device) |
| What the system observes (vibration) | Vibration analysis approach (FFT, RMS, anomaly detection) |
| Storage type (time series) | Specific TSDB (TimescaleDB), retention policies |
| | All 6 entities (Machine, Sensor, VibrationReading, DeviceHealth, Alert, User) |
| | All 38 business rules |
| | All 6 architectural decisions (ADRs) |
| | Standards compliance (ISO 10816 for vibration severity) |

The skills produced 5,367 lines of documentation from these 3 sentences. The inference is bounded by SWEBOK structure and the universal framework — it isn't free invention.
