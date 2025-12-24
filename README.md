# 🌡️ Sonoff Smart Climate - Home Assistant Blueprint

Ein flexibles Home Assistant Blueprint zur intelligenten Steuerung von Sonoff Thermostaten mit externen Temperatursensoren und optionaler Fenster-Erkennung.

**Funktioniert für:**
- 🏠 Einzelne Räume mit einem Thermostat
- 🏘️ Große Räume mit mehreren Thermostaten
- 📊 Einen oder mehrere Temperatursensoren (Durchschnitt bei mehreren)
- 🪟 Optionale Fenster-Erkennung

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.6.0+-blue.svg)](https://www.home-assistant.io/)
[![Blueprint](https://img.shields.io/badge/Blueprint-automation-orange.svg)](https://www.home-assistant.io/docs/automation/using_blueprints/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📋 Inhaltsverzeichnis

- [Features](#-features)
- [Voraussetzungen](#-voraussetzungen)
- [Installation](#-installation)
- [Verwendung](#-verwendung)
- [Beispiele](#-beispiele)
- [Fehlerbehebung](#-fehlerbehebung)
- [FAQ](#-faq)
- [Changelog](#-changelog)

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
- **Einstellbares Update-Interval** (10-300 Sekunden)
- **Anpassbare Temperatur-Grenzen** für Validierung
- **Rundungs-Präzision** konfigurierbar (0-2 Nachkommastellen)
- **Übersichtliche UI** mit zusammenklappbaren Sektionen
- **Ein Blueprint für alles** - Kein separates Blueprint für Zonen nötig

## 🔧 Voraussetzungen

- **Home Assistant** Version 2024.6.0 oder höher
- **Sonoff Thermostat(e)** (z.B. TRVZB, NSPanel Thermostat)
  - Muss die "External Temperature Input" Funktion unterstützen
  - Optional: "Open Window Switch" für Fenster-Erkennung
- **Externe(r) Temperatursensor(en)** (z.B. Aqara, Zigbee Temperature Sensor)
- Optional: **Fenster-/Türkontakte** für die Fenster-Erkennung

## 📥 Installation

### Methode 1: Über die UI (empfohlen)

1. In Home Assistant navigiere zu: **Einstellungen** → **Automationen & Szenen** → **Blueprints**
2. Klicke auf den Button **Blueprint importieren** (unten rechts)
3. Füge folgende URL ein:
   ```
   https://github.com/EyJunge1/ha-sonoff-smart-climate/blob/main/blueprint.yml
   ```
4. Klicke auf **Vorschau** und dann **Importieren**

### Methode 2: Manuell

1. Lade die `blueprint.yml` herunter
2. Kopiere die Datei nach: `<config>/blueprints/automation/sonoff_smart_climate/`
3. Erstelle den Ordner falls er nicht existiert
4. Starte Home Assistant neu oder lade die Automationen neu

## 🎯 Verwendung

### Einzelnes Thermostat einrichten

1. Erstelle eine neue Automation aus dem Blueprint
2. Gib ihr einen aussagekräftigen Namen (z.B. "Wohnzimmer Thermostat")
3. Wähle **ein** Thermostat, **einen** Temp Input, **einen** Sensor
4. Optional: Aktiviere Fenster-Erkennung
5. Speichern & aktivieren

### Mehrere Thermostate (große Räume/Zonen)

1. Erstelle eine neue Automation aus dem Blueprint
2. Gib ihr einen Namen (z.B. "Wohnzimmer Zone - 3 Heizkörper")
3. Wähle **mehrere** Thermostate und entsprechende Temp Inputs
4. Wähle einen oder **mehrere** Temperatursensoren (Durchschnitt wird berechnet)
5. Optional: Aktiviere Fenster-Erkennung mit mehreren Window Switches
6. **WICHTIG:** Reihenfolge bei Thermostaten/Inputs/Switches muss übereinstimmen!

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

## 🔍 Fehlerbehebung

### Problem: Temperatur wird nicht übertragen

**Lösung:**
1. Prüfe ob die externen Sensoren funktionieren und Werte liefern
2. Überprüfe die Entity-IDs in der Automation
3. Schaue in die Logs: **Einstellungen** → **System** → **Protokolle**
4. Stelle sicher, dass die Temperatur zwischen Min/Max liegt

### Problem: Bei mehreren Thermostaten funktioniert nur eines

**Lösung:**
1. Prüfe die **Reihenfolge** von Thermostaten, Temp Inputs und Window Switches
2. Die Position muss übereinstimmen:
   - Position 1: Thermostat 1 ↔ Temp Input 1 ↔ Window Switch 1
   - Position 2: Thermostat 2 ↔ Temp Input 2 ↔ Window Switch 2
3. Gleiche **Anzahl** von Thermostaten und Temp Inputs erforderlich

### Problem: Fenster-Erkennung funktioniert nicht

**Lösung:**
1. Aktiviere "Fenster-Erkennung aktivieren"
2. Stelle sicher, dass Window Switches und Fenstersensoren konfiguriert sind
3. Prüfe ob die Fenstersensoren den korrekten Status melden (`on`/`open` für offen)

### Problem: Durchschnittstemperatur scheint falsch

**Lösung:**
1. Überprüfe ob alle Sensoren gültige Werte liefern (nicht `unavailable` oder `unknown`)
2. Ungültige Werte werden automatisch ignoriert - prüfe ob genug Sensoren verfügbar sind
3. Ändere `round_precision` falls mehr/weniger Genauigkeit gewünscht ist

## ❓ FAQ

**F: Kann ich nur ein Thermostat steuern?**  
A: Ja! Wähle einfach ein Thermostat, einen Temp Input und einen Sensor. Das Blueprint funktioniert für beides.

**F: Kann ich mehrere Thermostate mit einem Blueprint steuern?**  
A: Ja! Wähle einfach mehrere Thermostate und entsprechende Temp Inputs. Alle werden synchron mit der gleichen Temperatur versorgt.

**F: Wie funktioniert die Durchschnittsberechnung?**  
A: Alle gültigen Temperatursensoren werden addiert und durch die Anzahl geteilt. Ungültige Werte (`unavailable`, außerhalb Min/Max) werden automatisch ignoriert.

**F: Müssen die Reihenfolgen übereinstimmen?**  
A: Ja! Bei mehreren Thermostaten:
- Thermostat 1 → Temp Input 1 → Window Switch 1
- Thermostat 2 → Temp Input 2 → Window Switch 2
- usw.

**F: Was passiert wenn ein Sensor offline geht?**  
A: Der Blueprint ignoriert ungültige Werte automatisch. Solange mindestens ein Sensor gültige Werte liefert, funktioniert die Automation.

**F: Muss ich die Fenster-Erkennung nutzen?**  
A: Nein, sie ist komplett optional. Du kannst sie aktivieren oder deaktivieren.

**F: Welche Sonoff Thermostate werden unterstützt?**  
A: Alle Sonoff Thermostate die "External Temperature Input" unterstützen, z.B. TRVZB, NSPanel Thermostat.

**F: Kann ich verschiedene Update-Intervalle pro Raum haben?**  
A: Ja! Erstelle separate Automationen mit unterschiedlichen Einstellungen.

## 📝 Changelog

### Version 1.0.0 (2025-01-XX)
- 🎉 Erste öffentliche Version
- ✨ **Flexibles Blueprint** für einzelne oder mehrere Thermostate
- ✨ Durchschnittstemperatur aus mehreren Sensoren
- ✨ Gemeinsame Fenster-Erkennung für Zonen
- ✨ Konfigurierbare Einstellungen
- ✨ Input Sections für bessere UX
- 📖 Umfangreiche Dokumentation

## 📄 Lizenz

MIT License - siehe [LICENSE](LICENSE) Datei für Details.

## 🙏 Credits

- Entwickelt für die [Home Assistant](https://www.home-assistant.io/) Community
- Inspiriert von [Sonoff](https://sonoff.tech/) Thermostaten
- Icons von [Material Design Icons](https://materialdesignicons.com/)

## 🤝 Beitragen

Contributions sind willkommen! Bitte:
1. Fork das Repository
2. Erstelle einen Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit deine Änderungen (`git commit -m 'Add some AmazingFeature'`)
4. Push zum Branch (`git push origin feature/AmazingFeature`)
5. Öffne einen Pull Request

## 📞 Support

- 🐛 **Issues:** [GitHub Issues](https://github.com/EyJunge1/ha-sonoff-smart-climate/issues)
- 💬 **Diskussionen:** [Home Assistant Community](https://community.home-assistant.io/)
- 📖 **Dokumentation:** [Wiki](https://github.com/EyJunge1/ha-sonoff-smart-climate/wiki)

---

**⭐ Wenn dir dieses Blueprint gefällt, gib dem Repository einen Stern!**

Made with ❤️ for the Home Assistant Community
