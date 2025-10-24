# Wasser/Wasser Wärmepumpe

## EL3152: 4-20mA Eingang

- pressureHigh: Channel 1
- pressureLow: Channel 2

## EL2004: Digital Ausgang

- actor.operatingStateCompressor: Channel 1
- actor.operatingStateLed: Channel 3
- actor.operatingStateWaterPump: Channel 2

## EL3204: Analog Eingang - PT 1000

- temperatureEvaporatingIn: RTD Inputs Channel 2
- temperatureEvaporatingOut: RTD Inputs Channel 1
- temperatureFlow: RTD Inputs Channel 3
- temperatureReturn: RTD Inputs Channel 4

## EL3102: Analog Eingang - 0-10 Volt

- pressureDifferenceEvaporator: Channel 1

## EL1008: Analog Eingang

- incident.alarmExpansionValve: Channel 3
- incident.incidentCompressor: Channel 5
- incident.incidentFlow: Channel 6
- incident.incidentHighPressure: Channel 7
- incident.incidentLowPressure: Channel 8
- boilerHeatRequest: Channel 2
- photovoltaicSurplus: Channel 4
- userConfirmation: Channel 1

## EL3204: Analog Eingang - PT 1000

- boilerTemperature: RTD Inputs Channel 4
- bufferMiddleTemperature: RTD Inputs Channel 2
- bufferTopTemperature: RTD Inputs Channel 3
- temperatureOverheatedGas: RTD Inputs Channel 1

## EL2622: Digital Ausgang -Relais 230V AC, 4A

- closeFlowVentil: Channel 1
- openFlowVentil: Channel 2

# Zusammenfassung

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
