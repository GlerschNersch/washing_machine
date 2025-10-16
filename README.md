# 🧺 Smart Washer Dashboard (Home Assistant)

### Overview
A fully automated **washer monitoring dashboard** for Home Assistant that visually tracks wash cycles, power usage, and energy history — all in a **Pip-Boy-inspired green UI** with glowing **orange charts**.  

This dashboard automatically:
- Detects when your washer starts or stops using power thresholds  
- Displays *Running*, *Done*, and *Idle* states with animations  
- Resets to *Idle* when a new cycle begins (no manual reset needed)  
- *(Optional)* Sends a push notification when the washer finishes  
- Graphs real-time wattage and daily/monthly energy use  

---

## 🔧 Requirements

**Integrations**
- [Home Assistant Energy / Utility Meter](https://www.home-assistant.io/integrations/utility_meter/)
- [ApexCharts Card](https://github.com/RomRider/apexcharts-card)
- [Mushroom Cards](https://github.com/piitaya/lovelace-mushroom)
- [Card Mod](https://github.com/thomasloven/lovelace-card-mod)

**Entities Needed**
- `sensor.washer_current_consumption` – real-time power draw  
- `sensor.washer_energy_today` – daily energy use  
- `sensor.washer_energy_month` – monthly energy use  

---

## ⚙️ Template Sensors

Add the following to your `configuration.yaml` (or inside `/config/sensors/washer.yaml` if using splits):

```yaml
template:
  - binary_sensor:
      - name: "Washer Running"
        state: >
          {{ (states('sensor.washer_current_consumption') | float(0)) > 10 }}
        delay_on:
          seconds: 30
        delay_off:
          minutes: 2

      - name: "Washer Done"
        state: >
          {% set power = states('sensor.washer_current_consumption') | float(0) %}
          {% set running = is_state('binary_sensor.washer_running', 'on') %}
          {{ not running and power < 5 }}
        delay_on:
          minutes: 3
        delay_off:
          seconds: 5

  - sensor:
      - name: "Washer Status"
        state: >
          {% set power = states('sensor.washer_current_consumption') | float(0) %}
          {% set running = is_state('binary_sensor.washer_running', 'on') %}
          {% set done = is_state('binary_sensor.washer_done', 'on') %}
          {% if running %}
            Running
          {% elif done %}
            Done
          {% else %}
            Idle
          {% endif %}
