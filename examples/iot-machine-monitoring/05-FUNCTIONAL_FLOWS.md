# Functional Flows — Industrial Machine Monitoring System

## Actors

| Actor | Role | Permissions |
|---|---|---|
| Sensor Device | Automated hardware attached to a machine | Sends vibration measurements, reports its own health, receives configuration changes, installs firmware updates |
| Maintenance Engineer | Monitors machine health and responds to problems | Views machine status and charts, inspects vibration trends, acknowledges and resolves alerts, adds notes to findings, adjusts detection sensitivity |
| Administrator | Manages the overall system | Everything an Engineer can do, plus: registers machines and sensors, creates and manages user accounts, pushes configuration to devices, triggers firmware updates |
| Viewer | Read-only observer | Views machine status, charts, and alert history — cannot take action on alerts or change anything |

---

## Flows by Actor

### Sensor Device

#### Flow: Collect and Send Vibration Data

**Goal**: Continuously monitor a machine's vibration and send summarized readings to the central system.

**Steps**:
1. The device powers on and connects to the factory WiFi network
2. The device reads its identity and configuration from internal storage
3. Every second, the device captures 1,024 vibration samples from the accelerometer
4. The device processes the raw samples to extract: overall vibration intensity (RMS), peak vibration, and the dominant vibration frequency
5. The device sends a compact summary of these values to the central system
6. If the device loses its network connection, it stores readings locally (up to approximately 17 minutes of data)
7. When the connection is restored, the device sends the stored readings in order, using the original timestamps

**Outcome**: The central system receives a continuous stream of vibration summaries for the machine, even across brief network outages.

**Possible errors**:
- Network unavailable for longer than 17 minutes → oldest stored readings are discarded; a count of lost readings is included in the next health report
- Sensor hardware failure → the device logs an error and continues attempting to read

---

#### Flow: Detect a Problem Locally

**Goal**: Immediately flag when vibration exceeds a safe threshold, without waiting for the central system.

**Steps**:
1. After processing each batch of samples, the device compares the vibration intensity and peak values against configured warning and critical thresholds
2. If a value exceeds the warning threshold, the device sends a warning alert to the central system and blinks a yellow light
3. If a value exceeds the critical threshold, the device sends a critical alert, turns on a steady red light, and sounds a short buzzer
4. After sending an alert, the device waits 60 seconds before sending the same type of alert again (to avoid flooding)

**Outcome**: On-site personnel see an immediate visual/audible indication, and the central system receives the alert within seconds.

**Possible errors**:
- Device is offline → alert is stored locally and sent when connection returns; the local light and buzzer still activate

---

#### Flow: Receive a Configuration Change

**Goal**: Adjust device behavior (sampling speed, alert thresholds) remotely without physical access.

**Steps**:
1. An Administrator pushes a new configuration from the dashboard
2. The device receives the new settings
3. The device checks that the configuration version is newer than its current one
4. The device saves the new settings to persistent storage and applies them immediately — no restart needed
5. If the configuration version is not newer, the device ignores it

**Outcome**: The device operates with the new settings. The change takes effect within seconds.

---

#### Flow: Install a Firmware Update

**Goal**: Update the device's software remotely.

**Steps**:
1. An Administrator triggers a firmware update from the dashboard, providing the new version and a download link
2. The device downloads the new firmware in the background (vibration monitoring continues)
3. The device verifies the download is intact (checksum verification)
4. If the download is valid, the device installs the update to a backup area and restarts
5. After restarting, the device has 60 seconds to confirm it's running correctly
6. If the device crashes before confirming, it automatically reverts to the previous firmware version
7. If the device crashes 3 times in a row, it enters a safe mode — network only, no vibration sampling — to allow another update attempt

**Outcome**: The device runs the new firmware, or safely falls back to the previous version if the update is faulty.

**Possible errors**:
- Download fails or is corrupted → the device rejects the update and continues with the current firmware
- New firmware has a bug → automatic rollback to previous version within 60 seconds

---

#### Flow: Report Device Health

**Goal**: Let the central system know the device is alive and in good condition.

**Steps**:
1. Every 60 seconds (configurable), the device reports: how long it's been running, memory usage, WiFi signal strength, battery level, firmware version, and count of any readings that were lost during outages
2. The central system records this health data and updates the device's "last seen" timestamp

**Outcome**: Administrators can see at a glance which devices are online, their signal quality, battery status, and firmware version.

---

### Maintenance Engineer

#### Flow: View the Fleet Overview

**Goal**: Get a quick picture of which machines are healthy, which need attention.

**Steps**:
1. The Engineer logs in and sees the fleet dashboard
2. The dashboard shows summary cards: total machines, how many are healthy, how many have warnings, how many are critical
3. Below the summary, each machine appears as a card (or row) showing: name, location, current status (green/yellow/red), time since last reading, and number of active alerts
4. The Engineer can filter by status (e.g., show only critical machines) or search by name or location
5. The data refreshes automatically every 30 seconds and updates in real time when new information arrives

**Outcome**: The Engineer immediately sees which machines need attention and can click through to investigate.

---

#### Flow: Inspect a Machine's Vibration History

**Goal**: Understand a machine's vibration behavior over time to diagnose developing problems.

**Steps**:
1. The Engineer clicks on a machine from the fleet overview
2. The detail page shows a chart of vibration intensity (RMS), peak vibration, and dominant frequency over time
3. The Engineer selects a time range: last hour, 6 hours, 24 hours, 7 days, or 30 days, or picks custom dates
4. The chart adjusts automatically — showing fine detail for short ranges and summarized trends for longer ranges
5. Alert events appear as colored vertical markers on the chart, so the Engineer can see when problems occurred
6. If the machine has multiple sensors, the Engineer selects which sensor to view
7. The Engineer clicks on a specific point in the chart to see a frequency spectrum (FFT), which shows which frequencies are most active — useful for diagnosing bearing wear, imbalance, or misalignment
8. When viewing the most recent time period, the chart updates live as new readings arrive

**Outcome**: The Engineer has a complete picture of the machine's vibration behavior and can identify trends, spikes, or developing issues.

---

#### Flow: Respond to an Alert

**Goal**: Investigate and resolve an anomaly detected on a machine.

**Steps**:
1. The Engineer notices a new alert — either via the alert counter in the sidebar, a browser notification, or the alert page
2. The Engineer opens the alert to see: which machine, which sensor, what type of anomaly, severity (warning or critical), and the vibration value that triggered it
3. The Engineer clicks the machine name to jump to the vibration chart and inspect the data around the alert time
4. After investigating, the Engineer acknowledges the alert, signaling "I've seen this and I'm on it"
5. The Engineer adds notes describing what they found (e.g., "Bearing shows early wear signature — scheduled replacement for next maintenance window")
6. Once the issue is addressed, the Engineer resolves the alert
7. The machine's status automatically updates — if no other open alerts remain, the machine returns to healthy (green)

**Outcome**: The alert is documented with findings, the issue is tracked through resolution, and the machine status reflects reality.

**Possible errors**:
- Attempting to resolve an alert that hasn't been acknowledged first → the system prevents this and asks the Engineer to acknowledge first

---

#### Flow: Review Alert History

**Goal**: Review past alerts to identify patterns or recurring issues.

**Steps**:
1. The Engineer opens the Alerts page
2. Alerts are shown in a table with columns: severity, machine name, type of anomaly, status, when it was detected, and who acknowledged it
3. The Engineer filters by: status (open, acknowledged, resolved), severity (warning, critical), machine, or date range
4. The Engineer clicks any alert to see full details, including notes from the person who handled it
5. Clicking the machine name navigates to that machine's vibration chart

**Outcome**: The Engineer can spot trends (e.g., "Machine #7 has had 5 bearing wear alerts this month") and recommend preventive action.

---

#### Flow: Adjust Detection Sensitivity

**Goal**: Make the anomaly detection more or less sensitive for a specific sensor.

**Steps**:
1. The Engineer opens the Devices page and finds the sensor
2. The Engineer opens the sensor's configuration panel
3. The Engineer adjusts the threshold values: warning and critical levels for vibration intensity (RMS) and peak vibration
4. The Engineer confirms the change
5. The new thresholds are sent to the device and take effect within seconds
6. The anomaly detection system on both the device and the backend uses the updated thresholds

**Outcome**: The detection is tuned appropriately — reducing false alarms on noisy machines or increasing sensitivity on critical equipment.

---

### Administrator

*Note: Administrators can do everything an Engineer can do, plus the following.*

#### Flow: Register a New Machine

**Goal**: Add a newly installed machine to the monitoring system.

**Steps**:
1. The Administrator opens the fleet dashboard and clicks "Add Machine"
2. The Administrator enters the machine's name and location
3. The system creates the machine record with a healthy (green) status

**Outcome**: The machine appears in the fleet overview, ready for sensors to be attached.

---

#### Flow: Register a Sensor to a Machine

**Goal**: Associate a newly installed ESP32 sensor with its machine.

**Steps**:
1. The Administrator opens the machine's detail page and clicks "Add Sensor"
2. The Administrator enters the sensor's unique identifier (printed on the device or shown during provisioning), its position on the machine (e.g., "Drive end bearing"), and the sensor type
3. The system registers the sensor and links it to the machine
4. Alternatively, if a sensor starts sending data before being registered, the system automatically creates a placeholder record with position marked as "unassigned" — the Administrator then updates the position

**Outcome**: The sensor is linked to its machine. Vibration data from this sensor appears under the correct machine.

---

#### Flow: Push Configuration to a Device

**Goal**: Remotely change a sensor's operating parameters.

**Steps**:
1. The Administrator opens the Devices page and selects a sensor
2. The Administrator adjusts settings: sampling speed (100, 500, or 1,000 samples per second), reporting interval, alert thresholds, and whether to include detailed frequency data
3. The Administrator clicks "Push Config" and confirms the action
4. The system sends the configuration to the device and records who made the change and when
5. Previous configurations are kept in a history log for reference

**Outcome**: The device applies the new configuration immediately.

**Possible errors**:
- Invalid values (e.g., sampling speed of 200, which isn't supported) → the system rejects the change and explains what values are accepted
- Device is offline → the configuration is stored by the message broker and delivered when the device reconnects

---

#### Flow: Trigger a Firmware Update

**Goal**: Update a sensor device's software remotely.

**Steps**:
1. The Administrator opens the Devices page and selects a sensor (or multiple sensors)
2. The Administrator clicks "Trigger OTA Update"
3. The Administrator provides: the download location of the new firmware, the version number, and a verification code (checksum)
4. The Administrator confirms the action
5. The system sends the update command to the device
6. The device downloads, verifies, and installs the update (see Device flow: "Install a Firmware Update")

**Outcome**: The device updates its firmware. The firmware version shown in the device list updates after successful installation.

---

#### Flow: Manage User Accounts

**Goal**: Control who can access the system and what they can do.

**Steps**:
1. The Administrator opens the Users page
2. To create a new user: clicks "Create User" and enters name, email, password, and role (Viewer, Engineer, or Administrator)
3. To change a user's role: clicks the user and selects the new role from a dropdown
4. To deactivate a user: clicks the deactivate button and confirms — the user can no longer log in but their historical actions (alert acknowledgments, notes) are preserved
5. The Administrator cannot change their own role or deactivate their own account (prevents accidental lockout)

**Outcome**: The right people have the right level of access to the system.

**Possible errors**:
- Email already in use → the system rejects the creation and explains the email is taken
- Trying to deactivate yourself → the system prevents this action

---

#### Flow: Monitor Device Fleet Health

**Goal**: Ensure all sensor devices are online and functioning properly.

**Steps**:
1. The Administrator opens the Devices page
2. Each device shows: sensor ID, which machine it's on, position, firmware version, when it was last heard from, WiFi signal strength (shown as signal bars), and battery level (shown as a percentage bar)
3. Devices are color-coded: green (heard from in the last 5 minutes), yellow (5–30 minutes), red (more than 30 minutes)
4. The Administrator can filter by machine or by connection status
5. Clicking a device shows a health chart: signal strength and battery over the last 24 hours
6. The Administrator can select multiple devices and push the same configuration to all of them at once (up to 20 at a time)

**Outcome**: The Administrator has full visibility into the health and status of every sensor in the field.

---

### Viewer

#### Flow: View Machine Status and History (Read-Only)

**Goal**: Check on machine status and review historical data without making changes.

**Steps**:
1. The Viewer logs in and sees the fleet overview — identical to the Engineer's view
2. The Viewer can navigate to any machine's detail page and inspect vibration charts
3. The Viewer can open the Alerts page and review alert history with full details and notes
4. The Viewer cannot acknowledge, resolve, or annotate alerts
5. The Viewer cannot access the Devices or Users pages

**Outcome**: The Viewer has full visibility into the system's data for reporting or oversight purposes, without the ability to change anything.

---

## Cross-Actor Flows

### Flow: From Anomaly to Resolution

A complete flow showing how a machine problem is detected, communicated, and resolved.

1. A **Sensor Device** detects unusually high vibration on a press machine and sends the reading to the central system
2. The central system's anomaly detection compares the reading to the machine's 24-hour baseline and determines it's significantly above normal (critical severity)
3. The system creates a new alert, sets the machine's status to critical (red), and sends notifications:
   - All connected dashboards receive a real-time update
   - Engineers with the dashboard open see the machine turn red and the alert counter increase
   - A browser notification appears: "Critical alert — Press Machine #3: RMS vibration 3x above baseline"
   - An email is sent to the configured alert recipients
4. A **Maintenance Engineer** opens the alert, reviews the vibration chart, and clicks the point where the anomaly occurred to view the frequency spectrum
5. The Engineer identifies a bearing defect frequency pattern, acknowledges the alert, and adds a note: "Inner race bearing defect detected at 142Hz. Scheduled bearing replacement for tomorrow."
6. The next day, after replacing the bearing, the Engineer verifies that vibration levels have returned to normal on the chart
7. The Engineer resolves the alert
8. The machine status returns to healthy (green)
9. An **Administrator** reviews the alert history for the month and sees a pattern of bearing failures on similar machines, leading to a preventive maintenance schedule change

---

### Flow: New Device Commissioning

How a new sensor goes from unboxing to active monitoring.

1. An **Administrator** creates the machine record in the system (name, location)
2. A technician physically installs the ESP32 sensor on the machine and powers it on
3. The **Sensor Device** connects to WiFi, then to the central system's message broker
4. The device begins sending vibration data using its unique hardware identifier
5. The system detects data from an unknown sensor and automatically creates a placeholder sensor record (position: "unassigned") linked to the machine
6. The **Administrator** opens the Devices page, finds the new sensor, and updates its position (e.g., "Non-drive end bearing") and type
7. The **Administrator** pushes a configuration to the sensor appropriate for this machine type (sampling rate, thresholds)
8. The device applies the configuration and begins operating with the correct settings
9. After 24 hours of baseline data collection, the anomaly detection begins statistical monitoring for this machine

---

### Flow: Firmware Rollout Across Fleet

How an Administrator updates firmware on multiple devices.

1. An **Administrator** prepares and tests the new firmware version
2. The Administrator selects a pilot device on a non-critical machine and triggers the update
3. The **Sensor Device** downloads, verifies, and installs the update
4. The Administrator monitors the device for 24 hours: checks that vibration data continues flowing and health metrics look normal
5. Satisfied with the pilot, the Administrator selects a batch of devices (up to 20) and triggers the update
6. Each device independently downloads and installs the update
7. If any device fails (the firmware crashes within 60 seconds), that device automatically reverts to the previous version — the Administrator sees its firmware version hasn't changed in the device list
8. The Administrator investigates any failed devices and may push a corrected firmware or leave them on the previous version

---

## State Machines

### Machine Status

Represents the overall health of a monitored machine, computed automatically from its alerts.

```
                  ┌─────────┐
    Initial ────▶ │ Healthy │ ◀──── All alerts resolved
                  │ (green) │
                  └────┬────┘
                       │
          Warning alert detected
                       │
                  ┌────▼────┐
                  │ Warning │ ◀──── Critical alert resolved,
                  │(yellow) │       but warning alerts remain
                  └────┬────┘
                       │
         Critical alert detected
                       │
                  ┌────▼────┐
                  │Critical │
                  │  (red)  │
                  └─────────┘
```

- Machine status is **never set manually** — it always reflects the worst active alert
- Resolving all alerts returns the machine to Healthy

---

### Alert Lifecycle

Represents how an alert progresses from detection to resolution.

```
                  ┌──────┐
   Detected ────▶ │ Open │
                  └──┬───┘
                     │
       Engineer acknowledges
                     │
              ┌──────▼───────┐
              │ Acknowledged │
              └──────┬───────┘
                     │
         Engineer resolves
                     │
              ┌──────▼───────┐
              │   Resolved   │
              └──────────────┘
```

**Rules:**
- Only Engineers and Administrators can acknowledge and resolve
- An alert **must** be acknowledged before it can be resolved (no skipping)
- Once resolved, an alert cannot be reopened — a new alert is created if the problem returns
- Each transition records who did it and when

---

### Sensor Device Connection Status

Represents whether a device is actively communicating, derived from its last contact time.

```
       ┌────────┐    No data for     ┌───────┐    No data for     ┌─────────┐
       │ Online │ ──── 5 minutes ──▶ │ Stale │ ──── 30 minutes ──▶│ Offline │
       │(green) │                    │(yellow)│                    │  (red)  │
       └────────┘                    └───────┘                    └─────────┘
            ▲                             │                            │
            └─── Data received ───────────┘                            │
            └─── Data received ────────────────────────────────────────┘
```

- Status is computed from the "last seen" timestamp — not stored as a separate field
- Any data received resets the device to Online

---

### Firmware Update Lifecycle

Represents the stages of a firmware update on a device.

```
       ┌───────────┐     Download      ┌──────────────┐
       │   Idle    │ ────started────▶  │ Downloading  │
       └───────────┘                   └──────┬───────┘
                                              │
                                      Verified + flashed
                                              │
                                       ┌──────▼───────┐
                                       │  Rebooting   │
                                       └──────┬───────┘
                                              │
                              ┌───────────────┼───────────────┐
                              │               │               │
                     Confirmed OK      Crashed once     Crashed 3 times
                              │               │               │
                      ┌───────▼──┐    ┌───────▼──┐    ┌───────▼──┐
                      │ Running  │    │ Rollback │    │Safe Mode │
                      │ (new FW) │    │ (old FW) │    │(recovery)│
                      └──────────┘    └──────────┘    └──────────┘
```

- Download failed or checksum mismatch → stays on current firmware, no reboot
- Rollback is automatic — no human intervention needed
- Safe mode allows network access for another update attempt

---

## Business Rules

### Access Control

1. Only Administrators can create, edit, or remove machines
2. Only Administrators can register or remove sensors
3. Only Administrators can create, modify, or deactivate user accounts
4. Only Administrators can trigger firmware updates
5. Engineers and Administrators can acknowledge and resolve alerts
6. Engineers and Administrators can push configuration to devices
7. Engineers and Administrators can add notes to alerts
8. Viewers can see everything but cannot change anything
9. An Administrator cannot change their own role or deactivate their own account
10. Deactivated users cannot log in, but their past actions remain in the system

### Machine Management

11. A machine cannot be removed if it has active sensors — sensors must be removed first
12. Removing a machine does not delete its historical data — the machine is hidden from active views but data is preserved
13. Machine status (healthy/warning/critical) is computed automatically from active alerts and cannot be set manually

### Sensor and Device Management

14. If the system receives data from an unknown sensor, it automatically creates a placeholder record linked to the machine (position: "unassigned")
15. Configuration changes take effect immediately on the device — no restart required
16. Configuration changes are versioned — the device ignores a configuration if its version number isn't newer than the current one
17. Each configuration push is recorded with who did it and when (audit trail)
18. Bulk configuration pushes are limited to 20 devices at a time

### Alert Management

19. An alert must be acknowledged before it can be resolved — no skipping the acknowledgment step
20. Once resolved, an alert cannot be reopened — a new alert is created if the issue recurs
21. If the same type of anomaly is detected on the same machine within 5 minutes, a duplicate alert is not created
22. After an alert is acknowledged, the same type of anomaly is suppressed for a configurable period (default: 1 hour) to allow time for repair
23. When all alerts on a machine are resolved, the machine automatically returns to healthy status

### Anomaly Detection

24. Statistical anomaly detection requires at least 100 readings (approximately 100 seconds of data) to establish a baseline — before that, only fixed thresholds apply
25. The baseline is calculated from the most recent 24 hours of data
26. A reading more than 3 standard deviations above the baseline triggers a warning
27. A reading more than 5 standard deviations above the baseline triggers a critical alert
28. The device also checks its own thresholds locally — if it detects a problem, it triggers both a local alarm (light/buzzer) and a remote alert, even if the central system's anomaly detection hasn't flagged it yet

### Data Retention

29. Detailed (per-second) vibration data is kept for 30 days
30. Per-minute summaries are kept for 1 year
31. Per-hour summaries are kept for 5 years
32. Per-day summaries are kept indefinitely
33. Device health data (signal strength, battery, etc.) is kept for 90 days

### Offline and Resilience

34. Devices store up to approximately 17 minutes of data during network outages; older readings are dropped if the buffer fills
35. A count of dropped readings is included in the next health report so administrators know data was lost
36. Stored readings are sent in order with their original timestamps — the data appears in charts as if the outage never happened
37. If a firmware update causes the device to crash, it automatically reverts to the previous version
38. If a device crashes 3 times in a row, it enters a recovery mode that only maintains network connectivity (no vibration monitoring) to allow a corrective update
