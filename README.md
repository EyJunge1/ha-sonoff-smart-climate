# 🌡️ Sonoff Smart Climate - Home Assistant Blueprint

Ein Home Assistant Blueprint zur intelligenten Steuerung von Sonoff Thermostaten mit externen Temperatursensoren und optionaler Fenster-Erkennung.

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.6.0+-blue.svg)](https://www.home-assistant.io/)
[![Blueprint](https://img.shields.io/badge/Blueprint-automation-orange.svg)](https://www.home-assistant.io/docs/automation/using_blueprints/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📋 Inhaltsverzeichnis

- [Features](#-features)
- [Voraussetzungen](#-voraussetzungen)
- [Installation](#-installation)
- [Konfiguration](#️-konfiguration)
- [Verwendung](#-verwendung)
- [Beispiele](#-beispiele)
- [Fehlerbehebung](#-fehlerbehebung)
- [FAQ](#-faq)
- [Changelog](#-changelog)
- [Lizenz](#-lizenz)

## ✨ Features

### 🌡️ Temperatur-Synchronisation
- **Automatische Übertragung** von externen Temperatursensoren zu Sonoff Thermostaten
- **Präzisere Temperaturmessung** durch Nutzung genauerer externer Sensoren (z.B. Aqara, Zigbee)
- **Echtzeit-Updates** bei Temperaturänderungen
- **Validierung** der Temperaturwerte (Min/Max-Grenzen)

### 🪟 Fenster-Erkennung (Optional)
- **Automatische Heizungssteuerung** bei offenen Fenstern
- **Multi-Sensor-Support** - Überwacht mehrere Fenster/Türen pro Raum
- **Intelligente Steuerung** des "Open Window Switch" am Thermostat

### ⚙️ Flexibel & Konfigurierbar
- **Einstellbares Update-Interval** (10-300 Sekunden)
- **Anpassbare Temperatur-Grenzen** für Validierung
- **Übersichtliche UI** mit zusammenklappbaren Sektionen
- **Pro-Thermostat-Setup** - Ein Blueprint, mehrfach verwendbar

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

## ⚙️ Konfiguration

### Schritt 1: Blueprint hinzufügen

1. Gehe zu: **Einstellungen** → **Automationen & Szenen** → **Blueprints**
2. Finde "Sonoff Thermostat Sync" und klicke auf **Automation erstellen**

### Schritt 2: Thermostat Konfiguration

| Feld | Beschreibung | Erforderlich |
|------|--------------|--------------|
| **Thermostat** | Dein Sonoff Thermostat (climate entity) | ✅ Ja |
| **External Temperature Input** | Die "External Temperature Input" Number-Entity des Thermostats | ✅ Ja |
| **Temperatursensor** | Dein externer Temperatursensor (sensor entity) | ✅ Ja |

### Schritt 3: Einstellungen (Optional)

| Feld | Standard | Beschreibung |
|------|----------|--------------|
| **Update Interval** | 30s | Wie oft die Temperatur synchronisiert wird (10-300s) |
| **Minimale Temperatur** | 0°C | Untere Grenze für gültige Temperaturwerte |
| **Maximale Temperatur** | 50°C | Obere Grenze für gültige Temperaturwerte |

### Schritt 4: Fenster-Erkennung (Optional)

| Feld | Beschreibung | Erforderlich |
|------|--------------|--------------|
| **Fenster-Erkennung aktivieren** | Toggle zum Aktivieren | ❌ Nein |
| **Window Switch** | Die "Open Window Switch" Switch-Entity des Thermostats | Nur wenn aktiviert |
| **Fensterkontakte** | Alle Fenster-/Türsensoren für den Raum | Nur wenn aktiviert |

## 🎯 Verwendung

### Einzelnes Thermostat einrichten

1. Erstelle eine neue Automation aus dem Blueprint
2. Gib ihr einen aussagekräftigen Namen (z.B. "Wohnzimmer Thermostat Sync")
3. Konfiguriere die erforderlichen Felder
4. Speichern & aktivieren

### Mehrere Thermostate

Für jeden Raum/Thermostat:
- Erstelle eine **separate Automation** aus demselben Blueprint
- Jede kann **individuell konfiguriert** werden (verschiedene Intervalle, mit/ohne Fenster-Erkennung)

**Beispiel-Setup:**
- ✅ Automation 1: "Wohnzimmer Thermostat" (mit Fenster-Erkennung)
- ✅ Automation 2: "Schlafzimmer Thermostat" (mit Fenster-Erkennung)
- ✅ Automation 3: "Badezimmer Thermostat" (ohne Fenster-Erkennung)

## 💡 Beispiele

### Beispiel 1: Einfaches Setup (ohne Fenster-Erkennung)

```yaml
alias: Wohnzimmer Thermostat Sync
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostat: climate.wohnzimmer_thermostat
    temp_input: number.wohnzimmer_external_temp
    temp_sensor: sensor.wohnzimmer_temperatur
    update_interval: 30
```

### Beispiel 2: Mit Fenster-Erkennung

```yaml
alias: Schlafzimmer Thermostat Sync
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostat: climate.schlafzimmer_thermostat
    temp_input: number.schlafzimmer_external_temp
    temp_sensor: sensor.schlafzimmer_temperatur
    enable_window_detection: true
    window_switch: switch.schlafzimmer_open_window
    window_sensors:
      - binary_sensor.schlafzimmer_fenster_1
      - binary_sensor.schlafzimmer_fenster_2
```

### Beispiel 3: Angepasste Einstellungen

```yaml
alias: Badezimmer Thermostat Sync
use_blueprint:
  path: sonoff_smart_climate/blueprint.yml
  input:
    thermostat: climate.badezimmer_thermostat
    temp_input: number.badezimmer_external_temp
    temp_sensor: sensor.badezimmer_temperatur
    update_interval: 60  # Längeres Interval
    temp_min: 10         # Höhere Minimaltemperatur
    temp_max: 35         # Niedrigere Maximaltemperatur
```

## 🔍 Fehlerbehebung

### Problem: Temperatur wird nicht übertragen

**Lösung:**
1. Prüfe ob der externe Sensor funktioniert und Werte liefert
2. Überprüfe die Entity-IDs in der Automation
3. Schaue in die Logs: **Einstellungen** → **System** → **Protokolle**
4. Stelle sicher, dass die Temperatur zwischen Min/Max liegt

### Problem: Fenster-Erkennung funktioniert nicht

**Lösung:**
1. Aktiviere "Fenster-Erkennung aktivieren"
2. Stelle sicher, dass Window Switch und Fensterkontakte konfiguriert sind
3. Prüfe ob die Fenstersensoren den korrekten Status melden (`on`/`open` für offen)

### Problem: "Max exceeded" Fehler

**Lösung:**
- Das ist normal bei schnellen Temperaturänderungen
- Die Automation hat `max_exceeded: silent` - Fehler werden ignoriert
- Erhöhe ggf. das Update-Interval

### Problem: Blueprint erscheint nicht nach Import

**Lösung:**
1. Lade Automationen neu: **Entwicklerwerkzeuge** → **YAML** → **Automationen neu laden**
2. Prüfe die Home Assistant Version (min. 2024.6.0)
3. Überprüfe Syntax-Fehler in der blueprint.yml

## ❓ FAQ

**F: Kann ich mehrere Thermostate mit einem Blueprint steuern?**  
A: Nein. Das Design ist bewusst "ein Blueprint = ein Thermostat". Erstelle für jeden Raum eine separate Automation. Das macht die Konfiguration einfacher und flexibler.

**F: Welche Sonoff Thermostate werden unterstützt?**  
A: Alle Sonoff Thermostate die "External Temperature Input" unterstützen, z.B. TRVZB, NSPanel Thermostat.

**F: Funktioniert das mit anderen Thermostat-Marken?**  
A: Theoretisch ja, wenn sie eine "External Temperature Input" Number-Entity haben. Wurde aber primär für Sonoff entwickelt.

**F: Wie oft wird die Temperatur synchronisiert?**  
A: Standard ist alle 30 Sekunden, aber auch bei jeder Temperaturänderung (State-Trigger). Du kannst das Interval von 10-300 Sekunden einstellen.

**F: Was passiert wenn mein Sensor offline geht?**  
A: Der Blueprint prüft ob die Temperatur zwischen Min/Max liegt. Ungültige Werte (0 oder unavailable) werden ignoriert.

**F: Muss ich die Fenster-Erkennung nutzen?**  
A: Nein, sie ist komplett optional. Du kannst sie aktivieren oder deaktivieren.

## 📝 Changelog

### Version 1.0.0 (2025-01-XX)
- 🎉 Erste öffentliche Version
- ✨ Temperatur-Synchronisation
- ✨ Fenster-Erkennung
- ✨ Konfigurierbare Einstellungen
- ✨ Input Sections für bessere UX

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
