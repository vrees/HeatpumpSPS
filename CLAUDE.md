# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TwinCAT 3 PLC project for controlling a water/water heat pump system. The system runs on a Beckhoff CX5110-0115-9020 embedded IPC with TwinCAT Runtime 3.1 Build 4024.62.

**Target Hardware:**
- Controller: Beckhoff CX5110-0115-9020 (ARM7, Windows CE 7)
- Network: Static IP 192.168.178.14 (subnet 255.255.255.0, gateway 192.168.178.1)
- HMI Dashboard: http://192.168.178.14:1010/ (credentials: __SystemAdministrator/isdn98)
- Device Manager: https://192.168.178.14:1020/Config (case-sensitive, Administrator/1)

**Key Components:**
- 8x PT1000 temperature sensors (evaporator, flow, return, buffer, boiler)
- 2x 4-20mA pressure sensors (high/low refrigerant pressure)
- 1x 0-10V pressure differential sensor (evaporator)
- 8x digital inputs (alarms, status, user inputs)
- 5x outputs (3x digital, 2x 230V AC relays)

## Project Structure

**Main Solution:** `HeatpumpSPS.sln`
- **HeatpumpSPS** (.tsproj) - TwinCAT 3 project configuration
- **HeatpumpHMI** (.hmiproj) - Web-based HMI interface
- **HeatpumpScope** (.tcmproj) - Measurement/monitoring project

**PLC Project:** `Heatpump/Heatpump.plcproj`
- `POUs/` - Program Organization Units (control logic)
- `DUTs/` - Data Unit Types (20 structures/enums)
- `GVLs/` - Global Variable Lists (configuration, error history)
- `_Config/IO/` - EtherCAT I/O module configuration
- `_Libraries/` - Beckhoff automation libraries

## Architecture

### Control Flow (Entry Point)

**MAIN.TcPOU** is the entry point:
```
MAIN
├── fbProcessdata (FB_Processdata) - Sensor data acquisition & conversion
├── fbStateMachine (FB_StateMachine) - State-based control logic
└── pd (ST_ProcessData) - Shared process data structure
```

### State Machine Design

The heat pump control uses the **State Pattern** with polymorphic state objects implementing the `I_State` interface.

**Six operational states:**
1. **FB_OffState** - Powered off (manual stop)
2. **FB_IdleState** - Ready/standby (enforces 10-min minimum idle time)
3. **FB_StartWaterFlowState** - Starting water pump, waiting for flow
4. **FB_RunningState** - Active heating operation
5. **FB_CoolDownState** - Post-shutdown cooling (water pump runs until ΔT < 1°C)
6. **FB_ErrorState** - Fault condition detected

**State transitions** are managed by `FB_StateMachine.generateEvents()` which evaluates:
- Temperature setpoints (buffer top/middle vs. target on/off)
- Boiler heat request status
- PV surplus mode (enables higher temperature targets)
- Safety limits (max return temp 53°C, pressure limits)
- Minimum idle duration timer (prevents short-cycling)

### Key Data Structures

**ST_ProcessData** - Central data hub containing:
- `sensor: ST_Sensor` - All temperature, pressure, digital inputs
- `actor: ST_Actor` - Compressor, water pump, LED outputs
- `incident: ST_Incident` - Fault flags (alarms, thermostats, flow)
- `messages[]` - Error message array
- `timestamp` - Cycle execution time

**ST_ConfigValues** (persistent) - Tunable parameters:
- `targetTemperatureOn`: 33°C (standard) / 40°C (PV mode)
- `targetTemperatureOff`: 41°C (standard) / 50°C (PV mode)
- `maxReturnTemperature`: 53°C (safety limit)
- `minIdleSec`: 600 seconds (10 minutes between starts)
- `temperatureDiffBacklashOff`: 1.0°C (cooldown threshold)
- Pressure limits: high (16 bar), low (6 bar), diff (45 mbar)

## Control Logic Summary

**Heat pump starts when:**
- Buffer top temp < target ON temp (33°C or 40°C with PV)
- OR boiler heat request active AND buffer not hot enough (≤ boiler temp + 4°C)
- AND return temp ≤ 53°C
- AND currently in IDLE state

**Heat pump stops when:**
- Buffer middle temp ≥ target OFF temp (41°C or 50°C with PV)
- AND no boiler heat request
- OR return temp > 53°C

**After stopping:**
- Water pump continues until flow-return ΔT < 1.0°C (cooldown complete)
- System stays in IDLE for minimum 10 minutes before restart

## Common Development Tasks

### Opening the Project
1. Open `HeatpumpSPS.sln` in TwinCAT XAE Shell (Visual Studio)
2. Solution contains three projects - main PLC is in `Heatpump/Heatpump.plcproj`
3. To connect to remote target: TwinCAT → Choose Target System → 192.168.178.14

### Building the Project
- Build solution: Visual Studio Build menu or F7
- Activate configuration: TwinCAT → Activate Configuration (or Ctrl+F5)
- Note: License file required on target (`TLR_BI_00976901.tclrs`)

### Modifying Control Parameters
1. Edit `Heatpump/DUTs/ST_ConfigValues.TcDUT` to change structure
2. Values are persistent on controller (stored in nvRAM)
3. Modify defaults in `Heatpump/GVLs/GVL.TcGVL` if needed
4. Rebuild and download to controller

### Adding a New State
1. Create `POUs/FB_NewState.TcPOU` implementing `I_State` interface
2. Implement required methods: `HeatIsRequested()`, `ErrorDetected()`, etc.
3. Add `E_STATE.NEW_STATE` enum value in `DUTs/E_State.TcDUT`
4. Declare instance property in `FB_StateMachine`
5. Add transition logic in `FB_StateMachine.generateEvents()`
6. Initialize state with `stateFactory()` call in `FB_StateMachine`

### Modifying State Transition Logic
- Main logic in: `Heatpump/POUs/FB_StateMachine.TcPOU`
- Method: `generateEvents()` (evaluates all sensor inputs and generates state events)
- Helper functions: `calculateTargetTemperatureOn()`, `calculateTargetTemperatureOff()`
- Event handling: Each state's event methods determine valid transitions

### I/O Configuration
- I/O mapping documented in: `doc/Klemmen-Zusammenfassung.md`
- EtherCAT configuration: `_Config/IO/Gerät 1 (EtherCAT)/`
- Modules: EL3152 (4-20mA), EL3204 (PT1000), EL3102 (0-10V), EL1008 (DI), EL2004 (DO), EL2622 (relays)
- To add I/O: Scan devices in TwinCAT, link variables in `FB_Processdata`

### Viewing Logs and Errors
- Error history: `GVL.lastErrorsOccurred[0..6]` (7 most recent events)
- Error types defined in: `DUTs/E_ErrorEvent.TcDUT`
- Logging: `POUs/utils/LogMessage.TcPOU` for timestamped events
- HMI displays current state, temps, pressures, and error list

## Navigation Guide for Code Understanding

**To understand control flow:**
1. `POUs/MAIN.TcPOU` - Entry point (simple orchestration)
2. `POUs/FB_StateMachine.TcPOU` - Core state machine logic
3. `POUs/I_State.TcIO` - State interface contract
4. `POUs/FB_*State.TcPOU` - Individual state implementations

**To understand data structures:**
1. `DUTs/ST_ProcessData.TcDUT` - Top-level data container
2. `DUTs/ST_Sensor.TcDUT` - All input signals
3. `DUTs/ST_Actor.TcDUT` - All output signals
4. `DUTs/ST_ConfigValues.TcDUT` - Tunable parameters

**To understand sensor conversion:**
1. `POUs/FB_Processdata.TcPOU` - Raw I/O to physical units
2. `POUs/utils/ScaleAndRound.TcPOU` - Scaling utility

**To trace a state transition:**
1. Event generated in `FB_StateMachine.generateEvents()`
2. Event method called on current state (e.g., `currentState.HeatIsRequested()`)
3. State method checks validity and calls `stateMachine.transitionTo(newState)`
4. New state becomes active, outputs updated accordingly

## File Sizes and Complexity

- `FB_StateMachine.TcPOU`: ~19KB (most complex, ~400 lines)
- `FB_Processdata.TcPOU`: ~9KB (sensor conversion)
- State implementations: ~2-4KB each
- Total PLC code: ~1,817 lines across all POUs

## Important Notes

- Language: Structured Text (ST) - Pascal-like syntax
- All temperatures in °C, pressures in bar/mbar
- Rising edge triggers (R_TRIG) used for button inputs
- State transitions are event-driven, not polled continuously
- Minimum idle duration prevents compressor short-cycling (critical for longevity)
- Safety limits enforced: max return temp, pressure thresholds, flow detection
- PV surplus mode allows higher storage temperatures for self-consumption optimization