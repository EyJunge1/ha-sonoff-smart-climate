# 🌡️ Sonoff Smart Climate - Home Assistant Blueprint

Ein flexibles und robustes Home Assistant Blueprint für intelligente Steuerung von Sonoff Thermostaten mit externen Temperatursensoren, automatischer Synchronisierung und optionaler Fenstererkennung.

**✨ Smart, Schnell und Zuverlässig** - Optimiert für Batterielaufzeit, sofortige Reaktionen und sichere Behandlung aller Fehlerszenarien.

**Funktioniert für:**
- 🏠 Einzelne Räume mit einem Thermostat
- 🏘️ Große Räume mit mehreren Thermostaten
- 📊 Einen oder mehrere Temperatursensoren (Durchschnitt wird berechnet, defekte Sensoren werden ignoriert)
- 🪟 Optionale Fenstererkennung mit sofortiger Reaktion
- 🔄 Automatische Temperatur-Synchronisierung - Drehe an einem beliebigen Thermostat, alle anderen folgen!
- ⚡ Intelligente Updates - Nur wenn sich Werte tatsächlich ändern (spart Batterie & Netzwerk)
- 🛡️ **51 Edge Cases behandelt** - Robuste Fehlerbehandlung für alle Szenarien
- 🔄 Automatischer Fallback auf internen Sensor bei Sensor-Ausfall
- ✨ Automatische Erkennung aller Thermostat-Entities!

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
- [Edge Cases & Error Handling](#️-edge-cases--error-handling)
- [Technical Details](#-technical-details)
- [License](#-license)

**📖 Weitere Dokumentation:**
- [Edge Cases & Error Handling (Detailliert)](EDGE_CASES.md) - Vollständige Übersicht aller Fehlerszenarien

---

## ✨ Features

### 🌡️ Intelligent Temperature Control
- **One or multiple thermostats** - Flexible for any room
- **Automatic temperature synchronization** - When you adjust any thermostat, all others follow automatically
- **One or multiple sensors** - Average automatically calculated for multiple sensors
- **Smart sensor validation** - Unavailable or faulty sensors are automatically ignored
- **Automatic fallback** - If all external sensors become unavailable, automatically switches to internal sensor
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

### 🛡️ Robust & Zuverlässig
- **51 Edge Cases behandelt** - Umfassende Fehlerbehandlung für alle Szenarien
- **Restart mode** - Letzte Änderung gewinnt immer, keine verlorenen Updates
- **Sichere Sensor-Behandlung** - Defekte Sensoren brechen die Automatisierung nicht ab
- **Entity-Validierung** - Prüft verfügbare Entities vor jeder Ausführung
- **Smart Change Detection** - Verhindert unnötigen Zigbee-Traffic
- **Batteriefreundlich** - Updates nur bei tatsächlichen Änderungen
- **Automatische Recovery** - Geräte werden automatisch wieder einbezogen wenn sie online kommen
- **Graceful Degradation** - System arbeitet immer mit verfügbaren Geräten
- **Trigger-Validierung** - Robuste Behandlung aller Trigger-Typen
- **Division-durch-Null-Schutz** - Mathematisch robuste Berechnungen
- **Cleaner Code** - Gut strukturiert und wartbar

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
- **If all sensors become unavailable** → Automatically switches to internal sensor 🔄
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

The blueprint automatically ignores unavailable sensors and uses only the working ones. If all external sensors become unavailable (unknown, unavailable, or invalid values), the system automatically switches to the internal sensor of the thermostat as a fallback. When external sensors become available again (checked every 30 seconds), the system automatically switches back to using the external sensors.

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

## 🛡️ Edge Cases & Error Handling

Dieses Blueprint behandelt **51 verschiedene Edge Cases** und Fehlerszenarien elegant und automatisch.

**Grundprinzip:** Das Blueprint schlägt nie vollständig fehl. Es arbeitet immer mit dem, was verfügbar ist, und stellt automatisch wieder her, wenn Geräte wieder online gehen.

**Garantien:**
- ✅ Keine Hard Failures - System arbeitet immer
- ✅ Graceful Degradation - Funktioniert mit verfügbaren Geräten
- ✅ Auto-Recovery - Maximale Verzögerung 30 Sekunden
- ✅ Keine Endlosschleifen - Smart Change Detection
- ✅ Robuste Fehlerbehandlung - Alle States werden validiert

### 📖 Vollständige Dokumentation

**→ [EDGE_CASES.md](EDGE_CASES.md)** - Detaillierte Beschreibung aller 51 Edge Cases in 9 Kategorien:
- 🌡️ Temperatursensor-Fehler (9 Cases)
- 🔧 Thermostat-Fehler (7 Cases)
- 🪟 Fensterkontakt-Fehler (6 Cases)
- ⚙️ Konfigurationsfehler (9 Cases)
- 🔧 Service & Berechnungen (4 Cases)
- 🔍 Entity Detection & Validation (4 Cases)
- 🎯 Trigger Validation & Handling (4 Cases)
- ⚡ Performance (1 Case)
- 📡 Netzwerk & Kommunikation (3 Cases)

---

## 🔧 Technical Details

### How the Blueprint Works

Das Blueprint arbeitet in **4 intelligenten Schritten** bei jedem Trigger:

#### 1. **Window Detection (PRIORITY - läuft zuerst)**
   - Prüft ob ein Fenster geöffnet oder geschlossen wurde
   - Reagiert sofort (innerhalb 1-2 Sekunden)
   - Setzt "Open Window" Modus auf allen Thermostaten
   - Unavailable Fenstersensoren werden automatisch ignoriert
   - Stoppt Ausführung nach Window-Event (Priorität)

#### 2. **Calculate Average Temperature**
   - Liest alle konfigurierten Temperatursensoren
   - Ignoriert automatisch unavailable oder defekte Sensoren
   - Berechnet Durchschnitt aus gültigen Sensoren
   - Validiert gegen Min/Max-Temperaturgrenzen
   - Float(-999) Fallback für nicht-numerische Werte
   - Division-durch-Null-Schutz

#### 3. **Configure Thermostat Sensor Mode**
   - **Valid external sensors:** Schaltet alle Thermostate auf "external" Modus
   - **All sensors unavailable:** Automatischer Fallback auf "internal" Modus
   - Auto-erkennt die korrekten Entities für jeden Thermostat
   - Prüft alle 30 Sekunden und schaltet automatisch zurück wenn Sensoren wieder verfügbar
   - Sendet Durchschnittstemperatur nur im external Modus
   - Überspringt unavailable Entities automatisch

#### 4. **Synchronize Target Temperatures** (wenn aktiviert)
   - Wenn du an einem Thermostat drehst, folgen alle anderen
   - Smart Change Detection: Update nur bei tatsächlicher Änderung
   - Verhindert Endlosschleifen durch current_temp != new_target_temp Check
   - Thermostat synchronisiert sich nicht selbst (Optimierung)
   - Validiert Temperaturbereich (5-35°C)
   - Letzte Änderung gewinnt (restart mode)

### Triggers

Die Automatisierung reagiert auf diese Events:

| Trigger | Wann | Zweck | Edge Case Behandlung |
|---------|------|-------|---------------------|
| **Time Pattern** | Alle 30 Sekunden | Regelmäßige Temperatur-Updates | Prüft Auto-Recovery, kein entity_id |
| **HA Start** | Home Assistant Neustart | Initialisierung beim Start | Alle Entities werden erkannt |
| **Sensor Change** | Temperatursensor Update | Sofortige Temperatur-Updates | Hohe Update-Frequenz durch restart mode |
| **Thermostat Change** | Du drehst am Thermostat | Sofortige Synchronisierung | Race Conditions, letzte Änderung gewinnt |
| **Thermostat State** | Thermostat kommt online | Entity Re-Detection | Automatische Rekonfiguration |
| **Window Change** | Fenster öffnet/schließt | Sofortige Heizungskontrolle | Unavailable Sensoren werden ignoriert |

### Smart Features

**Restart Mode:**
- Wenn du schnell mehrere Änderungen machst, startet die Automatisierung mit letzter Änderung neu
- Garantiert, dass letzte Anpassung immer gewinnt
- Keine verlorenen Updates oder Konflikte
- Optimal für schnelle Benutzerinteraktionen

**Smart Sensor Validation & Fallback:**
- Sensoren mit State `unavailable`, `unknown`, `none` oder `''` werden automatisch übersprungen
- Nicht-numerische Werte werden durch `float(-999)` Fallback erkannt und gefiltert
- Wenn alle externen Sensoren unavailable: Automatischer Switch auf internen Sensor
- Wenn Sensoren wieder verfügbar (geprüft alle 30s): Automatischer Switch zurück auf external
- Verhindert Senden ungültiger Temperaturen
- Garantiert kontinuierliche Funktion auch bei Sensor-Ausfällen

**Optimized Updates:**
- Temperatur wird nur gesendet wenn sie sich vom aktuellen Wert unterscheidet
- Smart Change Detection: `current_temp != new_target_temp`
- Reduziert Zigbee-Netzwerk-Traffic erheblich
- Spart Batterie bei batteriebetriebenen Thermostaten
- Sauberere Home Assistant Logs

**Automatic Entity Detection:**
- Du wählst nur Thermostate - alle anderen Entities werden automatisch gefunden
- Findet: `external_temperature_input`, `temperature_sensor_select`, `open_window`
- Prüft `device_id` für jedes Thermostat
- Filtert Multi-Zone Entities (_2, _3) automatisch heraus
- Funktioniert mit jedem Sonoff Thermostat mit diesen Entities
- Graceful Degradation wenn Entities nicht gefunden werden

**Trigger Validation:**
- Prüft `trigger.entity_id is defined` vor Verwendung
- Prüft `trigger.platform == 'state'` für korrekte Ausführung
- Validiert `trigger.to_state` und `attributes` vor Zugriff
- Verhindert Template-Fehler durch undefined Variables
- Robuste Behandlung aller Trigger-Typen

**Mathematische Robustheit:**
- Division-durch-Null wird durch explizite Checks verhindert
- Float Conversion mit Fallback-Werten (0 oder -999)
- Nachfolgende Validierung filtert fehlerhafte Werte
- Keine mathematischen Fehler möglich

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.
