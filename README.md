# Sonoff Smart Climate - Home Assistant Blueprint

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.10.0+-blue.svg)](https://www.home-assistant.io/) [![Blueprint](https://img.shields.io/badge/Blueprint-automation-orange.svg)](https://www.home-assistant.io/docs/automation/using_blueprints/) [![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Intelligent control of Sonoff TRVZB thermostats with external temperature sensors — Zigbee2MQTT & ZHA, multi-TRV sync, window detection, and failsafe refresh.

## Features

- **Z2M + ZHA** — auto-detects the right entities for both
- **One or many thermostats / sensors** — average temperature, synced setpoints
- **Failsafe refresh** — re-pushes before the TRVZB ~2h external-timeout
- **Fallback** — switches to the internal sensor if external sensors fail
- **Unit conversion** — °F / K → °C
- **Skip when off** — optional; avoids waking a TRV that is turned off
- **Window detection** — instant open-window mode on all TRVs
- **Battery-friendly** — updates only when values change (+ rare failsafe push)

## Requirements

- Home Assistant **2024.10.0+**
- Sonoff **TRVZB** (firmware 1.2.1+) via Zigbee2MQTT or ZHA
- External temperature sensor(s)
- Optional: window/door contacts

## Installation

1. **Settings** → **Automations & Scenes** → **Blueprints** → **Import Blueprint**
2. Paste:
   ```
   https://github.com/EyJunge1/ha-sonoff-smart-climate/blob/main/blueprint.yaml
   ```
3. Preview → Import

## Usage

1. Create an automation from the blueprint
2. Select thermostat(s) and temperature sensor(s)
3. Optional: enable window detection / tweak settings
4. Save & activate

Related entities (`external_temperature_*`, sensor select / ZHA switch, open window) are found automatically.

## Examples

**Single thermostat**
```yaml
alias: Bedroom Thermostat
use_blueprint:
  path: eyjunge1/ha-sonoff-smart-climate/blueprint.yaml
  input:
    thermostats:
      - climate.bedroom_thermostat
    temp_sensors:
      - sensor.bedroom_temperature
```

**Zone with windows**
```yaml
alias: Living Room Zone
use_blueprint:
  path: eyjunge1/ha-sonoff-smart-climate/blueprint.yaml
  input:
    thermostats:
      - climate.living_room_radiator_1
      - climate.living_room_radiator_2
    temp_sensors:
      - sensor.living_room_temperature
    enable_window_detection: true
    window_sensors:
      - binary_sensor.living_room_window
```

## FAQ

**Sensor unavailable?** Ignored. If all fail → internal sensor. Recovers automatically.

**TRV drops to internal after ~2h?** Needs regular external updates. Blueprint re-pushes every 45 min by default. On Z2M, set `force_update: true` on the room sensor if HA never sees unchanged reports:

```yaml
'0xYOUR_SENSOR_IEEE':
  homeassistant:
    temperature:
      force_update: true
```

**TRV wakes from off?** Keep **Skip Updates When Thermostat Is Off** enabled (default).

**Z2M vs ZHA entities**

| | Temperature input | Mode |
|--|--|--|
| Z2M | `number.*_external_temperature_input` | `select.*_temperature_sensor_select` |
| ZHA | `number.*_external_temperature_sensor_value` | `switch.*_external_temperature_sensor` |

## How it works

1. **Window** (priority) — set open-window switches, then stop
2. **Average** — valid external temps (°F/K converted), ignore TRV’s own sensor
3. **Mode + push** — external/internal (Z2M options or ZHA switch), write average
4. **Sync** — on dial change, match setpoints across TRVs (`restart` mode)

Triggers: `interval` (30s), `ha_start`, `sensor_change`, `target_temp_change`, `thermostat_state`, `window_change`.

## License

MIT — see [LICENSE](LICENSE).
