# 🌡️ Sonoff Smart Climate - Home Assistant Blueprint

Ein flexibles Home Assistant Blueprint zur intelligenten Steuerung von Sonoff Thermostaten mit externen Temperatursensoren und optionaler Fenster-Erkennung.

**Funktioniert für:**
- 🏠 Einzelne Räume mit einem Thermostat
- 🏘️ Große Räume mit mehreren Thermostaten
- 📊 Einen oder mehrere Temperatursensoren (Durchschnitt bei mehreren)
- 🪟 Optionale Fenster-Erkennung
- ✨ **NEU:** Automatische Erkennung aller Thermostat-Entitäten!

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.6.0+-blue.svg)](https://www.home-assistant.io/)
[![Blueprint](https://img.shields.io/badge/Blueprint-automation-orange.svg)](https://www.home-assistant.io/docs/automation/using_blueprints/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📋 Inhaltsverzeichnis

- [Features](#-features)
- [Voraussetzungen](#-voraussetzungen)
- [Installation](#-installation)
- [Verwendung](#-verwendung)
- [Beispiele](#-beispiele)
- [Lizenz](#-lizenz)

---

## ✨ Features

### 🌡️ Intelligente Temperatur-Steuerung
- **Ein oder mehrere Thermostate** - Flexibel für jeden Raum
- **Ein oder mehrere Sensoren** - Bei mehreren wird automatisch der Durchschnitt berechnet
- **Präzise Messung** durch externe Sensoren (Aqara, Zigbee, etc.)
- **Echtzeit-Updates** bei Temperaturänderungen
- **Validierung** der Temperaturwerte (Min/Max-Grenzen)

### 🪟 Fenster-Erkennung (Optional)
- **Automatische Heizungssteuerung** bei offenen Fenstern
- **Multi-Sensor-Support** - Überwacht mehrere Fenster/Türen
- **Intelligente Steuerung** des "Open Window Switch" am Thermostat
- Bei mehreren Thermostaten: **EINES offen = ALLE informiert**

### ⚙️ Flexibel & Konfigurierbar
- **Automatische Erkennung** - Blueprint findet alle benötigten Entitäten selbst
- **Einfache Konfiguration** - Nur Thermostate und Sensoren auswählen, den Rest macht das Blueprint
- **Einstellbares Update-Interval** (10-300 Sekunden)
- **Anpassbare Temperatur-Grenzen** für Validierung
- **Rundungs-Präzision** konfigurierbar (0-2 Nachkommastellen)
- **Übersichtliche UI** mit zusammenklappbaren Sektionen
- **Ein Blueprint für alles** - Kein separates Blueprint für Zonen nötig

---

## 🔧 Voraussetzungen

- **Home Assistant** Version 2024.6.0 oder höher
- **Sonoff Thermostat(e)** mit "External Temperature Input" Funktion (z.B. TRVZB, NSPanel)
- **Externe(r) Temperatursensor(en)** (z.B. Aqara, Zigbee)
- Optional: **Fenster-/Türkontakte** für die Fenster-Erkennung

---

## 📥 Installation

### Über die UI (empfohlen)

1. In Home Assistant navigiere zu: **Einstellungen** → **Automationen & Szenen** → **Blueprints**
2. Klicke auf den Button **Blueprint importieren** (unten rechts)
3. Füge folgende URL ein:
   ```
   https://github.com/EyJunge1/ha-sonoff-smart-climate/blob/main/blueprint.yml
   ```
4. Klicke auf **Vorschau** und dann **Importieren**
5. Fertig! Das Blueprint erscheint in deiner Liste

---

## 🎯 Verwendung

### Einzelnes Thermostat einrichten

1. Erstelle eine neue Automation aus dem Blueprint
2. Gib ihr einen aussagekräftigen Namen (z.B. "Wohnzimmer Thermostat")
3. Wähle **ein** Thermostat
4. Wähle **einen oder mehrere** Temperatursensoren
5. Optional: Aktiviere Fenster-Erkennung
6. Speichern & aktivieren ✅

Das Blueprint findet automatisch die "External Temperature Input" und "Temperature Sensor Select" Entitäten!

### Mehrere Thermostate (große Räume/Zonen)

1. Erstelle eine neue Automation aus dem Blueprint
2. Gib ihr einen Namen (z.B. "Wohnzimmer Zone - 3 Heizkörper")
3. Wähle **mehrere** Thermostate
4. Wähle einen oder **mehrere** Temperatursensoren (Durchschnitt wird berechnet)
5. Optional: Aktiviere Fenster-Erkennung
6. Speichern & aktivieren ✅

Das Blueprint findet automatisch alle benötigten Entitäten für jedes Thermostat!

---

## 💡 Beispiele

### Beispiel 1: Einzelnes Thermostat (einfachster Fall)

```yaml
alias: Schlafzimmer Thermostat
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostats: climate.schlafzimmer_thermostat
    temp_inputs: number.schlafzimmer_external_temp
    temp_sensors: sensor.schlafzimmer_temperatur
```

### Beispiel 2: Einzelnes Thermostat mit Fenster-Erkennung

```yaml
alias: Kinderzimmer Thermostat
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostats: climate.kinderzimmer_thermostat
    temp_inputs: number.kinderzimmer_external_temp
    temp_sensors: sensor.kinderzimmer_temperatur
    enable_window_detection: true
    window_switches: switch.kinderzimmer_open_window
    window_sensors:
      - binary_sensor.kinderzimmer_fenster
```

### Beispiel 3: Großes Wohnzimmer - 3 Thermostate, 2 Sensoren

```yaml
alias: Wohnzimmer Zone
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostats:
      - climate.wohnzimmer_heizkoerper_1
      - climate.wohnzimmer_heizkoerper_2
      - climate.wohnzimmer_heizkoerper_3
    temp_inputs:
      - number.wohnzimmer_external_temp_1
      - number.wohnzimmer_external_temp_2
      - number.wohnzimmer_external_temp_3
    temp_sensors:
      - sensor.wohnzimmer_temp_ecke_links  # 20.5°C
      - sensor.wohnzimmer_temp_ecke_rechts # 21.0°C
    # Durchschnitt: 20.75°C wird an alle 3 Thermostate gesendet
```

**So funktioniert's:**
- Sensor 1: 20.5°C, Sensor 2: 21.0°C → **Durchschnitt: 20.75°C**
- Alle 3 Thermostate bekommen 20.75°C

### Beispiel 4: Offener Wohnbereich mit Fenster-Erkennung

```yaml
alias: Offener Wohnbereich
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostats:
      - climate.wohnzimmer_thermostat
      - climate.kueche_thermostat
      - climate.essbereich_thermostat
    temp_inputs:
      - number.wohnzimmer_external_temp
      - number.kueche_external_temp
      - number.essbereich_external_temp
    temp_sensors:
      - sensor.wohnzimmer_temperatur
      - sensor.kueche_temperatur
      - sensor.essbereich_temperatur
    enable_window_detection: true
    window_switches:
      - switch.wohnzimmer_window
      - switch.kueche_window
      - switch.essbereich_window
    window_sensors:
      - binary_sensor.wohnzimmer_fenster_1
      - binary_sensor.wohnzimmer_fenster_2
      - binary_sensor.kueche_fenster
      - binary_sensor.terrassentuer
    update_interval: 45
    round_precision: 1
```

**Fenster-Logik:**
- Terrassentür öffnet → **ALLE** 3 Window Switches gehen AN
- Alle Fenster zu → **ALLE** 3 Window Switches gehen AUS

---

## 📄 Lizenz

MIT License - siehe [LICENSE](LICENSE) Datei für Details.
