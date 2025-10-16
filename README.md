# Washing Machine dashboard

This repository contains a Home Assistant dashboard and helper templates to monitor a washing machine's power usage and run/done states.

## Highlights

- Lovelace dashboard in `dashboard/` with a retro green UI
- Template sensors and automations in `automation/` to detect running and done states

## Requirements

- Home Assistant (core)
- A power-measuring sensor for the washer (smart plug, energy monitor)
- Optional Lovelace cards: ApexCharts, Mushroom, Card-mod

### Hardware prerequisite

- This project assumes you have a Tapo P210M smart plug (or equivalent) capable of reporting voltage and wattage. The Tapo P210M is used here to read voltage levels and power consumption.

## Quick setup

1. Copy the files in `dashboard/` to your Home Assistant Lovelace configuration, or paste the YAML into the raw Lovelace editor.
2. Add the template sensors from `automation/` (or paste the snippet below) into `configuration.yaml` or into a split `sensors/washer.yaml`.
3. Restart Home Assistant and confirm the following entities exist (or map them to your device's entity IDs):
   - `sensor.washer_current_consumption` — power in watts
   - `binary_sensor.washer_running`
   - `sensor.washer_status`

## Template sensors (paste into your YAML)

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
```

## Repository layout

- `dashboard/` — Lovelace YAML and assets
- `automation/` — templates and automations

## License & contribution

Feel free to reuse or adapt these resources. Open an issue or PR to suggest improvements.
