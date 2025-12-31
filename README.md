# 🌡️ Sonoff Smart Climate - Home Assistant Blueprint

A flexible and robust Home Assistant Blueprint for intelligent control of Sonoff thermostats with external temperature sensors, automatic synchronization, and optional window detection.

**✨ Smart, Fast, and Reliable** - Optimized for battery life, instant reactions, and safe handling of unavailable sensors.

**Works for:**
- 🏠 Single rooms with one thermostat
- 🏘️ Large rooms with multiple thermostats
- 📊 One or multiple temperature sensors (average calculated, unavailable sensors ignored)
- 🪟 Optional window detection with instant reaction
- 🔄 **NEW:** Automatic temperature synchronization - Turn any thermostat, all others follow!
- ⚡ **NEW:** Smart updates - Only when values actually change (saves battery & network)
- 🛡️ **NEW:** Robust error handling - Ignores unavailable sensors safely
- ✨ Automatic detection of all thermostat entities!

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
- [FAQ & Troubleshooting](#-faq--troubleshooting)
- [Technical Details](#-technical-details)
- [License](#-license)

---

## ✨ Features

### 🌡️ Intelligent Temperature Control
- **One or multiple thermostats** - Flexible for any room
- **Automatic temperature synchronization** - When you adjust any thermostat, all others follow automatically
- **One or multiple sensors** - Average automatically calculated for multiple sensors
- **Smart sensor validation** - Unavailable or faulty sensors are automatically ignored
- **Precise measurement** through external sensors (Aqara, Zigbee, etc.)
- **Real-time updates** on temperature changes
- **Validation** of temperature values (Min/Max limits)
- **Optimized updates** - Only sends changes when values actually differ

### 🪟 Window Detection (Optional)
- **Instant reaction** - No delay when windows open or close
- **Automatic heating control** with open windows
- **Multi-sensor support** - Monitors multiple windows/doors
- **Automatic detection** of the "Open Window Switch" on the thermostat
- For multiple thermostats: **ONE open = ALL notified immediately**

### ⚙️ Flexible & Configurable
- **Automatic detection** - Blueprint finds all required entities automatically
- **Simple configuration** - Just select thermostats and sensors, the blueprint does the rest
- **Regular updates** - Automatic temperature sync every 30 seconds (plus instant reactions)
- **Customizable temperature limits** for validation
- **Rounding precision** configurable (0-2 decimal places)
- **Clear UI** with collapsible sections
- **One blueprint for everything** - No separate blueprint needed for zones

### 🛡️ Robust & Reliable
- **Restart mode** - Latest changes always win, no lost updates
- **Safe sensor handling** - Unavailable sensors don't break the automation
- **Entity validation** - Checks for required entities before executing
- **Smart change detection** - Avoids unnecessary Zigbee traffic
- **Battery friendly** - Only updates when needed
- **Clean code** - Well-structured and maintainable

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
5. Optional: Enable window detection and temperature synchronization (recommended!)
6. Save & activate ✅

The blueprint automatically finds all required entities for each thermostat!

**With temperature synchronization enabled (default):**
- Turn the dial on **any** thermostat → All others automatically match that temperature
- No more conflicts between thermostats with different target temperatures
- All radiators work together for optimal heating

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
- All 3 thermostats receive 20.75°C and work together
- **Turn the dial on any thermostat to 22°C** → All 3 instantly set to 22°C ⚡
- **Turn the dial on another thermostat to 19°C** → All 3 instantly set to 19°C ⚡
- **If sensor 1 becomes unavailable** → Uses only sensor 2 (21.0°C) 🛡️
- No conflicts, no wasted energy, perfect synchronization!

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
    round_precision: 1
```

**Window logic:**
- Patio door opens → **INSTANTLY** all 3 thermostats activate "Open Window" mode
- All windows closed → **INSTANTLY** all 3 thermostats deactivate "Open Window" mode
- No delay, reacts immediately!

---

## ❓ FAQ & Troubleshooting

### What happens if a temperature sensor becomes unavailable?

The blueprint automatically ignores unavailable sensors and uses only the working ones. If all sensors are unavailable, the automation stops safely without sending invalid data.

### Why use multiple thermostats in one room?

Large rooms or rooms with multiple radiators benefit from multiple thermostats. Each radiator can be controlled individually, but they all work together with:
- Same external temperature (from shared sensors)
- Same target temperature (automatically synchronized)
- Same window detection (all react together)

### Can I disable temperature synchronization?

Yes! In the settings, you can disable "Synchronize Target Temperature". This allows each thermostat to have its own target temperature (useful for specific use cases).

### How fast does window detection react?

**Instantly!** The automation uses state triggers, so it reacts within 1-2 seconds of a window opening or closing. No waiting for the 30-second interval.

### Does this work with other Zigbee thermostats?

The blueprint is designed for **Sonoff thermostats** (TRVZB) that expose these entities:
- `external_temperature_input` (number)
- `temperature_sensor_select` (select)
- `open_window` (switch)

If your thermostat has similar entities, it might work - but test carefully!

### Will this drain my thermostat batteries?

No! The blueprint is optimized to:
- Only send updates when values actually change
- Avoid unnecessary Zigbee traffic
- Work efficiently with battery-powered devices

---

## 🔧 Technical Details

### How the Blueprint Works

The blueprint operates in **5 steps** every time it's triggered:

1. **Calculate Average Temperature**
   - Reads all configured temperature sensors
   - Automatically ignores unavailable or faulty sensors
   - Calculates the average from valid sensors
   - Validates against min/max temperature limits

2. **Configure Thermostats**
   - Sets all thermostats to use "external" temperature sensor mode
   - Auto-detects the correct sensor select entity for each thermostat

3. **Send Temperature**
   - Sends the calculated average temperature to all thermostats
   - All thermostats work with the same perceived room temperature

4. **Synchronize Target Temperatures** (if enabled)
   - When you turn the dial on any thermostat, all others follow
   - Only updates thermostats if the temperature actually changed (saves battery)
   - Last change always wins (restart mode)

5. **Window Detection** (if enabled)
   - Checks if any window/door is open
   - Instantly activates "Open Window" mode on all thermostats when opened
   - Instantly deactivates when all windows are closed

### Triggers

The automation reacts to these events:

| Trigger | When | Purpose |
|---------|------|---------|
| **Time Pattern** | Every 30 seconds | Regular temperature updates |
| **HA Start** | Home Assistant restarts | Initialize on startup |
| **Sensor Change** | Temperature sensor updates | Instant temperature updates |
| **Thermostat Change** | You turn the dial | Instant synchronization |
| **Window Change** | Window opens/closes | Instant heating control |

### Smart Features

**Restart Mode:**
- If you quickly adjust multiple thermostats, the automation restarts with the latest change
- Ensures the last adjustment always wins
- No lost updates or conflicts

**Smart Sensor Validation:**
- Sensors with state `unavailable`, `unknown`, or `none` are automatically skipped
- If all sensors are unavailable, the automation stops safely
- Prevents sending invalid temperatures to thermostats

**Optimized Updates:**
- Temperature is only sent to a thermostat if it differs from current value
- Reduces Zigbee network traffic
- Saves battery life on battery-powered thermostats
- Cleaner Home Assistant logs

**Automatic Entity Detection:**
- You only select thermostats - all other entities are found automatically
- Finds: `external_temperature_input`, `temperature_sensor_select`, `open_window`
- Works with any Sonoff thermostat that exposes these entities

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.
