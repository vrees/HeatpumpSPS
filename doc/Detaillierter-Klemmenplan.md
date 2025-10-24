# Detaillierter Klemmenplan - Wärmepumpen-Steuerung

## Beckhoff CX5110-0115-9020 mit EtherCAT I/O

---

## Zusammenfassung aller I/O 

| Anzahl   | Beschreibung   | Bemerkung   |
|----------|----------------|-------------| 
| 2        | Analog Ein     | 4-20 mA     |
| 8        | Analog Ein     | PT1000      |
| 1        | Analog Ein     | 0-10 V      |
| 8        | Digital Ein    |             |
| 5        | Digital Aus    | Relais      |

- **Gesamt: 24**
- 19 Eingänge
- 5 Relais Ausgänge

## Übersicht der EtherCAT-Module

| Modul      | Typ            | Beschreibung                         | Anzahl Kanäle | Position       |
|------------|----------------|--------------------------------------|---------------|----------------|
| **EL3152** | Analog Input   | 4-20 mA Stromsensoren                | 2             | Bus Position 1 |
| **EL3204** | Analog Input   | PT1000 Temperatursensoren (1. Modul) | 4             | Bus Position 2 |
| **EL3204** | Analog Input   | PT1000 Temperatursensoren (2. Modul) | 4             | Bus Position 3 |
| **EL3102** | Analog Input   | 0-10 V Spannungssensoren             | 2             | Bus Position 4 |
| **EL1008** | Digital Input  | 24V DC Digitaleingänge               | 8             | Bus Position 5 |
| **EL2004** | Digital Output | 24V DC Digitalausgänge               | 4             | Bus Position 6 |
| **EL2622** | Relay Output   | 230V AC Relaisausgänge (4A)          | 2             | Bus Position 7 |

**Gesamt: 24 Signale**

- **19 Eingänge**: 2x 4-20mA, 8x PT1000, 1x 0-10V, 8x Digital
- **5 Ausgänge**: 3x Digital (24V), 2x Relais (230V AC)

---

## 1. EL3152 - Analog Eingang (4-20 mA Stromsensoren)

**Technische Daten:**

- Eingangsstrom: 4-20 mA
- Auflösung: 16 Bit
- Messbereich: 0-20 mA (4-20 mA nutzbar)
- Genauigkeit: ±0.3% vom Messbereich

### Kanal 1: Hochdruck Kältekreis

| Parameter                  | Wert                                  |
|----------------------------|---------------------------------------|
| **Klemme**                 | EL3152 - Channel 1                    |
| **Signal-Name**            | `pressureHigh`                        |
| **Variable**               | `pd.sensor.pressureHigh`              |
| **Typ**                    | Eingang, Analog (4-20 mA)             |
| **Sensor**                 | Drucksensor Hochdruckseite Kältekreis |
| **Messbereich**            | 0 - 18 bar                            |
| **Physikalischer Bereich** | 0 bar @ 4mA, 18 bar @ 20mA            |
| **Einheit**                | bar (absolut)                         |
| **Umrechnung**             | `(rawValue / 32767) * (18 - 0) + 0`   |
| **Rundung**                | 1 Dezimalstelle (0.1 bar)             |
| **Normalbetrieb**          | 8-14 bar (typisch)                    |
| **Warning-Grenze**         | > 16 bar (Software)                   |
| **Fehlergrenze**           | Hardware-Thermostat (Ranco)           |
| **Verwendung**             | Überwachung Hochdruck, COP-Berechnung |

### Kanal 2: Niederdruck Kältekreis

| Parameter                  | Wert                                                       |
|----------------------------|------------------------------------------------------------|
| **Klemme**                 | EL3152 - Channel 2                                         |
| **Signal-Name**            | `pressureLow`                                              |
| **Variable**               | `pd.sensor.pressureLow`                                    |
| **Typ**                    | Eingang, Analog (4-20 mA)                                  |
| **Sensor**                 | Drucksensor Niederdruckseite Kältekreis                    |
| **Messbereich**            | -0.5 - 7 bar                                               |
| **Physikalischer Bereich** | -0.5 bar @ 4mA, 7 bar @ 20mA                               |
| **Einheit**                | bar (absolut)                                              |
| **Umrechnung**             | `(rawValue / 32767) * (7 - (-0.5)) + (-0.5)`               |
| **Rundung**                | 1 Dezimalstelle (0.1 bar)                                  |
| **Normalbetrieb**          | 2-4 bar (typisch)                                          |
| **Warning-Grenze**         | > 6 bar (Software)                                         |
| **Fehlergrenze**           | Hardware-Thermostat (Ranco)                                |
| **Verwendung**             | Überwachung Niederdruck, Verdampfungstemperatur-Berechnung |

---

## 2. EL3204 (1. Modul) - Analog Eingang (PT1000 Temperatursensoren)

**Technische Daten:**

- Sensortyp: PT1000 (Widerstandsthermometer)
- Auflösung: 16 Bit
- Messbereich: -200°C bis +850°C
- Genauigkeit: ±0.1°C @ 25°C
- Anschluss: 2-Leiter oder 3-Leiter (3-Leiter empfohlen)

### Kanal 1: Verdampfer Austrittstemperatur

| Parameter         | Wert                                            |
|-------------------|-------------------------------------------------|
| **Klemme**        | EL3204 (1) - RTD Channel 1                      |
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
| **Klemme**        | EL3204 (1) - RTD Channel 2                     |
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

| Parameter         | Wert                                               |
|-------------------|----------------------------------------------------|
| **Klemme**        | EL3204 (1) - RTD Channel 3                         |
| **Signal-Name**   | `temperatureFlow`                                  |
| **Variable**      | `pd.sensor.temperatureFlow`                        |
| **Typ**           | Eingang, Analog (PT1000)                           |
| **Sensor**        | PT1000 am Wärmepumpen-Ausgang (Vorlauf)            |
| **Messbereich**   | 0°C - +80°C (typisch)                              |
| **Einheit**       | °C                                                 |
| **Umrechnung**    | `rawValue / 10`                                    |
| **Rundung**       | 1 Dezimalstelle (0.1°C)                            |
| **Normalbetrieb** | 35-50°C (Heizbetrieb)                              |
| **Verwendung**    | Cooldown-Steuerung (ΔT < 1°C), Leistungsberechnung |

### Kanal 4: Rücklauftemperatur

| Parameter         | Wert                                             |
|-------------------|--------------------------------------------------|
| **Klemme**        | EL3204 (1) - RTD Channel 4                       |
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

## 3. EL3204 (2. Modul) - Analog Eingang (PT1000 Temperatursensoren)

**Technische Daten:** (identisch zu Modul 1)

### Kanal 1: Überhitzungstemperatur (Heißgas)

| Parameter         | Wert                                    |
|-------------------|-----------------------------------------|
| **Klemme**        | EL3204 (2) - RTD Channel 1              |
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
| **Klemme**      | EL3204 (2) - RTD Channel 2                     |
| **Signal-Name** | `bufferMiddleTemperature`                      |
| **Variable**    | `pd.sensor.bufferMiddleTemperature`            |
| **Typ**         | Eingang, Analog (PT1000)                       |
| **Sensor**      | PT1000 in Pufferspeicher (mittlere Position)   |
| **Messbereich** | 0°C - +90°C                                    |
| **Einheit**     | °C                                             |
| **Umrechnung**  | `rawValue / 10`                                |
| **Rundung**     | 1 Dezimalstelle (0.1°C)                        |
| **Sollwert**    | 41°C (Standard) / 50°C (PV-Modus)              |
| **Verwendung**  | **Ausschalt-Bedingung** (targetTemperatureOff) |

### Kanal 3: Pufferspeicher Oben

| Parameter       | Wert                                          |
|-----------------|-----------------------------------------------|
| **Klemme**      | EL3204 (2) - RTD Channel 3                    |
| **Signal-Name** | `bufferTopTemperature`                        |
| **Variable**    | `pd.sensor.bufferTopTemperature`              |
| **Typ**         | Eingang, Analog (PT1000)                      |
| **Sensor**      | PT1000 in Pufferspeicher (obere Position)     |
| **Messbereich** | 0°C - +90°C                                   |
| **Einheit**     | °C                                            |
| **Umrechnung**  | `rawValue / 10`                               |
| **Rundung**     | 1 Dezimalstelle (0.1°C)                       |
| **Sollwert**    | 33°C (Standard) / 40°C (PV-Modus)             |
| **Verwendung**  | **Einschalt-Bedingung** (targetTemperatureOn) |

### Kanal 4: Boiler-Temperatur

| Parameter       | Wert                                           |
|-----------------|------------------------------------------------|
| **Klemme**      | EL3204 (2) - RTD Channel 4                     |
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

## 4. EL3102 - Analog Eingang (0-10 V Spannungssensoren)

**Technische Daten:**

- Eingangsspannung: 0-10 V (auch -10V bis +10V möglich)
- Auflösung: 16 Bit
- Genauigkeit: ±0.3% vom Messbereich

### Kanal 1: Druckdifferenz Verdampfer

| Parameter                  | Wert                                       |
|----------------------------|--------------------------------------------|
| **Klemme**                 | EL3102 - Channel 1                         |
| **Signal-Name**            | `pressureDifferenceEvaporator`             |
| **Variable**               | `pd.sensor.pressureDifferenceEvaporator`   |
| **Typ**                    | Eingang, Analog (0-10 V)                   |
| **Sensor**                 | Differenzdrucksensor am Verdampfer         |
| **Messbereich**            | 0 - 200 mbar                               |
| **Physikalischer Bereich** | 0 mbar @ 0V, 200 mbar @ 10V                |
| **Einheit**                | mbar (Millibar)                            |
| **Umrechnung**             | `(rawValue / 32767) * (200 - 0) + 0`       |
| **Rundung**                | Ganzzahl (1 mbar)                          |
| **Normalbetrieb**          | 10-30 mbar (abhängig von Durchfluss)       |
| **Fehlergrenze**           | > 45 mbar (Einfriergefahr!)                |
| **Verwendung**             | Überwachung Durchflussmenge, Verschmutzung |

### Kanal 2: Nicht belegt

| Parameter  | Wert                   |
|------------|------------------------|
| **Klemme** | EL3102 - Channel 2     |
| **Status** | Nicht belegt / Reserve |

---

## 5. EL1008 - Digital Eingang (24V DC)

**Technische Daten:**

- Eingangsspannung: 24V DC (typ.), 11-30V DC (Bereich)
- Logik: HIGH bei > 15V, LOW bei < 5V
- Eingangsfilter: 3 ms (typisch)

### Kanal 1: Benutzer-Bestätigung (Taster)

| Parameter         | Wert                                                |
|-------------------|-----------------------------------------------------|
| **Klemme**        | EL1008 - Channel 1                                  |
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
| **Klemme**      | EL1008 - Channel 2                          |
| **Signal-Name** | `boilerHeatRequest`                         |
| **Variable**    | `pd.sensor.boilerHeatRequest` (AT %I*)      |
| **Typ**         | Eingang, Digital (24V DC)                   |
| **Sensor**      | Rückmeldung Boiler-Ladepumpe (Hilfsschütz)  |
| **Logik**       | TRUE = Boiler-Ladepumpe läuft               |
| **Verwendung**  | **Einschalt-Bedingung** (Boiler-Lade-Logik) |

### Kanal 3: Expansionsventil Alarm

| Parameter       | Wert                                                |
|-----------------|-----------------------------------------------------|
| **Klemme**      | EL1008 - Channel 3                                  |
| **Signal-Name** | `alarmExpansionValve`                               |
| **Variable**    | `pd.incident.alarmExpansionValve` (AT %I*)          |
| **Typ**         | Eingang, Digital (24V DC)                           |
| **Sensor**      | Alarm-Ausgang EEV (elektronisches Expansionsventil) |
| **Logik**       | TRUE = Fehler am Expansionsventil                   |
| **Verwendung**  | **Fehler-Erkennung** → ERROR-Zustand                |

### Kanal 4: Photovoltaik-Überschuss

| Parameter       | Wert                                                     |
|-----------------|----------------------------------------------------------|
| **Klemme**      | EL1008 - Channel 4                                       |
| **Signal-Name** | `photovoltaicSurplus`                                    |
| **Variable**    | `pd.sensor.photovoltaicSurplus` (AT %I*)                 |
| **Typ**         | Eingang, Digital (24V DC)                                |
| **Sensor**      | PV-Überschuss-Signal (z.B. von Wechselrichter)           |
| **Logik**       | TRUE = PV-Überschuss verfügbar                           |
| **Verwendung**  | Erhöhte Zieltemperaturen (40°C / 50°C statt 33°C / 41°C) |

### Kanal 5: Motorschutz Kompressor

| Parameter       | Wert                                            |
|-----------------|-------------------------------------------------|
| **Klemme**      | EL1008 - Channel 5                              |
| **Signal-Name** | `incidentCompressor`                            |
| **Variable**    | `pd.incident.incidentCompressor` (AT %I*)       |
| **Typ**         | Eingang, Digital (24V DC)                       |
| **Sensor**      | Motorschutzschalter Kompressor (Öffner-Kontakt) |
| **Logik**       | TRUE = Motorschutz ausgelöst                    |
| **Verwendung**  | **Fehler-Erkennung** → ERROR-Zustand            |

### Kanal 6: Durchflusswächter OK

| Parameter        | Wert                                                                                 |
|------------------|--------------------------------------------------------------------------------------|
| **Klemme**       | EL1008 - Channel 6                                                                   |
| **Signal-Name**  | `incidentFlow`                                                                       |
| **Variable**     | `pd.incident.incidentFlow` (AT %I*)                                                  |
| **Typ**          | Eingang, Digital (24V DC)                                                            |
| **Sensor**       | Durchflusswächter (mind. 1800 L/h)                                                   |
| **Logik**        | TRUE = Durchfluss OK (mind. 1800 L/h)<br>FALSE = Durchfluss zu niedrig               |
| **Verwendung**   | **Transition** START_WATER_FLOW → RUNNING<br>**Fehler** bei FALSE im RUNNING-Zustand |
| **Besonderheit** | Invertierte Logik! TRUE = OK                                                         |

### Kanal 7: Hochdruck-Thermostat

| Parameter       | Wert                                        |
|-----------------|---------------------------------------------|
| **Klemme**      | EL1008 - Channel 7                          |
| **Signal-Name** | `incidentHighPressure`                      |
| **Variable**    | `pd.incident.incidentHighPressure` (AT %I*) |
| **Typ**         | Eingang, Digital (24V DC)                   |
| **Sensor**      | Ranco Hochdruck-Thermostat (Öffner)         |
| **Logik**       | TRUE = Hochdruck-Thermostat ausgelöst       |
| **Verwendung**  | **Fehler-Erkennung** → ERROR-Zustand        |

### Kanal 8: Niederdruck-Thermostat

| Parameter       | Wert                                       |
|-----------------|--------------------------------------------|
| **Klemme**      | EL1008 - Channel 8                         |
| **Signal-Name** | `incidentLowPressure`                      |
| **Variable**    | `pd.incident.incidentLowPressure` (AT %I*) |
| **Typ**         | Eingang, Digital (24V DC)                  |
| **Sensor**      | Ranco Niederdruck-Thermostat (Öffner)      |
| **Logik**       | TRUE = Niederdruck-Thermostat ausgelöst    |
| **Verwendung**  | **Fehler-Erkennung** → ERROR-Zustand       |

---

## 6. EL2004 - Digital Ausgang (24V DC)

**Technische Daten:**

- Ausgangsspannung: 24V DC (von interner Versorgung)
- Max. Strom: 0.5A pro Kanal
- Kurzschlussschutz: Ja
- Übertemperaturschutz: Ja

### Kanal 1: Kompressor Ein/Aus

| Parameter       | Wert                                |
|-----------------|-------------------------------------|
| **Klemme**      | EL2004 - Channel 1                  |
| **Signal-Name** | `operatingStateCompressor`          |
| **Variable**    | `pd.actor.operatingStateCompressor` |
| **Typ**         | Ausgang, Digital (24V DC, 0.5A)     |
| **Last**        | Schütz für Kompressor-Motor (Spule) |
| **Logik**       | TRUE = Kompressor einschalten       |
| **Verwendung**  | Nur in **RUNNING** Zustand = TRUE   |
| **Sicherheit**  | Darf nur laufen wenn Durchfluss OK  |

### Kanal 2: Wasserpumpe Ein/Aus

| Parameter       | Wert                                   |
|-----------------|----------------------------------------|
| **Klemme**      | EL2004 - Channel 2                     |
| **Signal-Name** | `operatingStateWaterPump`              |
| **Variable**    | `pd.actor.operatingStateWaterPump`     |
| **Typ**         | Ausgang, Digital (24V DC, 0.5A)        |
| **Last**        | Schütz für Quellwasser-Pumpe (Spule)   |
| **Logik**       | TRUE = Pumpe einschalten               |
| **Verwendung**  | In **RUNNING** und **COOLDOWN** = TRUE |

### Kanal 3: Status-LED

| Parameter       | Wert                               |
|-----------------|------------------------------------|
| **Klemme**      | EL2004 - Channel 3                 |
| **Signal-Name** | `operatingStateLed`                |
| **Variable**    | `pd.actor.operatingStateLed`       |
| **Typ**         | Ausgang, Digital (24V DC, 0.5A)    |
| **Last**        | Betriebs-LED oder Signallampe      |
| **Logik**       | TRUE = LED ein (Betriebsbereit)    |
| **Verwendung**  | Alle Zustände außer **OFF** = TRUE |

### Kanal 4: Nicht belegt

| Parameter  | Wert                   |
|------------|------------------------|
| **Klemme** | EL2004 - Channel 4     |
| **Status** | Nicht belegt / Reserve |

---

## 7. EL2622 - Relais Ausgang (230V AC, 4A)

**Technische Daten:**

- Kontakt: Wechsler (NO / NC)
- Max. Spannung: 230V AC
- Max. Strom: 4A (ohmsche Last)
- Schaltleistung: 920 VA (AC1)
- Lebensdauer: > 100.000 Schaltzyklen (bei Nennlast)

### Kanal 1: Ventil Schließen

| Parameter        | Wert                                              |
|------------------|---------------------------------------------------|
| **Klemme**       | EL2622 - Channel 1                                |
| **Signal-Name**  | `closeFlowVentil`                                 |
| **Variable**     | Gesteuert durch `switchWaterVentil(bOpen:=FALSE)` |
| **Typ**          | Ausgang, Relais (230V AC, 4A)                     |
| **Last**         | Bistabiles Ventil - Spule "Schließen" (230V AC)   |
| **Logik**        | Kurzer Impuls (ca. 200-500 ms) zum Schließen      |
| **Verwendung**   | Übergang zu COOLDOWN, OFF, ERROR                  |
| **Besonderheit** | Bistabil = bleibt in Position nach Impuls         |

### Kanal 2: Ventil Öffnen

| Parameter        | Wert                                             |
|------------------|--------------------------------------------------|
| **Klemme**       | EL2622 - Channel 2                               |
| **Signal-Name**  | `openFlowVentil`                                 |
| **Variable**     | Gesteuert durch `switchWaterVentil(bOpen:=TRUE)` |
| **Typ**          | Ausgang, Relais (230V AC, 4A)                    |
| **Last**         | Bistabiles Ventil - Spule "Öffnen" (230V AC)     |
| **Logik**        | Kurzer Impuls (ca. 200-500 ms) zum Öffnen        |
| **Verwendung**   | Übergang IDLE → START_WATER_FLOW                 |
| **Besonderheit** | Bistabil = bleibt in Position nach Impuls        |

---

## Verkabelungsübersicht

### Temperatursensoren (PT1000) - 8 Stück

| Nr | Sensor                      | Modul      | Kanal | Messposition         | Wertebereich         |
|----|-----------------------------|------------|-------|----------------------|----------------------|
| 1  | `temperatureEvaporatingOut` | EL3204 (1) | RTD 1 | Verdampfer Ausgang   | 5-12°C (typ.)        |
| 2  | `temperatureEvaporatingIn`  | EL3204 (1) | RTD 2 | Verdampfer Eingang   | 8-15°C (typ.)        |
| 3  | `temperatureFlow`           | EL3204 (1) | RTD 3 | Wärmepumpe Vorlauf   | 35-50°C              |
| 4  | `temperatureReturn`         | EL3204 (1) | RTD 4 | Wärmepumpe Rücklauf  | 25-45°C (max 53°C)   |
| 5  | `temperatureOverheatedGas`  | EL3204 (2) | RTD 1 | Kompressor Heißgas   | 60-90°C              |
| 6  | `bufferMiddleTemperature`   | EL3204 (2) | RTD 2 | Pufferspeicher Mitte | **Ausschalt-Sensor** |
| 7  | `bufferTopTemperature`      | EL3204 (2) | RTD 3 | Pufferspeicher Oben  | **Einschalt-Sensor** |
| 8  | `boilerTemperature`         | EL3204 (2) | RTD 4 | Boiler Oben          | 40-60°C              |

**Anschluss PT1000:** 3-Leiter empfohlen (rot-weiß-weiß)

- Rot: Speiseleitung (+)
- Weiß 1: Messleitung
- Weiß 2: Kompensationsleitung (-)

### Drucksensoren (4-20 mA) - 2 Stück

| Nr | Sensor         | Modul  | Kanal | Messposition           | Wertebereich              |
|----|----------------|--------|-------|------------------------|---------------------------|
| 1  | `pressureHigh` | EL3152 | CH 1  | Hochdruck Kältekreis   | 0-18 bar (8-14 bar typ.)  |
| 2  | `pressureLow`  | EL3152 | CH 2  | Niederdruck Kältekreis | -0.5-7 bar (2-4 bar typ.) |

**Anschluss 4-20mA:** 2-Leiter

-
    + : Speiseleitung (24V DC)
-
    - : Messleitung (Signal-GND)

### Spannungssensor (0-10 V) - 1 Stück

| Nr | Sensor                         | Modul  | Kanal | Messposition              | Wertebereich            |
|----|--------------------------------|--------|-------|---------------------------|-------------------------|
| 1  | `pressureDifferenceEvaporator` | EL3102 | CH 1  | Differenzdruck Verdampfer | 0-200 mbar (10-30 typ.) |

**Anschluss 0-10V:** 3-Leiter

-
    + : Speiseleitung (24V DC)
- Signal: Spannungsausgang (0-10V)
-
    - : Signal-GND

### Digitale Eingänge (24V DC) - 8 Stück

| Nr | Signal                 | Modul  | Kanal | Funktion               | Logik              |
|----|------------------------|--------|-------|------------------------|--------------------|
| 1  | `userConfirmation`     | EL1008 | CH 1  | Bestätigungs-Taster    | TRUE = gedrückt    |
| 2  | `boilerHeatRequest`    | EL1008 | CH 2  | Boiler-Ladepumpe aktiv | TRUE = Pumpe läuft |
| 3  | `alarmExpansionValve`  | EL1008 | CH 3  | EEV Alarm              | TRUE = Fehler      |
| 4  | `photovoltaicSurplus`  | EL1008 | CH 4  | PV-Überschuss          | TRUE = Überschuss  |
| 5  | `incidentCompressor`   | EL1008 | CH 5  | Motorschutz Kompressor | TRUE = ausgelöst   |
| 6  | `incidentFlow`         | EL1008 | CH 6  | Durchfluss OK          | TRUE = OK (!)      |
| 7  | `incidentHighPressure` | EL1008 | CH 7  | Hochdruck-Thermostat   | TRUE = ausgelöst   |
| 8  | `incidentLowPressure`  | EL1008 | CH 8  | Niederdruck-Thermostat | TRUE = ausgelöst   |

**Anschluss:** Schließer-Kontakte gegen 24V DC

### Digitale Ausgänge (24V DC) - 3 Stück

| Nr | Signal                     | Modul  | Kanal | Last              | Logik      |
|----|----------------------------|--------|-------|-------------------|------------|
| 1  | `operatingStateCompressor` | EL2004 | CH 1  | Kompressor-Schütz | TRUE = Ein |
| 2  | `operatingStateWaterPump`  | EL2004 | CH 2  | Pumpen-Schütz     | TRUE = Ein |
| 3  | `operatingStateLed`        | EL2004 | CH 3  | Betriebs-LED      | TRUE = Ein |

**Anschluss:** Schütz-Spulen oder LED mit Vorwiderstand

### Relais-Ausgänge (230V AC) - 2 Stück

| Nr | Signal            | Modul  | Kanal | Last                    | Logik              |
|----|-------------------|--------|-------|-------------------------|--------------------|
| 1  | `closeFlowVentil` | EL2622 | CH 1  | Ventil Schließen (230V) | Impuls = Schließen |
| 2  | `openFlowVentil`  | EL2622 | CH 2  | Ventil Öffnen (230V)    | Impuls = Öffnen    |

**Anschluss:** Bistabiles 230V AC Ventil (2 Spulen)

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

| Parameter        | Bedingung | Aktion                        |
|------------------|-----------|-------------------------------|
| **incidentFlow** | TRUE      | Durchfluss OK (≥ 1800 L/h)    |
| **incidentFlow** | FALSE     | Durchfluss zu niedrig → ERROR |

---

## Kalibrierung und Umrechnung

### PT1000 Temperatursensoren

Das EL3204 Modul liefert die Temperatur als **Integer-Wert in Zehntel-Grad**:

**Umrechnung:**

```
Temperatur [°C] = rawValue / 10
```

**Beispiel:**

- rawValue = 235 → Temperatur = 23.5°C
- rawValue = -50 → Temperatur = -5.0°C

### 4-20 mA Drucksensoren

Das EL3152 Modul liefert den Stromwert als **16-Bit Integer** (0-32767):

**Umrechnung (generisch):**

```
Physikalischer Wert = ((rawValue / 32767) * (max - min)) + min
```

**Hochdruck (0-18 bar):**

```
pressureHigh [bar] = ((rawValue / 32767) * 18) + 0
```

**Niederdruck (-0.5-7 bar):**

```
pressureLow [bar] = ((rawValue / 32767) * 7.5) - 0.5
```

### 0-10 V Differenzdrucksensor

Das EL3102 Modul liefert die Spannung als **16-Bit Integer** (0-32767):

**Umrechnung (0-200 mbar):**

```
pressureDifferenceEvaporator [mbar] = (rawValue / 32767) * 200
```

---

## Wartung und Kalibrierung

### Regelmäßige Prüfung (jährlich)

| Komponente           | Prüfung                 | Sollwert               |
|----------------------|-------------------------|------------------------|
| PT1000 Sensoren      | Vergleichsmessung       | ±0.5°C Abweichung max. |
| Drucksensoren        | Nullpunkt-Kalibrierung  | 4 mA bei 0 bar         |
| Differenzdrucksensor | Reinigung Verdampfer    | < 20 mbar bei Nennlast |
| Durchflusswächter    | Funktion prüfen         | Schaltet bei 1800 L/h  |
| Ventil               | Öffnen/Schließen testen | Frei beweglich         |

### Fehlersuche

| Symptom                        | Mögliche Ursache               | Prüfung                             |
|--------------------------------|--------------------------------|-------------------------------------|
| Sensor liefert 0°C             | PT1000 Kabelbruch              | Widerstand messen (1000Ω @ 0°C)     |
| Sensor liefert konstanten Wert | Sensor defekt oder eingefroren | Sensor tauschen                     |
| Drucksensor zeigt 0 bar        | Stromschleife unterbrochen     | 4-20 mA messen                      |
| Ventil öffnet nicht            | Relais defekt oder Spule       | Spannung an Ventil messen (230V AC) |

---

## Schaltplan-Referenzen

**Siehe auch:**

- `Heatpump/DUTs/ST_Sensor.TcDUT` - Sensor-Datenstruktur
- `Heatpump/DUTs/ST_Actor.TcDUT` - Aktor-Datenstruktur
- `Heatpump/POUs/FB_Processdata.TcPOU` - Sensor-Umrechnung (Code)
- `_Config/IO/Gerät 1 (EtherCAT)/` - EtherCAT Konfiguration

---

## Änderungshistorie

| Version | Datum      | Autor       | Änderung                        |
|---------|------------|-------------|---------------------------------|
| 1.0     | 2025-01-24 | Claude Code | Initiale Dokumentation erstellt |
