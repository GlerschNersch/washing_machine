# 🧺 Smart Washer Automation for Home Assistant

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

## 🧩 How It Works

1. The **washer_running** binary sensor activates when power draw exceeds 10 W.  
2. Once power drops below 2 W (or the lid opens), the **washer_done** sensor turns on after a short delay.  
3. When `binary_sensor.washer_done` turns **on**, an automation triggers multiple notifications and announcements.  

This ensures you only get notified when a full wash cycle has truly completed — not just when the washer pauses.

---

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
