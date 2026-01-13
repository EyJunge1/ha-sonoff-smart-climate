# 🛡️ Edge Cases & Error Handling

Dieses Blueprint ist darauf ausgelegt, verschiedene Fehlerszenarien elegant zu behandeln. Hier erfährst du, wie es mit häufigen Problemen umgeht:

## 📋 Inhaltsverzeichnis

- [Temperatursensor-Fehler](#️-temperatursensor-fehler)
- [Thermostat-Fehler](#-thermostat-fehler)
- [Fensterkontakt-Fehler](#-fensterkontakt-fehler)
- [Netzwerk- & Kommunikationsprobleme](#-netzwerk--kommunikationsprobleme)
- [Zusammenfassung](#-zusammenfassung-der-fehlerbehandlung)

---

## 🌡️ Temperatursensor-Fehler

### Einzelner Sensor wird unavailable/unknown

**Problem:** Einer von mehreren Temperatursensoren geht offline oder meldet ungültige Werte.

**Lösung:** 
- Der unavailable Sensor wird automatisch ignoriert
- Durchschnittstemperatur wird nur aus gültigen Sensoren berechnet
- System arbeitet normal mit den verbleibenden Sensoren weiter
- **Beispiel:** 3 Sensoren (20°C, 21°C, "unknown") → Nutzt 20°C und 21°C, Durchschnitt = 20.5°C

### Alle externen Sensoren werden unavailable

**Problem:** Alle konfigurierten Temperatursensoren melden `unavailable`, `unknown`, `none` oder ungültige Werte.

**Lösung:**
- System schaltet automatisch alle Thermostate auf **internen Sensor-Modus** um (Fallback)
- Keine ungültigen Temperaturen werden an Thermostate gesendet
- Alle 30 Sekunden prüft das System, ob externe Sensoren wieder verfügbar sind
- Wenn externe Sensoren wieder verfügbar sind, wird automatisch wieder auf externen Modus umgeschaltet
- Heizung funktioniert weiterhin mit dem internen Sensor des Thermostats

### Sensor meldet ungültige Temperaturwerte

**Problem:** Sensor meldet Werte außerhalb der konfigurierten Min/Max-Grenzen (z.B. -50°C oder 100°C).

**Lösung:**
- Ungültige Werte werden durch Min/Max-Temperaturgrenzen herausgefiltert
- Nur Werte innerhalb des konfigurierten Bereichs werden verwendet
- Wenn alle Werte ungültig sind, wird Fallback auf internen Sensor ausgelöst

### Sensor kommt wieder online

**Problem:** Ein zuvor unavailable Sensor wird wieder verfügbar.

**Lösung:**
- State-Change-Trigger erkennt, wenn Sensor wieder online geht
- Bei jedem 30-Sekunden-Check wird automatisch geprüft, ob Sensoren wieder verfügbar sind
- Wenn Sensor wieder verfügbar ist, wird er automatisch wieder in die Berechnung einbezogen
- System schaltet automatisch zurück auf externen Modus, wenn alle Sensoren wieder verfügbar sind

---

## 🔧 Thermostat-Fehler

### Thermostat geht offline/unavailable

**Problem:** Ein oder mehrere Thermostate werden unavailable (Batterie leer, Netzwerkproblem, etc.).

**Lösung:**
- Unavailable Thermostate werden automatisch übersprungen
- Keine Aktionen werden auf unavailable Entities versucht
- Verbleibende Thermostate arbeiten normal weiter
- Wenn Thermostat wieder online geht, wird es automatisch erkannt und rekonfiguriert

### Thermostat kommt wieder online

**Problem:** Ein zuvor unavailable Thermostat wird wieder verfügbar.

**Lösung:**
- State-Change-Trigger erkennt, wenn Thermostat wieder online geht
- Entity-Erkennung läuft automatisch und findet die Thermostat-Entities
- Thermostat wird automatisch rekonfiguriert:
  - Sensor-Modus wird gesetzt (external/internal basierend auf Sensor-Verfügbarkeit)
  - Temperatur wird gesendet, wenn externe Sensoren gültig sind
  - Thermostat wird wieder in Synchronisierung einbezogen
- **Maximale Verzögerung:** 30 Sekunden (nächster geplanter Check)

### Thermostat-Entities werden unavailable

**Problem:** Zugehörige Entities (`external_temperature_input`, `temperature_sensor_select`, `open_window`) werden unavailable.

**Lösung:**
- Unavailable Entities werden bei der Konfiguration übersprungen
- Nur verfügbare Entities werden aktualisiert
- System arbeitet weiterhin mit verfügbaren Thermostaten
- Wenn Entities wieder verfügbar sind, werden sie automatisch wieder einbezogen

### Synchronisierung mit offline Thermostat

**Problem:** Benutzer stellt Temperatur an einem Thermostat ein, aber ein anderes ist offline.

**Lösung:**
- Offline Thermostat wird übersprungen
- Nur verfügbare Thermostate erhalten die synchronisierte Temperatur
- Wenn offline Thermostat wieder kommt, wird es automatisch in nächster Synchronisierung einbezogen

### Gemischte Verfügbarkeit bei Synchronisierung

**Problem:** Einige Thermostate sind verfügbar, andere nicht.

**Lösung:**
- Synchronisierung betrifft nur verfügbare Thermostate
- Unavailable Thermostate werden automatisch übersprungen
- Keine Fehler treten auf, System arbeitet normal weiter

### Home Assistant Neustart

**Problem:** Home Assistant startet neu, alle Entities müssen initialisiert werden.

**Lösung:**
- Automatisierung wird bei Home Assistant Start-Event ausgelöst
- Alle Entities werden automatisch erkannt
- Thermostate werden basierend auf aktueller Sensor-Verfügbarkeit konfiguriert
- System initialisiert sich korrekt beim Start

---

## 🪟 Fensterkontakt-Fehler

### Fensterkontakt wird unavailable/unknown

**Problem:** Ein oder mehrere Fenster-/Türkontakte melden `unavailable` oder `unknown`.

**Lösung:**
- Unavailable Sensoren werden bei der Fensterstatus-Prüfung ignoriert
- Nur gültige Sensoren werden für "Fenster offen"-Erkennung berücksichtigt
- Wenn alle Sensoren unavailable sind, wird kein Fenster als offen betrachtet (Heizung läuft weiter)
- Wenn Sensor wieder verfügbar ist, wird er automatisch wieder in Fenstererkennung einbezogen

**Beispiel:**
- 3 Fenstersensoren: Fenster 1 = "off", Fenster 2 = "unknown", Fenster 3 = "on"
- **Ergebnis:** Fenster 2 wird ignoriert, Fenster 3 ist offen → "Fenster offen"-Modus wird aktiviert

### Fensterkontakt kommt wieder online

**Problem:** Ein zuvor unavailable Fensterkontakt wird wieder verfügbar.

**Lösung:**
- State-Change-Trigger erkennt, wenn Fensterkontakt wieder online geht
- Sensor wird automatisch wieder in Fenstererkennung einbezogen
- Fensterstatus wird sofort neu bewertet

---

## 📡 Netzwerk- & Kommunikationsprobleme

### Zigbee-Netzwerkprobleme

**Problem:** Temporäre Netzwerkprobleme führen dazu, dass Entities unavailable werden.

**Lösung:**
- Alle unavailable Entities werden elegant übersprungen
- System arbeitet weiterhin mit verfügbaren Geräten
- Wenn Netzwerk wiederhergestellt ist, werden Entities automatisch erkannt und rekonfiguriert
- Regelmäßige 30-Sekunden-Checks stellen sicher, dass Wiederherstellung schnell erkannt wird

---

## 📊 Zusammenfassung der Fehlerbehandlung

| Kategorie | Szenario | Verhalten |
|-----------|----------|-----------|
| **🌡️ Temperatursensor** | Einzelner Sensor unavailable | Wird ignoriert, Durchschnitt aus verbleibenden Sensoren |
| | Alle Sensoren unavailable | Fallback auf internen Sensor, Auto-Recovery alle 30s |
| | Ungültige Temperaturwerte | Gefiltert durch Min/Max-Grenzen |
| | Sensor kommt wieder online | Automatisch wieder einbezogen |
| **🔧 Thermostat** | Thermostat offline | Wird übersprungen, andere arbeiten weiter |
| | Thermostat kommt online | Auto-Erkennung und Rekonfiguration (max. 30s Verzögerung) |
| | Thermostat-Entities unavailable | Werden übersprungen, keine Fehler |
| | Synchronisierung mit offline Thermostat | Nur verfügbare Thermostate werden synchronisiert |
| | HA Neustart | Auto-Initialisierung beim Start |
| **🪟 Fensterkontakt** | Fensterkontakt unavailable | Wird ignoriert, nur gültige Sensoren geprüft |
| | Fensterkontakt kommt online | Automatisch wieder in Erkennung einbezogen |
| **📡 Netzwerk** | Netzwerkprobleme | Graceful Degradation, Auto-Recovery |

### Grundprinzip

**Das Blueprint schlägt nie vollständig fehl.** Es arbeitet immer mit dem, was verfügbar ist, und stellt automatisch wieder her, wenn Geräte wieder online gehen.

### Wichtige Features

- ✅ **Automatische Fehlerbehandlung** - Keine manuellen Eingriffe nötig
- ✅ **Graceful Degradation** - System arbeitet weiter, auch wenn einige Geräte offline sind
- ✅ **Auto-Recovery** - Geräte werden automatisch wieder einbezogen, wenn sie online gehen
- ✅ **Regelmäßige Checks** - Alle 30 Sekunden wird der Status geprüft
- ✅ **Sofortige Reaktion** - State-Change-Trigger reagieren sofort auf Änderungen

---

## 🔗 Weitere Informationen

- Zurück zur [Haupt-README](../README.md)
- [Technische Details](../README.md#-technical-details)
- [FAQ & Troubleshooting](../README.md#-faq--troubleshooting)
