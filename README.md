# 🌡️ Sonoff Smart Climate - Home Assistant Blueprint

A flexible Home Assistant Blueprint for intelligent control of Sonoff thermostats with external temperature sensors and optional window detection.

**Works for:**
- 🏠 Single rooms with one thermostat
- 🏘️ Large rooms with multiple thermostats
- 📊 One or multiple temperature sensors (average calculated for multiple sensors)
- 🪟 Optional window detection
- ✨ **NEW:** Automatic detection of all thermostat entities!

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.6.0+-blue.svg)](https://www.home-assistant.io/)
[![Blueprint](https://img.shields.io/badge/Blueprint-automation-orange.svg)](https://www.home-assistant.io/docs/automation/using_blueprints/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📋 Table of Contents

- [Features](#-features)
- [Requirements](#-requirements)
- [Installation](#-installation)
- [Usage](#-usage)
- [Examples](#-examples)
- [License](#-license)

---

## ✨ Features

### 🌡️ Intelligent Temperature Control
- **One or multiple thermostats** - Flexible for any room
- **One or multiple sensors** - Average automatically calculated for multiple sensors
- **Precise measurement** through external sensors (Aqara, Zigbee, etc.)
- **Real-time updates** on temperature changes
- **Validation** of temperature values (Min/Max limits)

### 🪟 Window Detection (Optional)
- **Automatic heating control** with open windows
- **Multi-sensor support** - Monitors multiple windows/doors
- **Automatic detection** of the "Open Window Switch" on the thermostat
- For multiple thermostats: **ONE open = ALL notified**

### ⚙️ Flexible & Configurable
- **Automatic detection** - Blueprint finds all required entities automatically
- **Simple configuration** - Just select thermostats and sensors, the blueprint does the rest
- **Adjustable update interval** (10-300 seconds)
- **Customizable temperature limits** for validation
- **Rounding precision** configurable (0-2 decimal places)
- **Clear UI** with collapsible sections
- **One blueprint for everything** - No separate blueprint needed for zones

---

## 🔧 Requirements

- **Home Assistant** Version 2024.6.0 or higher
- **Sonoff Thermostat(s)** with "External Temperature Input" function (e.g. TRVZB, NSPanel)
- **External temperature sensor(s)** (e.g. Aqara, Zigbee)
- Optional: **Window/door contacts** for window detection

---

## 📥 Installation

### Via UI (recommended)

1. In Home Assistant navigate to: **Settings** → **Automations & Scenes** → **Blueprints**
2. Click the **Import Blueprint** button (bottom right)
3. Paste the following URL:
   ```
   https://github.com/EyJunge1/ha-sonoff-smart-climate/blob/main/blueprint.yml
   ```
4. Click **Preview** and then **Import**
5. Done! The blueprint will appear in your list

---

## 🎯 Usage

### Setting up a single thermostat

1. Create a new automation from the blueprint
2. Give it a meaningful name (e.g. "Living Room Thermostat")
3. Select **one** thermostat
4. Select **one or multiple** temperature sensors
5. Optional: Enable window detection
6. Save & activate ✅

The blueprint automatically finds the "External Temperature Input" and "Temperature Sensor Select" entities!

### Multiple thermostats (large rooms/zones)

1. Create a new automation from the blueprint
2. Give it a name (e.g. "Living Room Zone - 3 Radiators")
3. Select **multiple** thermostats
4. Select one or **multiple** temperature sensors (average will be calculated)
5. Optional: Enable window detection
6. Save & activate ✅

The blueprint automatically finds all required entities for each thermostat!

---

## 💡 Examples

### Example 1: Single thermostat (simplest case)

```yaml
alias: Bedroom Thermostat
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostats: climate.bedroom_thermostat
    temp_inputs: number.bedroom_external_temp
    temp_sensors: sensor.bedroom_temperature
```

### Example 2: Single thermostat with window detection

```yaml
alias: Kids Room Thermostat
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostats: climate.kids_room_thermostat
    temp_sensors: sensor.kids_room_temperature
    enable_window_detection: true
    window_sensors:
      - binary_sensor.kids_room_window
```

### Example 3: Large living room - 3 thermostats, 2 sensors

```yaml
alias: Living Room Zone
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostats:
      - climate.living_room_radiator_1
      - climate.living_room_radiator_2
      - climate.living_room_radiator_3
    temp_sensors:
      - sensor.living_room_temp_left_corner  # 20.5°C
      - sensor.living_room_temp_right_corner # 21.0°C
    # Average: 20.75°C will be sent to all 3 thermostats
```

**How it works:**
- Sensor 1: 20.5°C, Sensor 2: 21.0°C → **Average: 20.75°C**
- All 3 thermostats receive 20.75°C

### Example 4: Open living area with window detection

```yaml
alias: Open Living Area
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostats:
      - climate.living_room_thermostat
      - climate.kitchen_thermostat
      - climate.dining_area_thermostat
    temp_sensors:
      - sensor.living_room_temperature
      - sensor.kitchen_temperature
      - sensor.dining_area_temperature
    enable_window_detection: true
    window_sensors:
      - binary_sensor.living_room_window_1
      - binary_sensor.living_room_window_2
      - binary_sensor.kitchen_window
      - binary_sensor.patio_door
    update_interval: 45
    round_precision: 1
```

**Window logic:**
- Patio door opens → **ALL** 3 thermostats automatically activate the "Open Window" mode
- All windows closed → **ALL** 3 thermostats deactivate the "Open Window" mode

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.
