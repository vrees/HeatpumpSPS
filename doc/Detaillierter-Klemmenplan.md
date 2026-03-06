# Detaillierter Klemmenplan - Wärmepumpen-Steuerung

## Beckhoff CX5110-0115-9020 mit EtherCAT I/O

---

## Zusammenfassung aller I/O

| Anzahl   | Beschreibung   | Bemerkung   |
|----------|----------------|-------------|
| 2        | Analog Ein     | 4-20 mA     |
| 10       | Analog Ein     | PT1000      |
| 1        | Analog Ein     | 0-10 V      |
| 9        | Digital Ein    |             |
| 3        | Digital Aus    | 24V DC      |
| 3        | Relais Aus     | 230V AC     |

- **Gesamt: 28**
- 22 Eingänge
- 6 Ausgänge

## Übersicht der EtherCAT-Module

Alle Module sind unter dem EK1200 Bus-Koppler (Term 10) zusammengefasst.

| Term | Modul      | Typ            | Beschreibung                         | Kanäle | Physisch belegt |
|------|------------|----------------|--------------------------------------|--------|-----------------|
| 2    | **EL3152** | Analog Input   | 4-20 mA Drucksensoren Kältekreis     | 2      | 2               |
| 3    | **EL2004** | Digital Output | 24V DC Digitalausgänge               | 4      | 3               |
| 4    | **EL3204** | Analog Input   | PT1000 Temperatursensoren (1. Modul) | 4      | 4               |
| 5    | **EL3102** | Analog Input   | 0-10 V Spannungssensoren             | 2      | 1               |
| 6    | **EL1008** | Digital Input  | 24V DC Digitaleingänge (1. Modul)    | 8      | 8               |
| 7    | **EL3204** | Analog Input   | PT1000 Temperatursensoren (2. Modul) | 4      | 4               |
| 8    | **EL2622** | Relay Output   | 230V AC Relaisausgang Zwischenkreis  | 2      | 1               |
| 9    | **EL2622** | Relay Output   | 230V AC Relaisausgänge Wasserventil  | 2      | 2               |
| 18   | **EL1008** | Digital Input  | 24V DC Digitaleingänge (2. Modul)    | 8      | 1               |
| 19   | **EL3204** | Analog Input   | PT1000 Temperatursensoren (3. Modul) | 4      | 2               |

---

## 1. Term 2 (EL3152) - Analog Eingang (4-20 mA Drucksensoren)

**Technische Daten:**

- Eingangsstrom: 4-20 mA
- Auflösung: 16 Bit
- Messbereich: 0-20 mA (4-20 mA nutzbar)
- Genauigkeit: ±0.3% vom Messbereich

### Kanal 1: Hochdruck Kältekreis

| Parameter                  | Wert                                  |
|----------------------------|---------------------------------------|
| **Klemme**                 | Term 2 (EL3152) - Channel 1           |
| **Signal-Name**            | `pressureHigh`                        |
| **Variable**               | `pd.sensor.pressureHigh`              |
| **Typ**                    | Eingang, Analog (4-20 mA)             |
| **Sensor**                 | Drucksensor Hochdruckseite Kältekreis |
| **Messbereich**            | 0 - 18 bar                            |
| **Physikalischer Bereich** | 0 bar @ 4mA, 18 bar @ 20mA            |
| **Einheit**                | bar (absolut)                         |
| **Umrechnung**             | `(rawValue / 32767) * 18`             |
| **Rundung**                | 1 Dezimalstelle (0.1 bar)             |
| **Normalbetrieb**          | 8-14 bar (typisch)                    |
| **Warning-Grenze**         | > 16 bar (Software)                   |
| **Fehlergrenze**           | Hardware-Thermostat (Ranco)           |
| **Verwendung**             | Überwachung Hochdruck, COP-Berechnung |

### Kanal 2: Niederdruck Kältekreis

| Parameter                  | Wert                                                       |
|----------------------------|------------------------------------------------------------|
| **Klemme**                 | Term 2 (EL3152) - Channel 2                                |
| **Signal-Name**            | `pressureLow`                                              |
| **Variable**               | `pd.sensor.pressureLow`                                    |
| **Typ**                    | Eingang, Analog (4-20 mA)                                  |
| **Sensor**                 | Drucksensor Niederdruckseite Kältekreis                    |
| **Messbereich**            | -0.5 - 7 bar                                               |
| **Physikalischer Bereich** | -0.5 bar @ 4mA, 7 bar @ 20mA                               |
| **Einheit**                | bar (absolut)                                              |
| **Umrechnung**             | `(rawValue / 32767) * 7.5 - 0.5`                           |
| **Rundung**                | 1 Dezimalstelle (0.1 bar)                                  |
| **Normalbetrieb**          | 2-4 bar (typisch)                                          |
| **Warning-Grenze**         | > 6 bar (Software)                                         |
| **Fehlergrenze**           | Hardware-Thermostat (Ranco)                                |
| **Verwendung**             | Überwachung Niederdruck, Verdampfungstemperatur-Berechnung |

---

## 2. Term 3 (EL2004) - Digital Ausgang (24V DC)

**Technische Daten:**

- Ausgangsspannung: 24V DC (von interner Versorgung)
- Max. Strom: 0.5A pro Kanal
- Kurzschlussschutz: Ja
- Übertemperaturschutz: Ja

### Kanal 1: Kompressor Ein/Aus

| Parameter       | Wert                                |
|-----------------|-------------------------------------|
| **Klemme**      | Term 3 (EL2004) - Channel 1         |
| **Signal-Name** | `operatingStateCompressor`          |
| **Variable**    | `pd.actor.operatingStateCompressor` |
| **Typ**         | Ausgang, Digital (24V DC, 0.5A)     |
| **Last**        | Schütz für Kompressor-Motor (Spule) |
| **Logik**       | TRUE = Kompressor einschalten       |
| **Verwendung**  | Nur in **RUNNING** Zustand = TRUE   |
| **Sicherheit**  | Darf nur laufen wenn Durchfluss OK  |

### Kanal 2: Wasserpumpe Ein/Aus

| Parameter       | Wert                                        |
|-----------------|---------------------------------------------|
| **Klemme**      | Term 3 (EL2004) - Channel 2                 |
| **Signal-Name** | `operatingStateWaterPump`                   |
| **Variable**    | `pd.actor.operatingStateWaterPump`          |
| **Typ**         | Ausgang, Digital (24V DC, 0.5A)             |
| **Last**        | Schütz für Quellwasser-Pumpe (Spule)        |
| **Logik**       | TRUE = Pumpe einschalten                    |
| **Verwendung**  | In **RUNNING** und **COOLDOWN** = TRUE      |

### Kanal 3: Status-LED

| Parameter       | Wert                               |
|-----------------|------------------------------------|
| **Klemme**      | Term 3 (EL2004) - Channel 3        |
| **Signal-Name** | `operatingStateLed`                |
| **Variable**    | `pd.actor.operatingStateLed`       |
| **Typ**         | Ausgang, Digital (24V DC, 0.5A)    |
| **Last**        | Betriebs-LED oder Signallampe      |
| **Logik**       | TRUE = LED ein (Betriebsbereit)    |
| **Verwendung**  | Alle Zustände außer **OFF** = TRUE |

### Kanal 4: Nicht belegt

| Parameter  | Wert                   |
|------------|------------------------|
| **Klemme** | Term 3 (EL2004) - Channel 4 |
| **Status** | Nicht belegt / Reserve |

---

## 3. Term 4 (EL3204) - Analog Eingang PT1000 (1. Modul)

**Technische Daten:**

- Sensortyp: PT1000 (Widerstandsthermometer)
- Auflösung: 16 Bit
- Messbereich: -200°C bis +850°C
- Genauigkeit: ±0.1°C @ 25°C

### Kanal 1: Verdampfer Austrittstemperatur

| Parameter         | Wert                                            |
|-------------------|-------------------------------------------------|
| **Klemme**        | Term 4 (EL3204) - RTD Channel 1                 |
| **Signal-Name**   | `temperatureEvaporatingOut`                     |
| **Variable**      | `pd.sensor.temperatureEvaporatingOut`           |
| **Typ**           | Eingang, Analog (PT1000)                        |
| **Sensor**        | PT1000 am Verdampfer-Ausgang (Quellwasser)      |
| **Messbereich**   | -50°C - +100°C (typisch)                        |
| **Einheit**       | °C                                              |
| **Umrechnung**    | `rawValue / 10` (Modul liefert °C * 10)         |
| **Rundung**       | 1 Dezimalstelle (0.1°C)                         |
| **Normalbetrieb** | 5-12°C (abhängig von Quelltemperatur)           |
| **Fehlergrenze**  | < 2.3°C (Gefrierschutz!)                        |
| **Verwendung**    | Überwachung Gefriergefahr, Verdampfungsleistung |

### Kanal 2: Verdampfer Eintrittstemperatur

| Parameter         | Wert                                           |
|-------------------|------------------------------------------------|
| **Klemme**        | Term 4 (EL3204) - RTD Channel 2                |
| **Signal-Name**   | `temperatureEvaporatingIn`                     |
| **Variable**      | `pd.sensor.temperatureEvaporatingIn`           |
| **Typ**           | Eingang, Analog (PT1000)                       |
| **Sensor**        | PT1000 am Verdampfer-Eingang (Quellwasser)     |
| **Messbereich**   | -50°C - +100°C (typisch)                       |
| **Einheit**       | °C                                             |
| **Umrechnung**    | `rawValue / 10`                                |
| **Rundung**       | 1 Dezimalstelle (0.1°C)                        |
| **Normalbetrieb** | 8-15°C (abhängig von Quelltemperatur)          |
| **Verwendung**    | Berechnung Wärmequellenleistung, ΔT Verdampfer |

### Kanal 3: Vorlauftemperatur

| Parameter         | Wert                                                |
|-------------------|-----------------------------------------------------|
| **Klemme**        | Term 4 (EL3204) - RTD Channel 3                     |
| **Signal-Name**   | `temperatureFlow`                                   |
| **Variable**      | `pd.sensor.temperatureFlow`                         |
| **Typ**           | Eingang, Analog (PT1000)                            |
| **Sensor**        | PT1000 am Wärmepumpen-Ausgang (Vorlauf)             |
| **Messbereich**   | 0°C - +80°C (typisch)                               |
| **Einheit**       | °C                                                  |
| **Umrechnung**    | `rawValue / 10`                                     |
| **Rundung**       | 1 Dezimalstelle (0.1°C)                             |
| **Normalbetrieb** | 35-50°C (Heizbetrieb)                               |
| **Verwendung**    | Cooldown-Steuerung (ΔT < 1°C), Leistungsberechnung |

### Kanal 4: Rücklauftemperatur

| Parameter         | Wert                                             |
|-------------------|--------------------------------------------------|
| **Klemme**        | Term 4 (EL3204) - RTD Channel 4                  |
| **Signal-Name**   | `temperatureReturn`                              |
| **Variable**      | `pd.sensor.temperatureReturn`                    |
| **Typ**           | Eingang, Analog (PT1000)                         |
| **Sensor**        | PT1000 am Wärmepumpen-Eingang (Rücklauf)         |
| **Messbereich**   | 0°C - +80°C (typisch)                            |
| **Einheit**       | °C                                               |
| **Umrechnung**    | `rawValue / 10`                                  |
| **Rundung**       | 1 Dezimalstelle (0.1°C)                          |
| **Normalbetrieb** | 25-45°C (Heizbetrieb)                            |
| **Max. Grenze**   | 53°C (Sicherheitsabschaltung)                    |
| **Verwendung**    | Cooldown (ΔT zu Vorlauf), Sicherheitsüberwachung |

---

## 4. Term 5 (EL3102) - Analog Eingang (0-10 V Spannungssensoren)

**Technische Daten:**

- Eingangsspannung: 0-10 V (auch -10V bis +10V möglich)
- Auflösung: 16 Bit
- Genauigkeit: ±0.3% vom Messbereich

### Kanal 1: Druckdifferenz Verdampfer

| Parameter                  | Wert                                       |
|----------------------------|--------------------------------------------|
| **Klemme**                 | Term 5 (EL3102) - Channel 1                |
| **Signal-Name**            | `pressureDifferenceEvaporator`             |
| **Variable**               | `pd.sensor.pressureDifferenceEvaporator`   |
| **Typ**                    | Eingang, Analog (0-10 V)                   |
| **Sensor**                 | Differenzdrucksensor am Verdampfer         |
| **Messbereich**            | 0 - 200 mbar                               |
| **Physikalischer Bereich** | 0 mbar @ 0V, 200 mbar @ 10V               |
| **Einheit**                | mbar (Millibar)                            |
| **Umrechnung**             | `(rawValue / 32767) * 200`                 |
| **Rundung**                | Ganzzahl (1 mbar)                          |
| **Normalbetrieb**          | 10-30 mbar (abhängig von Durchfluss)       |
| **Fehlergrenze**           | > 45 mbar (Einfriergefahr!)                |
| **Verwendung**             | Überwachung Durchflussmenge, Verschmutzung |

### Kanal 2: Nicht belegt

| Parameter  | Wert                        |
|------------|-----------------------------|
| **Klemme** | Term 5 (EL3102) - Channel 2 |
| **Status** | Nicht belegt / Reserve      |

---

## 5. Term 6 (EL1008) - Digital Eingang 24V DC (1. Modul)

**Technische Daten:**

- Eingangsspannung: 24V DC (typ.), 11-30V DC (Bereich)
- Logik: HIGH bei > 15V, LOW bei < 5V
- Eingangsfilter: 3 ms (typisch)

### Kanal 1: Benutzer-Bestätigung (Taster)

| Parameter         | Wert                                                |
|-------------------|-----------------------------------------------------|
| **Klemme**        | Term 6 (EL1008) - Channel 1                         |
| **Signal-Name**   | `userConfirmation`                                  |
| **Variable**      | `pd.sensor.userConfirmation` (AT %I*)               |
| **Typ**           | Eingang, Digital (24V DC)                           |
| **Sensor/Taster** | Bestätigungs-Taster (Schließer)                     |
| **Logik**         | TRUE = Taster gedrückt (24V)                        |
| **Verwendung**    | Quittierung von Fehlern, manuelles Ein-/Ausschalten |
| **Auswertung**    | Rising Edge (R_TRIG) im Programm                    |

### Kanal 2: Boiler-Ladepumpe aktiv

| Parameter       | Wert                                        |
|-----------------|---------------------------------------------|
| **Klemme**      | Term 6 (EL1008) - Channel 2                 |
| **Signal-Name** | `boilerHeatRequest`                         |
| **Variable**    | `pd.sensor.boilerHeatRequest` (AT %I*)      |
| **Typ**         | Eingang, Digital (24V DC)                   |
| **Sensor**      | Rückmeldung Boiler-Ladepumpe (Hilfsschütz)  |
| **Logik**       | TRUE = Boiler-Ladepumpe läuft               |
| **Verwendung**  | **Einschalt-Bedingung** (Boiler-Lade-Logik) |

### Kanal 3: Expansionsventil Alarm

| Parameter       | Wert                                                |
|-----------------|-----------------------------------------------------|
| **Klemme**      | Term 6 (EL1008) - Channel 3                         |
| **Signal-Name** | `alarmExpansionValve`                               |
| **Variable**    | `pd.incident.alarmExpansionValve` (AT %I*)          |
| **Typ**         | Eingang, Digital (24V DC)                           |
| **Sensor**      | Alarm-Ausgang EEV (elektronisches Expansionsventil) |
| **Logik**       | TRUE = Fehler am Expansionsventil                   |
| **Verwendung**  | **Fehler-Erkennung** → ERROR-Zustand                |

### Kanal 4: Photovoltaik-Überschuss

| Parameter       | Wert                                                      |
|-----------------|-----------------------------------------------------------|
| **Klemme**      | Term 6 (EL1008) - Channel 4                               |
| **Signal-Name** | `photovoltaicSurplus`                                     |
| **Variable**    | `pd.sensor.photovoltaicSurplus` (AT %I*)                  |
| **Typ**         | Eingang, Digital (24V DC)                                 |
| **Sensor**      | PV-Überschuss-Signal (z.B. von Wechselrichter)            |
| **Logik**       | TRUE = PV-Überschuss verfügbar                            |
| **Verwendung**  | Erhöhte Zieltemperaturen (40°C / 50°C statt 33°C / 41°C) |

### Kanal 5: Motorschutz Kompressor

| Parameter       | Wert                                            |
|-----------------|-------------------------------------------------|
| **Klemme**      | Term 6 (EL1008) - Channel 5                     |
| **Signal-Name** | `incidentCompressor`                            |
| **Variable**    | `pd.incident.incidentCompressor` (AT %I*)       |
| **Typ**         | Eingang, Digital (24V DC)                       |
| **Sensor**      | Motorschutzschalter Kompressor (Öffner-Kontakt) |
| **Logik**       | TRUE = Motorschutz ausgelöst                    |
| **Verwendung**  | **Fehler-Erkennung** → ERROR-Zustand            |

### Kanal 6: Durchflusswächter Quellwasser

| Parameter        | Wert                                                                                    |
|------------------|-----------------------------------------------------------------------------------------|
| **Klemme**       | Term 6 (EL1008) - Channel 6                                                             |
| **Signal-Name**  | `incidentWaterFlow`                                                                     |
| **Variable**     | `pd.incident.incidentWaterFlow` (AT %I*)                                                |
| **Typ**          | Eingang, Digital (24V DC)                                                               |
| **Sensor**       | Durchflusswächter Quellwasser (mind. 1800 L/h)                                          |
| **Logik**        | TRUE = Durchfluss OK (mind. 1800 L/h)<br>FALSE = Durchfluss zu niedrig                 |
| **Verwendung**   | **Transition** START_WATER_FLOW → RUNNING<br>**Fehler** bei FALSE im RUNNING-Zustand   |
| **Besonderheit** | Invertierte Logik! TRUE = OK                                                            |

### Kanal 7: Hochdruck-Thermostat

| Parameter       | Wert                                        |
|-----------------|---------------------------------------------|
| **Klemme**      | Term 6 (EL1008) - Channel 7                 |
| **Signal-Name** | `incidentHighPressure`                      |
| **Variable**    | `pd.incident.incidentHighPressure` (AT %I*) |
| **Typ**         | Eingang, Digital (24V DC)                   |
| **Sensor**      | Ranco Hochdruck-Thermostat (Öffner)         |
| **Logik**       | TRUE = Hochdruck-Thermostat ausgelöst       |
| **Verwendung**  | **Fehler-Erkennung** → ERROR-Zustand        |

### Kanal 8: Niederdruck-Thermostat

| Parameter       | Wert                                       |
|-----------------|--------------------------------------------|
| **Klemme**      | Term 6 (EL1008) - Channel 8               |
| **Signal-Name** | `incidentLowPressure`                      |
| **Variable**    | `pd.incident.incidentLowPressure` (AT %I*) |
| **Typ**         | Eingang, Digital (24V DC)                  |
| **Sensor**      | Ranco Niederdruck-Thermostat (Öffner)      |
| **Logik**       | TRUE = Niederdruck-Thermostat ausgelöst    |
| **Verwendung**  | **Fehler-Erkennung** → ERROR-Zustand       |

---

## 6. Term 7 (EL3204) - Analog Eingang PT1000 (2. Modul)

**Technische Daten:** (identisch zu Term 4)

### Kanal 1: Überhitzungstemperatur (Heißgas)

| Parameter         | Wert                                    |
|-------------------|-----------------------------------------|
| **Klemme**        | Term 7 (EL3204) - RTD Channel 1         |
| **Signal-Name**   | `temperatureOverheatedGas`              |
| **Variable**      | `pd.sensor.temperatureOverheatedGas`    |
| **Typ**           | Eingang, Analog (PT1000)                |
| **Sensor**        | PT1000 am Kompressor-Austritt (Heißgas) |
| **Messbereich**   | -50°C - +150°C                          |
| **Einheit**       | °C                                      |
| **Umrechnung**    | `rawValue / 10`                         |
| **Rundung**       | 1 Dezimalstelle (0.1°C)                 |
| **Normalbetrieb** | 60-90°C (typisch)                       |
| **Verwendung**    | Überhitzungsgrad-Berechnung, Diagnose   |

### Kanal 2: Pufferspeicher Mitte

| Parameter       | Wert                                           |
|-----------------|------------------------------------------------|
| **Klemme**      | Term 7 (EL3204) - RTD Channel 2                |
| **Signal-Name** | `bufferMiddleTemperature`                      |
| **Variable**    | `pd.sensor.bufferMiddleTemperature`            |
| **Typ**         | Eingang, Analog (PT1000)                       |
| **Sensor**      | PT1000 in Pufferspeicher (mittlere Position)   |
| **Messbereich** | 0°C - +90°C                                    |
| **Einheit**     | °C                                             |
| **Umrechnung**  | `rawValue / 10`                                |
| **Rundung**     | 1 Dezimalstelle (0.1°C)                        |
| **Sollwert**    | 41°C (Standard) / 50°C (PV-Modus)             |
| **Verwendung**  | **Ausschalt-Bedingung** (targetTemperatureOff) |

### Kanal 3: Pufferspeicher Oben

| Parameter       | Wert                                          |
|-----------------|-----------------------------------------------|
| **Klemme**      | Term 7 (EL3204) - RTD Channel 3               |
| **Signal-Name** | `bufferTopTemperature`                        |
| **Variable**    | `pd.sensor.bufferTopTemperature`              |
| **Typ**         | Eingang, Analog (PT1000)                      |
| **Sensor**      | PT1000 in Pufferspeicher (obere Position)     |
| **Messbereich** | 0°C - +90°C                                   |
| **Einheit**     | °C                                            |
| **Umrechnung**  | `rawValue / 10`                               |
| **Rundung**     | 1 Dezimalstelle (0.1°C)                       |
| **Sollwert**    | 33°C (Standard) / 40°C (PV-Modus)            |
| **Verwendung**  | **Einschalt-Bedingung** (targetTemperatureOn) |

### Kanal 4: Boiler-Temperatur

| Parameter       | Wert                                           |
|-----------------|------------------------------------------------|
| **Klemme**      | Term 7 (EL3204) - RTD Channel 4               |
| **Signal-Name** | `boilerTemperature`                            |
| **Variable**    | `pd.sensor.boilerTemperature`                  |
| **Typ**         | Eingang, Analog (PT1000)                       |
| **Sensor**      | PT1000 im Boiler (obere Position)              |
| **Messbereich** | 0°C - +90°C                                    |
| **Einheit**     | °C                                             |
| **Umrechnung**  | `rawValue / 10`                                |
| **Rundung**     | 1 Dezimalstelle (0.1°C)                        |
| **Verwendung**  | Boiler-Lade-Logik (Vergleich mit Puffer + 4°C) |

---

## 7. Term 8 (EL2622) - Relais Ausgang 230V AC (Zwischenkreispumpe)

**Technische Daten:**

- Kontakt: Wechsler (NO / NC)
- Max. Spannung: 230V AC
- Max. Strom: 4A (ohmsche Last)
- Schaltleistung: 920 VA (AC1)

### Kanal 1: Sole-Zwischenkreispumpe Ein/Aus

| Parameter        | Wert                                                     |
|------------------|----------------------------------------------------------|
| **Klemme**       | Term 8 (EL2622) - Channel 1                              |
| **Signal-Name**  | `operatingStateIntermediateCircuitPump`                  |
| **Variable**     | `pd.actor.operatingStateIntermediateCircuitPump`         |
| **Typ**          | Ausgang, Relais (230V AC, 4A)                            |
| **Last**         | Sole-Zwischenkreispumpe (230V AC Motor)                  |
| **Logik**        | TRUE = Pumpe ein                                         |
| **Verwendung**   | Manuell schaltbar im IDLE- und ERROR-Zustand             |
| **Besonderheit** | Schaltung über `toggleIntermediateCircuitPump()` Funktion |

### Kanal 2: Nicht belegt

| Parameter  | Wert                        |
|------------|-----------------------------|
| **Klemme** | Term 8 (EL2622) - Channel 2 |
| **Status** | Nicht belegt / Reserve      |

---

## 8. Term 9 (EL2622) - Relais Ausgang 230V AC (Wasserventil)

**Technische Daten:** (identisch zu Term 8)

### Kanal 1: Ventil Schließen

| Parameter        | Wert                                              |
|------------------|---------------------------------------------------|
| **Klemme**       | Term 9 (EL2622) - Channel 1                       |
| **Signal-Name**  | `closeFlowVentil`                                 |
| **Variable**     | `pd.actor.closeFlowVentil`                        |
| **Typ**          | Ausgang, Relais (230V AC, 4A)                     |
| **Last**         | Bistabiles Ventil - Spule "Schließen" (230V AC)   |
| **Logik**        | Kurzer Impuls (ca. 200-500 ms) zum Schließen      |
| **Verwendung**   | Übergang zu COOLDOWN, OFF, ERROR                  |
| **Besonderheit** | Bistabil = bleibt in Position nach Impuls         |

### Kanal 2: Ventil Öffnen

| Parameter        | Wert                                             |
|------------------|--------------------------------------------------|
| **Klemme**       | Term 9 (EL2622) - Channel 2                      |
| **Signal-Name**  | `openFlowVentil`                                 |
| **Variable**     | `pd.actor.openFlowVentil`                        |
| **Typ**          | Ausgang, Relais (230V AC, 4A)                    |
| **Last**         | Bistabiles Ventil - Spule "Öffnen" (230V AC)     |
| **Logik**        | Kurzer Impuls (ca. 200-500 ms) zum Öffnen        |
| **Verwendung**   | Übergang IDLE → START_WATER_FLOW                 |
| **Besonderheit** | Bistabil = bleibt in Position nach Impuls        |

---

## 9. Term 18 (EL1008) - Digital Eingang 24V DC (2. Modul)

**Technische Daten:** (identisch zu Term 6)

### Kanal 1: Durchflusswächter Zwischenkreis

| Parameter        | Wert                                                                                         |
|------------------|----------------------------------------------------------------------------------------------|
| **Klemme**       | Term 18 (EL1008) - Channel 1                                                                 |
| **Signal-Name**  | `incidentFlowIntermediateCircuit`                                                            |
| **Variable**     | `pd.incident.incidentFlowIntermediateCircuit` (AT %I*)                                       |
| **Typ**          | Eingang, Digital (24V DC)                                                                    |
| **Sensor**       | Durchflusswächter Sole-Zwischenkreis (Verdampfer)                                            |
| **Logik**        | TRUE = Durchfluss OK<br>FALSE = Durchfluss zu niedrig                                        |
| **Verwendung**   | Überwachung Sole-Kreislauf im Verdampfer                                                     |
| **Besonderheit** | Invertierte Logik! TRUE = OK                                                                 |

### Kanäle 2-8: Nicht belegt

| Parameter  | Wert                          |
|------------|-------------------------------|
| **Klemme** | Term 18 (EL1008) - CH 2 bis 8 |
| **Status** | Nicht belegt / Reserve        |

---

## 10. Term 19 (EL3204) - Analog Eingang PT1000 (3. Modul)

**Technische Daten:** (identisch zu Term 4)

### Kanal 1: Quellwassertemperatur Eingang

| Parameter         | Wert                                               |
|-------------------|----------------------------------------------------|
| **Klemme**        | Term 19 (EL3204) - RTD Channel 1                   |
| **Signal-Name**   | `temperatureSourceWaterIn`                         |
| **Variable**      | `pd.sensor.temperatureSourceWaterIn`               |
| **Typ**           | Eingang, Analog (PT1000)                           |
| **Sensor**        | PT1000 am Quellwasser-Eingang (Brunnen/Grundwasser) |
| **Messbereich**   | -10°C - +30°C (typisch)                            |
| **Einheit**       | °C                                                 |
| **Umrechnung**    | `rawValue / 10`                                    |
| **Rundung**       | 1 Dezimalstelle (0.1°C)                            |
| **Normalbetrieb** | 8-12°C (Grundwasser)                               |
| **Verwendung**    | Monitoring Quellwassertemperatur                   |

### Kanal 2: Quellwassertemperatur Ausgang

| Parameter         | Wert                                                 |
|-------------------|------------------------------------------------------|
| **Klemme**        | Term 19 (EL3204) - RTD Channel 2                     |
| **Signal-Name**   | `temperatureSourceWaterOut`                          |
| **Variable**      | `pd.sensor.temperatureSourceWaterOut`                |
| **Typ**           | Eingang, Analog (PT1000)                             |
| **Sensor**        | PT1000 am Quellwasser-Ausgang (nach Verdampfer)      |
| **Messbereich**   | -10°C - +30°C (typisch)                              |
| **Einheit**       | °C                                                   |
| **Umrechnung**    | `rawValue / 10`                                      |
| **Rundung**       | 1 Dezimalstelle (0.1°C)                              |
| **Normalbetrieb** | 5-10°C (nach Wärmeentnahme)                          |
| **Verwendung**    | Monitoring Quellwassertemperatur, ΔT Wärmequelle     |

### Kanäle 3-4: Nicht belegt

| Parameter  | Wert                           |
|------------|--------------------------------|
| **Klemme** | Term 19 (EL3204) - CH 3 und 4  |
| **Status** | Nicht belegt / Reserve         |

---

## Verkabelungsübersicht

### Temperatursensoren (PT1000) - 10 Stück

| Nr | Sensor                        | Term | Modul  | Kanal | Messposition             | Wertebereich         |
|----|-------------------------------|------|--------|-------|--------------------------|----------------------|
| 1  | `temperatureEvaporatingOut`   | 4    | EL3204 | RTD 1 | Verdampfer Ausgang       | 5-12°C (typ.)        |
| 2  | `temperatureEvaporatingIn`    | 4    | EL3204 | RTD 2 | Verdampfer Eingang       | 8-15°C (typ.)        |
| 3  | `temperatureFlow`             | 4    | EL3204 | RTD 3 | Wärmepumpe Vorlauf       | 35-50°C              |
| 4  | `temperatureReturn`           | 4    | EL3204 | RTD 4 | Wärmepumpe Rücklauf      | 25-45°C (max 53°C)   |
| 5  | `temperatureOverheatedGas`    | 7    | EL3204 | RTD 1 | Kompressor Heißgas       | 60-90°C              |
| 6  | `bufferMiddleTemperature`     | 7    | EL3204 | RTD 2 | Pufferspeicher Mitte     | **Ausschalt-Sensor** |
| 7  | `bufferTopTemperature`        | 7    | EL3204 | RTD 3 | Pufferspeicher Oben      | **Einschalt-Sensor** |
| 8  | `boilerTemperature`           | 7    | EL3204 | RTD 4 | Boiler Oben              | 40-60°C              |
| 9  | `temperatureSourceWaterIn`    | 19   | EL3204 | RTD 1 | Quellwasser Eingang      | 8-12°C (Grundwasser) |
| 10 | `temperatureSourceWaterOut`   | 19   | EL3204 | RTD 2 | Quellwasser Ausgang      | 5-10°C (nach WP)     |

**Anschluss PT1000:** 3-Leiter empfohlen (rot-weiß-weiß)

- Rot: Speiseleitung (+)
- Weiß 1: Messleitung
- Weiß 2: Kompensationsleitung (-)

### Drucksensoren (4-20 mA) - 2 Stück

| Nr | Sensor         | Term | Modul  | Kanal | Messposition           | Wertebereich              |
|----|----------------|------|--------|-------|------------------------|---------------------------|
| 1  | `pressureHigh` | 2    | EL3152 | CH 1  | Hochdruck Kältekreis   | 0-18 bar (8-14 bar typ.)  |
| 2  | `pressureLow`  | 2    | EL3152 | CH 2  | Niederdruck Kältekreis | -0.5-7 bar (2-4 bar typ.) |

**Anschluss 4-20mA:** 2-Leiter (+ Speiseleitung 24V DC, - Messleitung/Signal-GND)

### Spannungssensor (0-10 V) - 1 Stück

| Nr | Sensor                         | Term | Modul  | Kanal | Messposition              | Wertebereich            |
|----|--------------------------------|------|--------|-------|---------------------------|-------------------------|
| 1  | `pressureDifferenceEvaporator` | 5    | EL3102 | CH 1  | Differenzdruck Verdampfer | 0-200 mbar (10-30 typ.) |

**Anschluss 0-10V:** 3-Leiter (+ Speiseleitung 24V DC, Signal 0-10V, - Signal-GND)

### Digitale Eingänge (24V DC) - 9 Stück

| Nr | Signal                          | Term | Modul  | Kanal | Funktion                      | Logik              |
|----|---------------------------------|------|--------|-------|-------------------------------|--------------------|
| 1  | `userConfirmation`              | 6    | EL1008 | CH 1  | Bestätigungs-Taster           | TRUE = gedrückt    |
| 2  | `boilerHeatRequest`             | 6    | EL1008 | CH 2  | Boiler-Ladepumpe aktiv        | TRUE = Pumpe läuft |
| 3  | `alarmExpansionValve`           | 6    | EL1008 | CH 3  | EEV Alarm                     | TRUE = Fehler      |
| 4  | `photovoltaicSurplus`           | 6    | EL1008 | CH 4  | PV-Überschuss                 | TRUE = Überschuss  |
| 5  | `incidentCompressor`            | 6    | EL1008 | CH 5  | Motorschutz Kompressor        | TRUE = ausgelöst   |
| 6  | `incidentWaterFlow`             | 6    | EL1008 | CH 6  | Durchfluss Quellwasser OK     | TRUE = OK (!)      |
| 7  | `incidentHighPressure`          | 6    | EL1008 | CH 7  | Hochdruck-Thermostat          | TRUE = ausgelöst   |
| 8  | `incidentLowPressure`           | 6    | EL1008 | CH 8  | Niederdruck-Thermostat        | TRUE = ausgelöst   |
| 9  | `incidentFlowIntermediateCircuit` | 18 | EL1008 | CH 1  | Durchfluss Zwischenkreis OK   | TRUE = OK (!)      |

**Anschluss:** Schließer-Kontakte gegen 24V DC

### Digitale Ausgänge (24V DC) - 3 Stück

| Nr | Signal                     | Term | Modul  | Kanal | Last              | Logik      |
|----|----------------------------|------|--------|-------|-------------------|------------|
| 1  | `operatingStateCompressor` | 3    | EL2004 | CH 1  | Kompressor-Schütz | TRUE = Ein |
| 2  | `operatingStateWaterPump`  | 3    | EL2004 | CH 2  | Pumpen-Schütz     | TRUE = Ein |
| 3  | `operatingStateLed`        | 3    | EL2004 | CH 3  | Betriebs-LED      | TRUE = Ein |

**Anschluss:** Schütz-Spulen oder LED mit Vorwiderstand

### Relais-Ausgänge (230V AC) - 3 Stück

| Nr | Signal                              | Term | Modul  | Kanal | Last                           | Logik              |
|----|-------------------------------------|------|--------|-------|--------------------------------|--------------------|
| 1  | `operatingStateIntermediateCircuitPump` | 8 | EL2622 | CH 1  | Sole-Zwischenkreispumpe (230V) | TRUE = Ein         |
| 2  | `closeFlowVentil`                   | 9    | EL2622 | CH 1  | Ventil Schließen (230V)        | Impuls = Schließen |
| 3  | `openFlowVentil`                    | 9    | EL2622 | CH 2  | Ventil Öffnen (230V)           | Impuls = Öffnen    |

**Anschluss:** 230V AC Last direkt an Relaiskontakt

---

## Wichtige Grenzwerte und Schwellen

### Temperaturen

| Parameter                     | Normal  | Warning | Error            | Aktion                 |
|-------------------------------|---------|---------|------------------|------------------------|
| **temperatureEvaporatingOut** | 5-12°C  | < 5°C   | < 2.3°C          | ERROR → Stopp          |
| **temperatureReturn**         | 25-45°C | 45-53°C | > 53°C           | Sicherheitsabschaltung |
| **bufferTopTemperature**      | -       | -       | < 33°C (40°C PV) | Einschalten            |
| **bufferMiddleTemperature**   | -       | -       | ≥ 41°C (50°C PV) | Ausschalten            |

### Drücke

| Parameter                        | Normal     | Warning    | Error      | Aktion        |
|----------------------------------|------------|------------|------------|---------------|
| **pressureHigh**                 | 8-14 bar   | > 16 bar   | Thermostat | ERROR → Stopp |
| **pressureLow**                  | 2-4 bar    | > 6 bar    | Thermostat | ERROR → Stopp |
| **pressureDifferenceEvaporator** | 10-30 mbar | 30-45 mbar | > 45 mbar  | ERROR → Stopp |

### Durchfluss

| Parameter                          | Bedingung | Aktion                              |
|------------------------------------|-----------|-------------------------------------|
| **incidentWaterFlow**              | TRUE      | Quellwasser OK (≥ 1800 L/h)         |
| **incidentWaterFlow**              | FALSE     | Quellwasser zu niedrig → ERROR      |
| **incidentFlowIntermediateCircuit** | TRUE     | Zwischenkreis OK                    |
| **incidentFlowIntermediateCircuit** | FALSE    | Zwischenkreis zu niedrig → ERROR    |

---

## Kalibrierung und Umrechnung

### PT1000 Temperatursensoren

Das EL3204 Modul liefert die Temperatur als **Integer-Wert in Zehntel-Grad**:

```
Temperatur [°C] = rawValue / 10
```

**Beispiel:** rawValue = 235 → 23.5°C

### 4-20 mA Drucksensoren

Das EL3152 Modul liefert den Stromwert als **16-Bit Integer** (0x0000-0x7FFF = 0-32767):

```
Physikalischer Wert = ((rawMax - rawMin) * rawVal) / 32767 + rawMin
```

**Hochdruck (0-18 bar):** `pressureHigh = rawValue / 32767 * 18`

**Niederdruck (-0.5-7 bar):** `pressureLow = rawValue / 32767 * 7.5 - 0.5`

### 0-10 V Differenzdrucksensor

Das EL3102 Modul liefert die Spannung als **16-Bit Integer** (0-32767):

```
pressureDifferenceEvaporator [mbar] = (rawValue / 32767) * 200
```

---

## Wartung und Kalibrierung

### Regelmäßige Prüfung (jährlich)

| Komponente                  | Prüfung                 | Sollwert               |
|-----------------------------|-------------------------|------------------------|
| PT1000 Sensoren (alle 10)   | Vergleichsmessung       | ±0.5°C Abweichung max. |
| Drucksensoren               | Nullpunkt-Kalibrierung  | 4 mA bei 0 bar         |
| Differenzdrucksensor        | Reinigung Verdampfer    | < 20 mbar bei Nennlast |
| Durchflusswächter Quellwasser | Funktion prüfen       | Schaltet bei 1800 L/h  |
| Durchflusswächter Zwischenkreis | Funktion prüfen    | Schaltpunkt prüfen     |
| Wasserventil                | Öffnen/Schließen testen | Frei beweglich         |

### Fehlersuche

| Symptom                        | Mögliche Ursache               | Prüfung                             |
|--------------------------------|--------------------------------|-------------------------------------|
| Sensor liefert 0°C             | PT1000 Kabelbruch              | Widerstand messen (1000Ω @ 0°C)     |
| Sensor liefert konstanten Wert | Sensor defekt oder eingefroren | Sensor tauschen                     |
| Drucksensor zeigt 0 bar        | Stromschleife unterbrochen     | 4-20 mA messen                      |
| Ventil öffnet nicht            | Relais defekt oder Spule       | Spannung an Ventil messen (230V AC) |
| Pumpe schaltet nicht           | EL2622 Relais prüfen           | Spannung am Relaisausgang messen    |

---

## Schaltplan-Referenzen

**Siehe auch:**

- `Heatpump/DUTs/ST_Sensor.TcDUT` - Sensor-Datenstruktur
- `Heatpump/DUTs/ST_Actor.TcDUT` - Aktor-Datenstruktur
- `Heatpump/DUTs/ST_Incident.TcDUT` - Störungs-Datenstruktur
- `Heatpump/DUTs/ST_SensorRaw.TcDUT` - Rohdaten-Struktur mit EtherCAT-Modulzuordnung
- `Heatpump/POUs/FB_Processdata.TcPOU` - Sensor-Umrechnung (Code)
- `doc/Zuordnungen.xml` - TwinCAT I/O-Verknüpfungen (maßgeblich!)

---

## Änderungshistorie

| Version | Datum      | Autor       | Änderung                                                             |
|---------|------------|-------------|----------------------------------------------------------------------|
| 1.0     | 2025-01-24 | Claude Code | Initiale Dokumentation erstellt                                      |
| 1.1     | 2026-03-06 | Claude Code | Term 8 EL2622 (Zwischenkreispumpe), Term 18 EL1008 (Zwischenkreis-Durchfluss), Term 19 EL3204 (Quellwasser-Temperaturen) hinzugefügt; incidentFlow → incidentWaterFlow umbenannt |
