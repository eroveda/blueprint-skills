# User Guide — Industrial Machine Monitoring System

## Welcome

This system helps you keep your industrial machines running smoothly by continuously monitoring their vibration. Sensors attached to your machines detect problems — like worn bearings, imbalance, or misalignment — before they cause breakdowns. You'll see the health of your entire fleet at a glance, get alerts when something looks wrong, and have the tools to investigate and respond.

This guide is for **Maintenance Engineers**, **Administrators**, and **Viewers** who use the web dashboard.

---

## Getting Started

### First Login

1. Open the dashboard in your web browser (your administrator will give you the URL)
2. Enter the email address and password provided by your administrator
3. Click **Log In**
4. You'll land on the **Fleet Overview** — a live view of all your monitored machines

`[Screenshot: Login page with email and password fields]`

That's it — you're ready to start monitoring.

### Understanding Your Role

Your administrator assigned you one of three roles when creating your account. Here's what each role can do:

| What you can do | Viewer | Engineer | Admin |
|---|---|---|---|
| View machine status and charts | Yes | Yes | Yes |
| View alert history and details | Yes | Yes | Yes |
| Acknowledge and resolve alerts | — | Yes | Yes |
| Add notes to alerts | — | Yes | Yes |
| Adjust detection thresholds | — | Yes | Yes |
| Register machines and sensors | — | — | Yes |
| Push configuration to devices | — | — | Yes |
| Trigger firmware updates | — | — | Yes |
| Manage user accounts | — | — | Yes |

If you need a different role, ask your administrator to change it.

### Navigating the Dashboard

The dashboard has a sidebar on the left with these sections:

- **Dashboard** — Fleet overview showing all machines
- **Alerts** — All alerts with filters and actions (shows a red badge with the count of open alerts)
- **Devices** — Sensor device management and health
- **Users** — User account management (visible to Administrators only)

The header at the top shows your name, role, and a logout button. A small dot next to the header indicates your live connection status: green means real-time updates are active, yellow means reconnecting, red means disconnected.

`[Screenshot: Dashboard layout with sidebar, header, and main content area labeled]`

---

## Daily Usage

### For Maintenance Engineers

These are the tasks you'll do most often, in order of frequency.

#### Check the Fleet Overview

This is your starting point every shift.

1. Click **Dashboard** in the sidebar
2. At the top, you'll see four summary cards: total machines, healthy (green), warning (yellow), and critical (red)
3. Below that, each machine appears as a card showing its name, location, status, time since last reading, and active alert count
4. To focus on problem machines, click the **Critical** or **Warning** filter button
5. To find a specific machine, type its name or location in the search box
6. The page refreshes automatically every 30 seconds — no need to reload

`[Screenshot: Fleet overview showing summary cards and machine grid with one critical machine highlighted]`

**Tip:** If a machine's "last reading" time is more than a few minutes old, its sensor may be offline. Check the Devices page.

---

#### Investigate a Machine's Vibration

When you see a warning or critical machine, here's how to dig deeper.

1. Click the machine's card on the Fleet Overview
2. You'll see a chart with three lines: vibration intensity (RMS), peak vibration, and dominant frequency
3. Use the **time range buttons** above the chart to zoom in or out: 1h, 6h, 24h, 7d, 30d, or pick custom dates
4. Vertical colored lines on the chart mark when alerts occurred — red for critical, yellow for warning
5. **Hover** over any point to see exact values and timestamp
6. **Scroll** your mouse wheel on the chart to zoom, or **click and drag** to pan
7. If the machine has multiple sensors, use the **sensor dropdown** above the chart to switch between them

`[Screenshot: Machine detail page showing vibration chart with time range selector and an alert marker]`

**To see the frequency spectrum (advanced diagnostics):**

1. Click on any point in the vibration chart
2. A second chart appears below showing which frequencies are most active at that moment
3. This is useful for identifying specific problems:
   - A spike at 1x the machine's rotation speed suggests **imbalance**
   - A spike at 2x the rotation speed suggests **misalignment**
   - Spikes at bearing defect frequencies suggest **bearing wear**

`[Screenshot: FFT spectrum chart showing a bearing defect frequency highlighted]`

---

#### Respond to an Alert

When something abnormal is detected, an alert is created automatically.

1. You'll know about new alerts in several ways:
   - The **alert badge** in the sidebar shows the count of open alerts
   - A **browser notification** pops up for critical alerts (you may need to allow notifications the first time)
   - If you're on the Fleet Overview, the affected machine turns red or yellow instantly
2. Click **Alerts** in the sidebar
3. Find the alert — the newest ones are at the top
4. Click the alert row to open its details: machine, sensor, anomaly type, severity, the vibration value that triggered it, and when it was detected
5. Click the **machine name** to jump to its vibration chart and investigate the data around the time of the alert
6. Once you've reviewed it, click **Acknowledge** to signal that you're aware and handling it
7. Add a **note** describing what you found — for example: "Bearing wear detected. Replacement parts ordered, scheduled for Thursday." Your note saves automatically when you click away
8. After the issue is fixed, return to the alert and click **Resolve**
9. The machine's status will update automatically — if there are no other open alerts, it returns to green (healthy)

`[Screenshot: Alert detail panel showing acknowledge button, notes field, and machine link]`

**Important:** You must acknowledge an alert before you can resolve it. This ensures every alert is reviewed before being closed.

---

#### Review Past Alerts

Looking at alert history helps you spot recurring problems.

1. Go to **Alerts** in the sidebar
2. Click the **Resolved** tab to see closed alerts
3. Use the filters to narrow your search:
   - **Severity:** Show only critical alerts
   - **Machine:** Show alerts for a specific machine
   - **Date range:** Show alerts from a specific period
4. Click any alert to see its full details, including who handled it and what notes they left

**Tip:** If you see the same type of alert coming up repeatedly for one machine (e.g., "bearing wear" every few weeks), it may indicate a deeper issue worth escalating.

---

#### Adjust Detection Sensitivity

If a machine is generating too many false alarms (or not catching real problems), you can tune the thresholds.

1. Go to **Devices** in the sidebar
2. Find the sensor attached to the machine you want to adjust
3. Click the sensor row to open its detail panel
4. In the **Configuration** section, adjust these values:
   - **RMS Warning Threshold** — vibration intensity that triggers a warning alert
   - **RMS Critical Threshold** — vibration intensity that triggers a critical alert
   - **Peak Warning Threshold** — peak vibration that triggers a warning
   - **Peak Critical Threshold** — peak vibration that triggers a critical
5. Click **Push Config** and confirm
6. The new thresholds take effect on both the device and the central system within seconds

`[Screenshot: Device config panel with threshold fields and Push Config button]`

**Tip:** If you're unsure what values to use, look at the machine's vibration chart over the past 7 days. Set the warning threshold slightly above the normal range, and the critical threshold at the point where you'd want immediate attention.

---

### For Administrators

You can do everything an Engineer can do, plus these additional tasks.

#### Add a New Machine

1. On the Fleet Overview, click **Add Machine**
2. Enter the machine's **name** (e.g., "CNC Lathe #4") and **location** (e.g., "Building 2, Bay 7")
3. Click **Create**
4. The machine appears in the fleet with a green (healthy) status

`[Screenshot: Add Machine dialog with name and location fields]`

---

#### Register a Sensor

After physically installing a sensor on a machine:

1. Go to the machine's detail page
2. Click **Add Sensor**
3. Enter:
   - **Sensor ID** — the unique identifier printed on the device or shown during setup
   - **Position** — where the sensor is mounted (e.g., "Drive end bearing", "Motor housing top")
   - **Type** — the type of sensor (e.g., "ADXL345", "MPU6050")
4. Click **Register**

**Automatic registration:** If the sensor is already powered on and sending data, you'll find it in the Devices page with position marked as "unassigned". Simply click it and update the position — no need to re-register.

`[Screenshot: Add Sensor dialog and the Devices page showing an unassigned sensor]`

---

#### Push Configuration to a Device

1. Go to **Devices** and click on the sensor you want to configure
2. In the Configuration section, you can adjust:
   - **Sampling rate** — how many times per second the sensor reads vibration (100, 500, or 1,000)
   - **Reporting interval** — how often the device sends health data (10 to 300 seconds)
   - **Alert thresholds** — warning and critical levels for RMS and peak vibration
   - **Include frequency data** — whether to send detailed frequency information (uses more bandwidth)
3. Click **Push Config** and confirm the action
4. You'll see a success message when the configuration is sent

**Bulk configuration:** Select up to 20 devices using the checkboxes, then click **Push Config to Selected** to apply the same settings to multiple devices at once.

The configuration history (who changed what, when) is available at the bottom of each device's detail panel.

---

#### Trigger a Firmware Update

1. Go to **Devices** and click on the sensor you want to update
2. Click **Trigger OTA Update**
3. Enter:
   - **Firmware URL** — the web address where the new firmware can be downloaded
   - **Version** — the new firmware version number
   - **Checksum** — the verification code for the firmware file (provided by the firmware build process)
4. Confirm the action
5. The device will download, verify, and install the update automatically
6. Check back after a few minutes — the firmware version in the device list should reflect the new version

`[Screenshot: OTA dialog with firmware URL, version, and checksum fields]`

**If the update fails:** The device automatically reverts to its previous firmware version. You'll see that the firmware version in the device list hasn't changed. Check the firmware file and try again.

**Recommended approach:** Test the update on one device first. Monitor it for 24 hours to make sure everything is working, then roll out to the rest of the fleet.

---

#### Manage User Accounts

1. Go to **Users** in the sidebar (only visible to Administrators)
2. You'll see a list of all users with their name, email, role, and status

**To create a new user:**
1. Click **Create User**
2. Fill in: name, email address, password, and role (Viewer, Engineer, or Administrator)
3. Click **Create**
4. Share the email and password with the new user — they can change their password after logging in

**To change a user's role:**
1. Click on the user
2. Select the new role from the dropdown
3. The change takes effect immediately — the user's permissions update on their next action

**To deactivate a user:**
1. Click on the user
2. Click **Deactivate** and confirm
3. The user can no longer log in, but their name still appears on alerts they previously handled

**Note:** You cannot change your own role or deactivate your own account. This prevents accidental lockout.

---

#### Change Your Password

Any user (regardless of role) can change their own password:

1. Click your name in the top-right header area
2. Select **Change Password**
3. Enter your current password, then your new password
4. Click **Save**

---

## Understanding the System

### Machine Statuses

| Status | Color | What it means | What to do |
|---|---|---|---|
| **Healthy** | Green | Vibration is within normal range. No active alerts. | Nothing — keep monitoring. |
| **Warning** | Yellow | Vibration is elevated above the normal baseline. A warning alert is active. | Check the vibration chart. May indicate a developing problem that should be watched. |
| **Critical** | Red | Vibration is significantly above normal. A critical alert is active. | Investigate immediately. Check the vibration chart and frequency spectrum to diagnose the issue. |

Machine status is **automatic** — it always reflects the most severe active alert. Resolving all alerts returns the machine to Healthy.

### Alert Severities

| Severity | Meaning | Typical cause |
|---|---|---|
| **Info** | Slightly unusual reading, for awareness only | Minor deviation from baseline |
| **Warning** | Elevated vibration, worth monitoring | Developing wear, early-stage imbalance |
| **Critical** | Significantly abnormal vibration, needs immediate attention | Advanced bearing wear, severe imbalance, misalignment |

### Alert Statuses

| Status | Meaning |
|---|---|
| **Open** | Just detected, no one has looked at it yet |
| **Acknowledged** | Someone has reviewed it and is working on it |
| **Resolved** | The issue has been addressed and closed |

### Device Connection Statuses

| Status | Color | Meaning |
|---|---|---|
| **Online** | Green | Data received in the last 5 minutes |
| **Stale** | Yellow | No data for 5–30 minutes — might be a temporary network issue |
| **Offline** | Red | No data for over 30 minutes — investigate the device |

---

## Common Scenarios

### Scenario: Your shift starts and you need to check on everything

1. Log in and look at the Fleet Overview
2. Check the summary cards — are any machines yellow or red?
3. If yes, click the **Warning** or **Critical** filter to see only problem machines
4. For each problem machine, click through to its detail page and review the vibration chart
5. Check the **Alerts** page for any open alerts from the previous shift
6. Acknowledge any alerts you're now responsible for
7. Check the **Devices** page to make sure all sensors are online (green)

---

### Scenario: A machine suddenly shows a critical alert

1. You'll see a browser notification or the sidebar alert badge will update
2. Go to **Alerts** and find the new critical alert at the top of the list
3. Click the alert — note the machine, the type of anomaly, and the vibration value
4. Click the machine name to view its vibration chart
5. Look at the chart around the alert time — is this a sudden spike or a gradual increase?
6. Click the alert point on the chart to see the frequency spectrum
7. If the spectrum shows a spike at a bearing defect frequency, it's likely bearing wear
8. Acknowledge the alert and add a note with your diagnosis
9. Arrange for maintenance — schedule it or escalate if immediate action is needed
10. After repairs, check the vibration chart to confirm levels are back to normal
11. Resolve the alert

---

### Scenario: A sensor keeps going offline

1. Go to **Devices** and find the sensor — it should show as red (Offline)
2. Click the sensor and look at the health chart for the last 24 hours
3. Check the **WiFi signal strength** chart — if signal is weak or dropping, the sensor may be too far from the access point or there may be interference
4. Check the **battery level** — if the sensor is battery-powered and the level is near zero, it needs charging or replacement
5. If both look fine, the sensor hardware may have an issue — check if it's physically powered on
6. If the sensor comes back online, check for any gaps in the vibration chart — stored readings from the outage should appear with their original timestamps

---

### Scenario: You want to reduce false alarms on a noisy machine

Some machines naturally vibrate more than others. If a machine keeps triggering warnings that aren't real problems:

1. Go to the machine's detail page and look at its vibration chart over 7 days
2. Note the typical RMS range during normal operation (e.g., 1.5–2.5g)
3. Go to **Devices** and find the sensor on this machine
4. Open the configuration panel
5. Set the **RMS Warning Threshold** above the normal range (e.g., 3.0g instead of the current 2.0g)
6. Set the **RMS Critical Threshold** at the level where you'd want immediate attention (e.g., 5.0g)
7. Click **Push Config** and confirm
8. Monitor for a few days to see if the alert frequency is more reasonable

---

### Scenario: A new machine needs to be added to the system

1. Ask your administrator to create the machine in the system (or do it yourself if you're an admin)
2. Install the ESP32 sensor on the machine in the correct position (e.g., on the bearing housing)
3. Power on the sensor — it will connect to the network and start sending data automatically
4. In the dashboard, the sensor will appear in the Devices page as "unassigned"
5. Update the sensor's position and type in the Devices page
6. Push an appropriate configuration to the sensor (sampling rate, thresholds)
7. For the first 24 hours, the system only uses fixed thresholds (the ones you configured). After 24 hours of data, the statistical anomaly detection kicks in and adapts to the machine's normal behavior

---

## Troubleshooting

### "I can't acknowledge or resolve alerts"

**Probably because:** Your account has the Viewer role, which is read-only.

**To fix:** Ask your administrator to change your role to Engineer or Administrator.

---

### "I can't see the Users page in the sidebar"

**Probably because:** The Users page is only visible to Administrators.

**To fix:** If you need to manage users, ask an existing administrator to upgrade your role.

---

### "The vibration chart shows a gap"

**Probably because:** The sensor lost its network connection during that period. If the gap is shorter than about 17 minutes, the data should fill in retroactively as the device sends its stored readings. If the gap is longer, some data was lost because the device's local storage filled up.

**To check:** Go to the Devices page and look at the sensor's health chart — check if the WiFi signal strength dropped during the gap period.

---

### "A machine shows as Healthy but I can see elevated vibration on the chart"

**Probably because:** The vibration is elevated but hasn't crossed the alert thresholds yet.

**To fix:** If you believe the thresholds are too high, adjust them in the sensor's configuration on the Devices page. Lower the warning threshold so the system catches the elevated vibration.

---

### "I pushed a configuration but the device doesn't seem to change"

**Probably because:** The device is currently offline and hasn't received the configuration yet.

**To fix:** The configuration is stored and will be delivered automatically when the device reconnects. Check the device's status on the Devices page — when it turns green (Online), the configuration will be applied.

---

### "The fleet overview isn't updating"

**Probably because:** The real-time connection may have dropped. Check the dot in the header:
- **Green dot** — connection is active, data should update automatically
- **Yellow dot** — reconnecting, should recover in a few seconds
- **Red dot** — disconnected, try refreshing the page

**To fix:** Refresh the browser page. If the problem persists, check your network connection.

---

### "I got logged out unexpectedly"

**Probably because:** Your login session expired. Sessions last up to 7 days, but if you close the browser, you may need to log in again.

**To fix:** Log in again with your email and password.

---

### "I can't delete a machine"

**Probably because:** The machine still has sensors registered to it. You must remove all sensors from the machine first.

**To fix:** Go to the machine's detail page, remove each sensor, then try deleting the machine again.

---

### "A firmware update didn't take effect"

**Probably because:** The update failed and the device automatically reverted to its previous firmware version. This is a safety feature — it prevents a bad update from bricking the device.

**To check:** Look at the firmware version in the Devices page. If it still shows the old version, the update failed.

**To fix:** Verify the firmware file is correct (right version, correct checksum). Try the update again. If it fails repeatedly, the firmware may have a bug — contact the firmware team.

---

## FAQs

### How quickly does the system detect a problem?

The device processes vibration data every second. If the vibration exceeds the device's local thresholds, an alert is raised within seconds. The central system's statistical analysis also evaluates every incoming reading, so anomalies based on historical patterns are detected within seconds of the data arriving.

### Can the system detect problems even if the device is offline?

Yes — partially. The device runs its own threshold checks locally. If vibration exceeds the thresholds while the device is offline, it triggers a local alarm (LED and buzzer on the device). The alert message is stored and sent to the central system when connectivity returns.

However, the central system's statistical analysis (which compares against the machine's historical baseline) only runs when data reaches the backend.

### How long is vibration data kept?

- **Detailed data** (per-second readings): 30 days
- **Minute-level summaries**: 1 year
- **Hour-level summaries**: 5 years
- **Day-level summaries**: Forever

When you zoom out on a vibration chart (e.g., looking at 30 days), the chart automatically uses summarized data so it stays fast and responsive.

### What happens if the MQTT broker or database goes down?

Devices will continue sampling and storing data locally (up to about 17 minutes). When the backend services recover, devices reconnect automatically and send their stored data. You may see a brief gap in real-time updates on the dashboard, but historical data will backfill.

### Can I get alerts by email?

Yes. Critical alerts are sent by email to a configured list of recipients. Ask your administrator if your email address is on the list.

### What do the vibration numbers mean?

- **RMS (g)** — the overall vibration intensity, measured in multiples of gravity. Higher means more vibration. A healthy machine might read 0.5–2.0g; a problem machine might read 5.0g or more.
- **Peak (g)** — the single strongest vibration impulse in the measurement window. Spikes can indicate impacts or severe defects.
- **Dominant Frequency (Hz)** — the frequency that has the most vibration energy. This helps identify the source: bearing defects, imbalance, and misalignment each produce characteristic frequencies.
- **RMS Velocity (mm/s)** — vibration intensity measured as speed rather than acceleration. Some industry standards (like ISO 10816) use velocity for severity classification.

### Can I use the dashboard on a tablet?

Yes. The dashboard is responsive and works on tablets. The sidebar collapses into a menu button on smaller screens. For control room displays, the Fleet Overview is designed to be readable at a distance.

### How do I know if a sensor's battery is low?

Go to the Devices page and look at the battery column. The battery level is shown as a percentage bar. If a sensor is plugged into mains power, the battery indicator shows "Mains" instead of a percentage.

### Why does a new machine only use fixed thresholds for the first 24 hours?

The system's statistical anomaly detection needs to learn what "normal" looks like for each machine. It needs at least 100 readings (about 100 seconds of data) for a minimum baseline, but a full 24-hour baseline gives much more reliable detection. During this learning period, the fixed thresholds you configured still protect against obvious problems.

---

## Getting Help

If you run into an issue that isn't covered here:

1. **Check the Troubleshooting section** above for common problems
2. **Contact your system administrator** — they can check device status, user accounts, and system health
3. **For system issues**, provide the following information to help with diagnosis:
   - What you were trying to do
   - What you expected to happen
   - What actually happened (exact error messages if any)
   - The machine or device name involved
   - The approximate time the issue occurred
