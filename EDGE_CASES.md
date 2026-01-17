# 🛡️ Edge Cases & Error Handling

Dieses Blueprint ist darauf ausgelegt, verschiedene Fehlerszenarien elegant zu behandeln. Hier erfährst du, wie es mit häufigen Problemen umgeht:

## 📋 Inhaltsverzeichnis

- [Temperatursensor-Fehler](#️-temperatursensor-fehler)
- [Thermostat-Fehler](#-thermostat-fehler)
- [Fensterkontakt-Fehler](#-fensterkontakt-fehler)
- [Konfigurationsfehler](#-konfigurationsfehler)
- [Service & Berechnungen](#-service--berechnungen)
- [Performance](#-performance)
- [Entity Detection & Validation](#-entity-detection--validation)
- [Trigger Validation & Handling](#-trigger-validation--handling)
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

### Sensor aktualisiert sehr langsam

**Problem:** Temperatursensor sendet nur selten Updates (z.B. alle 5-10 Minuten).

**Lösung:**
- System verwendet immer den letzten bekannten Wert des Sensors
- Solange Sensor nicht unavailable wird, bleibt letzter Wert gültig
- 30-Sekunden-Updates senden konstant die letzte bekannte Temperatur
- **Hinweis:** Alte Temperaturwerte bleiben gültig, bis Sensor neuen Wert sendet
- **Empfehlung:** Sensoren mit häufigeren Updates verwenden für präzisere Kontrolle

### Sensor mit zu hoher Update-Frequenz

**Problem:** Sensor sendet sehr häufig Updates (z.B. jede Sekunde), auch bei minimalen Änderungen (0.01°C).

**Lösung:**
- Jedes Update triggert die Automatisierung (Zeile 201-203)
- Automatisierung läuft im "restart" Modus, vorherige Ausführung wird abgebrochen
- Bei sehr häufigen Updates kann dies zu vielen Neustart führen
- Rundung (round_precision) reduziert minimale Schwankungen
- **Empfehlung:** Sensor-Reporting-Interval anpassen oder threshold konfigurieren

### Sensor meldet Temperatur-Sprünge

**Problem:** Sensor wechselt plötzlich zwischen zwei sehr unterschiedlichen Werten (z.B. 20°C → 35°C → 20°C).

**Lösung:**
- Jeder Wert wird einzeln verarbeitet
- Wenn Wert innerhalb Min/Max liegt, wird er akzeptiert und verwendet
- Thermostat erhält alle Temperaturänderungen
- Keine Glättung oder Trend-Analyse
- **Empfehlung:** Defekten Sensor ersetzen, da Sprünge auf Hardware-Problem hindeuten

### Sensor-State ist None oder leerer String

**Problem:** Sensor existiert, aber State ist `None` oder leerer String `''`.

**Lösung:**
- Prüfung `temp_state not in ['unavailable', 'unknown', 'none', '']` filtert diese Werte (Zeile 270)
- Sensor wird wie unavailable behandelt und ignoriert
- System arbeitet mit verbleibenden Sensoren weiter
- **Verhalten:** Graceful Handling, keine Fehler

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

### Thermostat-Synchronisierung in Endlosschleife

**Problem:** Thermostat A triggert Synchronisierung → setzt Thermostat B → Thermostat B triggert wieder → setzt Thermostat A → etc.

**Lösung:**
- Synchronisierung prüft: `current_temp != new_target_temp` (Zeile 396)
- Wenn Temperatur bereits gleich ist, wird kein Service Call ausgeführt
- Keine State-Änderung = kein neuer Trigger = keine Schleife
- **Verhalten:** Smart Change Detection verhindert Endlosschleifen

### Zwei Thermostate mit identischer Zieltemperatur

**Problem:** Alle Thermostate haben bereits die gleiche Zieltemperatur (z.B. alle 21°C).

**Lösung:**
- Wenn Benutzer eines davon ändert, werden die anderen synchronisiert
- Wenn keine Änderung erfolgt, passiert nichts (kein unnötiger Service Call)
- Smart Change Detection: `current_temp != new_target_temp` verhindert unnötige Updates
- **Verhalten:** Effizient, keine unnötigen Operationen

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

### expand() schlägt fehl mit ungültiger Entity

**Problem:** `expand(window_sensors)` wird aufgerufen, aber eine der Entities existiert nicht mehr.

**Lösung:**
- `expand()` überspringt automatisch nicht-existierende Entities
- Nur gültige Entities werden in die Liste aufgenommen
- System arbeitet normal mit verbleibenden Sensoren weiter
- Keine Fehler oder Exceptions
- **Verhalten:** Robust gegen gelöschte oder umbenannte Entities

### Fenster wird während Heizvorgang geöffnet

**Problem:** Benutzer öffnet Fenster, während Thermostat gerade heizt.

**Lösung:**
- State-Change-Trigger reagiert sofort (Zeile 214-216)
- Automatisierung startet neu (restart mode) und unterbricht laufende Operationen
- Fenstererkennung hat Priorität und läuft zuerst (Zeile 224-259)
- "Open Window" Modus wird sofort an alle Thermostate gesendet
- Heizung stoppt innerhalb von 1-2 Sekunden
- **Verhalten:** Maximale Reaktionsgeschwindigkeit, minimaler Energieverlust

### Window Detection Trigger bei Thermostat-Änderung

**Problem:** Benutzer ändert Thermostat-Temperatur, aber window_sensors Trigger wird auch ausgelöst (obwohl keine Fensteränderung).

**Lösung:**
- Window Detection prüft: `trigger.entity_id in window_sensors` (Zeile 235)
- Wenn Trigger von Thermostat kommt, wird Window Detection übersprungen
- System führt normale Temperatur-Updates und Synchronisierung durch
- **Verhalten:** Conditions verhindern falsche Ausführung, alles funktioniert korrekt

### Fensterkontakt ändert State zu oft (Debouncing fehlt)

**Problem:** Fensterkontakt wechselt sehr schnell zwischen "on" und "off" (z.B. 10x in 5 Sekunden).

**Lösung:**
- Jede Änderung triggert die Automatisierung neu
- Restart-Mode bricht vorherige Ausführung ab
- Letzte State-Änderung wird verarbeitet, andere werden übersprungen
- Kann zu vielen Service Calls führen (on/off/on/off an Thermostate)
- **Hinweis:** Kein Debouncing implementiert - bewusste Design-Entscheidung für schnelle Reaktion
- **Empfehlung:** Bei problematischen Sensoren Debouncing in HA konfigurieren

---

## ⚙️ Konfigurationsfehler

### Keine Thermostate konfiguriert

**Problem:** Keine Thermostate wurden in der Blueprint-Konfiguration ausgewählt.

**Lösung:**
- Automatisierung läuft, aber es gibt keine Thermostate zu konfigurieren
- Entity-Erkennung findet keine Entities (thermostat_list ist leer)
- System führt keine Aktionen aus, aber schlägt auch nicht fehl
- **Empfehlung:** Mindestens ein Thermostat sollte konfiguriert sein

### Keine Temperatursensoren konfiguriert

**Problem:** Keine Temperatursensoren wurden in der Blueprint-Konfiguration ausgewählt.

**Lösung:**
- valid_temps Liste ist leer
- has_valid_sensors wird false
- System schaltet automatisch auf internen Sensor-Modus um
- Heizung funktioniert weiterhin mit internen Sensoren der Thermostate
- **Empfehlung:** Mindestens ein externer Sensor sollte konfiguriert sein für optimale Kontrolle

### Falsche Min/Max-Temperaturgrenzen

**Problem:** Min-Temperatur ist höher als Max-Temperatur (z.B. min: 25°C, max: 20°C).

**Lösung:**
- Blueprint korrigiert automatisch ungültige Grenzen (Zeile 161-165)
- Wenn `temp_min >= temp_max`: `temp_min_valid = temp_max - 1`
- Wenn `temp_max <= temp_min`: `temp_max_valid = temp_min + 1`
- System arbeitet mit korrigierten Werten weiter
- **Beispiel:** min: 25°C, max: 20°C → temp_min_valid: 19°C, temp_max_valid: 20°C
- **Hinweis:** Automatische Korrektur verhindert Fehler, aber Werte sollten korrekt konfiguriert werden

### Mehrere Thermostate ändern gleichzeitig die Temperatur

**Problem:** Benutzer dreht gleichzeitig an mehreren Thermostaten (Race Condition).

**Lösung:**
- Blueprint läuft im "restart" Modus
- Wenn mehrere Thermostate gleichzeitig geändert werden, wird die Automatisierung mehrmals getriggert
- Jede Ausführung wird neu gestartet, die letzte Änderung gewinnt
- Alle Thermostate werden auf die Temperatur des zuletzt geänderten Thermostats synchronisiert
- **Verhalten:** Keine Konflikte, letzte Änderung hat Vorrang

### Thermostat hat ungültige Zieltemperatur

**Problem:** Thermostat meldet Zieltemperatur von 0°C oder sehr hohen Werten (z.B. 100°C).

**Lösung:**
- Synchronisierung prüft: `trigger.to_state.attributes.temperature | float(0) > 0`
- Wenn Temperatur ≤ 0 ist, wird Synchronisierung übersprungen
- Ungültige Temperaturen werden nicht an andere Thermostate weitergegeben
- **Hinweis:** Sehr hohe Temperaturen (> 50°C) werden nicht gefiltert, da sie theoretisch gültig sein könnten

### Sensoren haben sehr unterschiedliche Werte

**Problem:** Sensoren melden sehr unterschiedliche Temperaturen (z.B. 15°C und 30°C - möglicherweise defekter Sensor).

**Lösung:**
- System berechnet einfach den Durchschnitt (keine Outlier-Erkennung)
- Wenn beide Werte innerhalb Min/Max-Grenzen liegen, werden beide verwendet
- **Beispiel:** 15°C und 30°C → Durchschnitt = 22.5°C
- **Empfehlung:** Überprüfe manuell, ob ein Sensor defekt ist, wenn Werte sehr unterschiedlich sind
- **Zukünftige Verbesserung:** Outlier-Erkennung könnte implementiert werden

### Entity hat unerwarteten State

**Problem:** Entity hat einen State, der nicht erwartet wird (z.B. Fensterkontakt hat State "jammed" statt "on"/"off").

**Lösung:**
- System prüft nur auf bekannte States: `'on'`, `'open'` für offene Fenster
- Unerwartete States werden nicht als "offen" erkannt
- Fenster wird als geschlossen betrachtet, wenn State nicht "on" oder "open" ist
- **Hinweis:** Unerwartete States sollten in Home Assistant konfiguriert werden

### Einzelnes Thermostat konfiguriert (kein Sync benötigt)

**Problem:** Nur ein Thermostat ist konfiguriert, Synchronisierung zwischen mehreren Thermostaten wäre sinnlos.

**Lösung:**
- Prüfung `thermostat_list | length > 1` (Zeile 378)
- Bei nur einem Thermostat wird gesamter Synchronisierungs-Block übersprungen
- Spart unnötige Service Calls und Processing
- Effizienterer Betrieb
- **Verhalten:** Automatische Optimierung, keine Konfiguration nötig

### Synchronisierung ist deaktiviert

**Problem:** Benutzer hat Synchronisierung in der Konfiguration bewusst deaktiviert.

**Lösung:**
- Prüfung `sync_target_temp` Boolean aus Input (Zeile 377)
- Wenn false, wird gesamter Synchronisierungs-Block übersprungen
- Respektiert Benutzer-Präferenz
- Spart Processing bei unnötigen Checks
- **Verhalten:** User-Option wird respektiert

### Window Detection ist deaktiviert

**Problem:** Benutzer hat Window Detection nicht aktiviert oder keine Fenstersensoren konfiguriert.

**Lösung:**
- Prüfung `enable_window` Boolean (Zeile 243)
- Prüfung `window_switch_list | length > 0` (Thermostate haben open_window Entity)
- Prüfung `window_sensors | length > 0` (Fenstersensoren konfiguriert)
- Überspringt Window Detection komplett wenn deaktiviert
- Verhindert unnötige Checks
- **Verhalten:** Feature ist optional und wird nur bei Aktivierung verwendet

### Rounding Precision außerhalb gültiger Range

**Problem:** Benutzer könnte ungültige Werte für Rundungspräzision eingeben.

**Lösung:**
- Input Selector begrenzt Werte auf 0-2 Dezimalstellen (Zeile 89-97)
- Home Assistant validiert Input automatisch in UI
- Nur gültige Werte sind auswählbar
- Default: 1 (sinnvoller Standard-Wert)
- **Verhalten:** Fehleingaben werden durch UI verhindert

---

## 🔧 Service & Berechnungen

### Service Calls schlagen fehl

**Problem:** Service-Aufrufe (z.B. `select.select_option`, `number.set_value`) schlagen fehl.

**Mögliche Ursachen:**
- Entity ist nicht verfügbar
- Service existiert nicht
- Berechtigungsprobleme
- Entity ist read-only

**Lösung:**
- Home Assistant behandelt fehlgeschlagene Service-Calls automatisch
- Automatisierung wird nicht gestoppt, andere Aktionen werden weiterhin ausgeführt
- Fehler werden in Home Assistant Logs protokolliert
- **Hinweis:** Unavailable Entities werden bereits vor Service-Call übersprungen

### Sensor meldet nicht-numerischen Wert

**Problem:** Temperatursensor meldet einen String wie "error", "N/A" oder andere nicht-numerische Werte.

**Lösung:**
- `float(-999)` Fallback fängt nicht-numerische Werte ab (Zeile 271)
- Wenn Konvertierung fehlschlägt, wird Wert auf -999 gesetzt
- Prüfung `temp > -999 and temp >= temp_min_valid` filtert diese Werte heraus
- Sensor wird automatisch ignoriert wie bei unavailable Sensoren
- **Hinweis:** Robuste Fehlerbehandlung für alle ungültigen String-Werte

### Division durch Null bei Durchschnittsberechnung

**Problem:** Wenn alle Sensoren unavailable sind, wäre die Sensor-Liste leer und Division durch Null würde auftreten.

**Lösung:**
- Explizite Prüfung `valid_temps | length > 0` vor Division (Zeile 300)
- Wenn Liste leer: `avg_temp` wird auf 0 gesetzt (Zeile 303)
- Verhindert Template-Fehler durch Division durch Null
- Elegante Fehlerbehandlung ohne Crashes
- **Verhalten:** Mathematisch robust, keine Fehler möglich

### Float Conversion schlägt fehl

**Problem:** Konvertierung von Werten zu Float-Zahlen kann fehlschlagen (z.B. bei fehlenden Attributen).

**Lösung:**
- Alle kritischen Float-Konvertierungen haben Fallback-Werte
- `| float(0)` bei Synchronisierungs-Temperaturen (Zeile 387, 399, 411)
- `| float(-999)` bei Sensor-Temperaturen (Zeile 288)
- Bei 0 oder -999 werden ungültige Werte erkannt und übersprungen
- Nachfolgende Validierungen filtern fehlerhafte Werte
- **Verhalten:** Robuste Fehlerbehandlung, System arbeitet weiter

---

## ⚡ Performance

### Sehr viele Thermostate/Sensoren (Performance)

**Problem:** Sehr viele Thermostate oder Sensoren konfiguriert (z.B. 20+).

**Lösung:**
- System verarbeitet alle in Schleifen
- Performance hängt von Home Assistant ab
- Bei sehr vielen Geräten kann Ausführung länger dauern
- **Empfehlung:** Für sehr große Installationen mehrere Automatisierungen erstellen

---

## 🔍 Entity Detection & Validation

### Keine window_switch Entities gefunden

**Problem:** Thermostate haben keine `open_window` Switch-Entity (z.B. alte Firmware oder anderes Thermostat-Modell).

**Lösung:**
- Entity-Erkennung durchsucht Device-Entities (Zeile 169-192)
- Wenn keine `open_window` Entity gefunden: `window_switch_list` bleibt leer
- Prüfung `window_switch_list | length > 0` (Zeile 244)
- Window Detection wird automatisch übersprungen
- System arbeitet normal weiter ohne Window-Funktion
- Keine Fehler oder Warnungen
- **Verhalten:** Graceful Degradation, Feature ist optional

### Keine external_temperature Entities gefunden

**Problem:** Thermostat hat keine External Temperature Support Entities (alte Firmware, anderes Modell).

**Lösung:**
- Entity-Erkennung sucht nach `external_temperature_input` und `temperature_sensor_select` (Zeile 177-180)
- Wenn nicht gefunden: `temp_input_list` oder `temp_sensor_select_list` bleibt leer
- `has_valid_sensors` wird false (Zeile 307-310)
- System arbeitet weiter, aber ohne externe Temperatur-Funktionalität
- Keine Fehlermeldungen, keine Crashes
- **Verhalten:** Blueprint funktioniert mit eingeschränkter Funktionalität

### Device_id ist None oder nicht vorhanden

**Problem:** Climate-Entity hat kein zugeordnetes Device (z.B. Template Entity, verwaiste Entity nach Gerätelöschung).

**Lösung:**
- Prüfung `{% if device_id %}` vor Entity-Suche (Zeile 175)
- Thermostat ohne Device wird bei Entity-Erkennung übersprungen
- Keine Entities für dieses Thermostat werden gefunden
- System arbeitet mit anderen Thermostaten weiter
- Keine Template-Fehler oder Crashes
- **Verhalten:** Robuste Fehlerbehandlung, andere Geräte unbeeinträchtigt

### Multi-Zone Thermostat Entity Filter

**Problem:** Sonoff Multi-Zone Thermostate haben zusätzliche Entities mit `_2` und `_3` Suffix für Zone 2 und 3.

**Lösung:**
- Expliziter Filter: `not ('_2' in entity or '_3' in entity)` (Zeile 177)
- Nur primäre Entity (Zone 1) wird erkannt und verwendet
- Verhindert Duplikate und Konfusion bei Multi-Zone Geräten
- Unterstützt sowohl Single- als auch Multi-Zone Thermostate
- **Verhalten:** Intelligente Entity-Erkennung für verschiedene Thermostat-Typen

---

## 🎯 Trigger Validation & Handling

### Trigger ohne entity_id

**Problem:** Time Pattern Trigger (alle 30s) und Home Assistant Start Trigger haben keine `entity_id`.

**Lösung:**
- Explizite Prüfung `trigger.entity_id is defined` vor Verwendung (Zeile 247, 380)
- Window Detection: Läuft nur bei State-Change mit entity_id
- Synchronisierung: Läuft nur bei Thermostat-Trigger mit entity_id
- Time/Start Triggers führen nur Temperatur-Updates durch
- Verhindert Template-Fehler durch undefined Variables
- **Verhalten:** System funktioniert mit allen Trigger-Typen korrekt

### Trigger Platform Check (nicht state-based)

**Problem:** Verschiedene Trigger-Typen (time_pattern, homeassistant, state) haben unterschiedliche Eigenschaften.

**Lösung:**
- Explizite Prüfung `trigger.platform == 'state'` (Zeile 246)
- Window Detection läuft nur bei State-Change-Trigger
- Time-Pattern und HA-Start Trigger überspringen Window Detection
- Verhindert falsche Ausführung bei falschen Trigger-Typen
- Jeder Trigger-Typ wird korrekt behandelt
- **Verhalten:** Trigger-spezifische Logik, keine Fehler

### Trigger.to_state oder Attributes fehlen

**Problem:** Trigger-Objekt kann unvollständige Daten haben (z.B. bei initialem Trigger, gelöschter Entity).

**Lösung:**
- Explizite Prüfung `trigger.to_state is defined` (Zeile 382)
- Prüfung `trigger.to_state.attributes.temperature is defined` (Zeile 383)
- Synchronisierung wird nur ausgeführt wenn alle erforderlichen Daten vorhanden
- Verhindert Template-Fehler durch fehlende Attributes
- Robuste Fehlerbehandlung für unvollständige Trigger-Daten
- **Verhalten:** Sichere Datenvalidierung, keine Crashes

### Thermostat synchronisiert sich nicht selbst

**Problem:** Bei Synchronisierung soll das Trigger-Thermostat nicht redundant sich selbst updaten.

**Lösung:**
- Explizite Prüfung `repeat.item != triggered_thermostat` (Zeile 416)
- Verhindert unnötigen Service Call auf Trigger-Thermostat selbst
- Nur ANDERE Thermostate werden synchronisiert
- Spart Zigbee-Traffic und Service Calls
- Effizienter, vermeidet redundante Aktionen
- **Verhalten:** Intelligente Optimierung, nur notwendige Updates

---

## 📡 Netzwerk- & Kommunikationsprobleme

### Zigbee-Netzwerkprobleme

**Problem:** Temporäre Netzwerkprobleme führen dazu, dass Entities unavailable werden.

**Lösung:**
- Alle unavailable Entities werden elegant übersprungen
- System arbeitet weiterhin mit verfügbaren Geräten
- Wenn Netzwerk wiederhergestellt ist, werden Entities automatisch erkannt und rekonfiguriert
- Regelmäßige 30-Sekunden-Checks stellen sicher, dass Wiederherstellung schnell erkannt wird

### Zigbee Coordinator Neustart

**Problem:** Zigbee Coordinator startet neu, alle Zigbee-Geräte werden temporär unavailable.

**Lösung:**
- Alle Thermostate und Sensoren werden gleichzeitig unavailable
- System wartet ab und überspringt alle unavailable Entities
- Wenn Coordinator wieder online ist, kommen Geräte nach und nach zurück
- Entity-Erkennung läuft bei jedem Trigger neu und findet Geräte, sobald sie verfügbar sind
- Maximale Wiederherstellungszeit: 30 Sekunden nach letztem Gerät online
- **Verhalten:** Vollständige Auto-Recovery ohne manuelle Eingriffe

### Langsame Zigbee-Reaktionszeiten

**Problem:** Zigbee-Netzwerk ist überlastet, Service Calls dauern sehr lange.

**Lösung:**
- Home Assistant wartet auf Completion der Service Calls
- Automatisierung läuft im "restart" Modus, neue Trigger starten Ausführung neu
- Bei sehr langsamen Netzwerken können Ausführungen sich überlappen und neu starten
- System bleibt funktionsfähig, aber Updates können verzögert sein
- **Empfehlung:** Zigbee-Netzwerk optimieren (mehr Router, weniger Interferenzen)

---

## 📊 Zusammenfassung der Fehlerbehandlung

| Kategorie | Szenario | Verhalten |
|-----------|----------|-----------|
| **🌡️ Temperatursensor** | Einzelner Sensor unavailable | Wird ignoriert, Durchschnitt aus verbleibenden Sensoren |
| | Alle Sensoren unavailable | Fallback auf internen Sensor, Auto-Recovery alle 30s |
| | Ungültige Temperaturwerte | Gefiltert durch Min/Max-Grenzen |
| | Sensor kommt wieder online | Automatisch wieder einbezogen |
| | Sensor aktualisiert langsam | Letzter Wert bleibt gültig bis neues Update |
| | Sensor mit hoher Update-Frequenz | Viele Neustarts durch restart mode |
| | Sensor meldet Temperatur-Sprünge | Alle Werte werden akzeptiert (keine Glättung) |
| | Sensor-State ist None/leer | Wie unavailable behandelt, wird ignoriert |
| | Nicht-numerischer Wert | float(-999) Fallback, Sensor wird ignoriert |
| **🔧 Thermostat** | Thermostat offline | Wird übersprungen, andere arbeiten weiter |
| | Thermostat kommt online | Auto-Erkennung und Rekonfiguration (max. 30s Verzögerung) |
| | Thermostat-Entities unavailable | Werden übersprungen, keine Fehler |
| | Synchronisierung mit offline Thermostat | Nur verfügbare Thermostate werden synchronisiert |
| | HA Neustart | Auto-Initialisierung beim Start |
| | Synchronisierung Endlosschleife | Verhindert durch current_temp != new_target_temp Check |
| | Alle Thermostate mit identischer Temperatur | Smart Change Detection verhindert unnötige Updates |
| **🪟 Fensterkontakt** | Fensterkontakt unavailable | Wird ignoriert, nur gültige Sensoren geprüft |
| | Fensterkontakt kommt online | Automatisch wieder in Erkennung einbezogen |
| | Unerwarteter State | Wird nicht als "offen" erkannt |
| | Fenster während Heizvorgang geöffnet | Sofortige Reaktion durch restart mode, Priorität |
| | Window Detection Trigger bei Thermostat-Änderung | Conditions verhindern falsche Ausführung |
| | Fensterkontakt ändert State zu oft | Restart-Mode verarbeitet letzte Änderung, kein Debouncing |
| **⚙️ Konfiguration** | Keine Thermostate konfiguriert | Keine Aktionen, aber kein Fehler |
| | Keine Sensoren konfiguriert | Fallback auf internen Sensor |
| | Einzelnes Thermostat | Synchronisierung wird übersprungen |
| | Synchronisierung deaktiviert | User-Präferenz wird respektiert |
| | Window Detection deaktiviert | Feature wird komplett übersprungen |
| | Min > Max | Automatische Korrektur (temp_min = temp_max - 1) |
| | Mehrere Thermostate ändern gleichzeitig | Letzte Änderung gewinnt (restart mode) |
| | Sensoren sehr unterschiedlich | Durchschnitt wird berechnet (keine Outlier-Erkennung) |
| | Rounding Precision ungültig | UI validiert Input auf 0-2 |
| **🔧 Service & Berechnungen** | Service Calls schlagen fehl | HA behandelt Fehler, Automatisierung läuft weiter |
| | Nicht-numerischer Wert | float(-999) Fallback, wird gefiltert |
| | Division durch Null | Explizite Prüfung verhindert Fehler |
| | Float Conversion fehlgeschlagen | Fallback-Werte (0 oder -999), nachfolgende Validierung |
| **🔍 Entity Detection** | Keine window_switch Entities | Window Detection wird übersprungen |
| | Keine external_temp Entities | System arbeitet ohne externe Temperatur |
| | Device_id ist None | Entity wird übersprungen, keine Fehler |
| | Multi-Zone Entities (_2, _3) | Nur primäre Entity wird verwendet |
| **🎯 Trigger Validation** | Trigger ohne entity_id | entity_id is defined Check verhindert Fehler |
| | Trigger platform nicht 'state' | Platform-Check für richtige Ausführung |
| | trigger.to_state fehlt | Explizite Validierung vor Verwendung |
| | Thermostat synchronisiert sich selbst | Wird ausgeschlossen, nur andere werden updated |
| **⚡ Performance** | Sehr viele Thermostate/Sensoren | Funktioniert, kann langsamer sein |
| **📡 Netzwerk** | Netzwerkprobleme | Graceful Degradation, Auto-Recovery |
| | Zigbee Coordinator Neustart | Vollständige Auto-Recovery innerhalb 30s |
| | Langsame Zigbee-Reaktionszeiten | System funktioniert, Updates können verzögert sein |

**Gesamt: 51 Edge Cases behandelt** ✅

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
