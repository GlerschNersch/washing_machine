# 🧺 "Dumb" Smart Washer Automation for Home Assistant

Automate your laundry notifications and never forget a finished load again!  
This project integrates a smart plug, power monitoring, and Home Assistant templates to track washer activity and notify your household when the cycle completes.

---

## 📖 Overview

This automation package tracks your washer’s **power consumption**, **lid status**, and **runtime** to detect when a laundry cycle has finished. Once detected, Home Assistant automatically:

- Sends a push notification to your phone 📱  
- Announces completion on your smart display 🔊  
- Displays a visual popup via Browser Mod 💡  

It’s designed for any washer connected to a **power-reporting smart plug** such as a TP-Link Kasa, Shelly Plug, or Tasmota device.

---

## ⚙️ Features

- 🧠 Intelligent cycle detection using real power thresholds  
- 🔔 TTS voice announcements on Nest Hub or smart speaker  
- 💬 Push notifications for phones and tablets  
- 🖥️ Optional Browser Mod popup with dismiss button  
- 📊 Built-in daily/weekly/monthly energy tracking  
- 🎨 Integrated dashboard card for live washer status  

---
| Requirement                                                                     | Description                                                              | Link                                                                   |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| **Home Assistant** (2024.6+)                                                    | Core platform for automations                                            | [home-assistant.io](https://www.home-assistant.io/)                    |
| **Power-Reporting Smart Plug**                                                  | Any plug that reports real-time power draw (e.g., Kasa, Shelly, Tasmota) | —                                                                      |
| **[Browser Mod](https://github.com/thomasloven/hass-browser_mod)**              | Enables popups and frontend modals                                       | [GitHub](https://github.com/thomasloven/hass-browser_mod)              |
| **[Mushroom Cards](https://github.com/piitaya/lovelace-mushroom)**              | Modern UI cards for dashboard visualization                              | [GitHub](https://github.com/piitaya/lovelace-mushroom)                 |
| **[ApexCharts Card](https://github.com/RomRider/apexcharts-card)** *(optional)* | For plotting power and energy data                                       | [GitHub](https://github.com/RomRider/apexcharts-card)                  |
| **TTS Integration**                                                             | `tts.google_translate_en_com` or `tts.piper` for announcements           | [Home Assistant Docs](https://www.home-assistant.io/integrations/tts/) |

---
## 🧩 How It Works

1. The **washer_running** binary sensor activates when power draw exceeds 10 W.  
2. Once power drops below 2 W (or the lid opens), the **washer_done** sensor turns on after a short delay.  
3. When `binary_sensor.washer_done` turns **on**, an automation triggers multiple notifications and announcements.  

This ensures you only get notified when a full wash cycle has truly completed — not just when the washer pauses.

---
## ⚙️ Components Breakdown

### 1. 🧠 Binary Sensors – Washer & Dryer States

**Purpose:** Determine when the washer is actively running, has completed a cycle, or when the lid is open.

| Sensor                      | Description                                                                                                   |
|-----------------------------|---------------------------------------------------------------------------------------------------------------|
| `binary_sensor.washer_running`     | Detects when washer power draw exceeds 10 W for more than 30 seconds, marking it as “running.”                 |
| `binary_sensor.washer_done`        | Detects when washer stops drawing significant power (< 2 W) or when the lid opens after a cycle, marking it as “done.” |
| `binary_sensor.washer_lid_open`    | Determines when the lid is open by checking for very low power draw (0 – 0.8 W). Includes delay to avoid flicker.      |

**Logic Summary:**

- `washer_running` → power > 10 W  
- `washer_done` → not running AND (power < 2 W OR lid_open)  
- `washer_lid_open` → 0 W < power < 0.8 W  

These binary sensors are the foundation for notifications and dashboard indicators.

---

### 2. 🧩 Template Sensors – Status and Thresholds

**Purpose:** Create human-readable and numeric sensors for dashboards, thresholds, and logic in automations.

| Sensor                        | Description                                                                    |
|-------------------------------|--------------------------------------------------------------------------------|
| `sensor.laundry_status`       | Displays overall state: Washing, Drying, Done, or Idle based on laundry sensors.|
| `sensor.washer_power_min, max, avg` | Tracks min/max/avg power consumption from washer statistics.                      |
| `sensor.washer_running_threshold` | Defines power threshold above idle (`min + 5 W`) for "running" detection.           |
| `sensor.washer_power_instant` | Instantaneous washer power (W) for dashboards and analytics.                     |

**Example Output:**
- Laundry Status: Washing  
- Washer Power Avg: 75 W  
- Washer Running Threshold: 6.5 W  

---

### 3. ⏱️ History Stats – Usage Tracking

**Purpose:** Track daily runtime durations for both the washer and dryer.

| Sensor                     | Description                                                   |
|----------------------------|---------------------------------------------------------------|
| `sensor.washer_runtime_today` | Calculates total active minutes of washer usage since midnight. |
| `sensor.dryer_runtime_today`  | Same as above but for the dryer (optional).                  |

Show dashboards with “Total runtime today” or “Average weekly usage.”

---

### 4. ⚡ Integration Sensors – Energy Tracking

**Purpose:** Integrate power readings over time to calculate daily energy use (kWh).

| Sensor                     | Description                                              |
|----------------------------|----------------------------------------------------------|
| `sensor.washer_energy_today` | Converts power readings into total daily kWh usage.      |
| `sensor.dryer_energy_today`  | Optional; same for dryer if sensor available.            |

Turns instantaneous power (W) into cumulative energy (kWh) for cost and efficiency tracking.

---

### 5. 📊 Utility Meters – Daily, Weekly, Monthly Totals

**Purpose:** Automatically reset and track energy by time period.

| Sensor                          | Cycle    | Description                                 |
|----------------------------------|----------|---------------------------------------------|
| `utility_meter.washer_energy_daily`   | Daily    | Energy used today by the washer.             |
| `utility_meter.washer_energy_weekly`  | Weekly   | Energy consumed over the past week.          |
| `utility_meter.washer_energy_monthly` | Monthly  | Rolling total for monthly energy use.        |
| (Dryer equivalents)             |          | Same structure for dryer tracking.           |

This allows you to visualize your laundry’s energy cost trends over time.
## 🧾 Example YAML

### **Washer Package**

```yaml
binary_sensor:
  - platform: template
    sensors:
      washer_running:
        friendly_name: "Washer Running"
        value_template: >
          {{ states('sensor.washer_current_consumption') | float(0) > 10 }}
        delay_on:
          seconds: 30
        delay_off:
          minutes: 2

      washer_done:
        friendly_name: "Washer Done"
        value_template: >
          {% set power = states('sensor.washer_current_consumption') | float(0) %}
          {% set running = is_state('binary_sensor.washer_running', 'on') %}
          {% set lid_open = is_state('binary_sensor.washer_lid_open', 'on') %}
          {{ not running and (power < 2 or lid_open) }}
        delay_on:
          minutes: 5
        delay_off:
          minutes: 1
