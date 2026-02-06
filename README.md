# Maestro Hydronic

**An open-source ESP32-based zone controller for DIY van and RV hydronic heating systems.**

This project is in early development — I'm sharing the design publicly to gather feedback from the community before building. If you're into hydronic heating, van builds, or ESP32 projects, I'd love your input. Open an issue, start a discussion, or just poke holes in the design.

---

## Complete Project Specification

**Version:** 1.0  
**Date:** February 5, 2026  
**Status:** Hardware Selection Complete / Ready for Prototyping

---

## 1. Project Overview

### 1.1 What This Is

An open-source ESP32-based zone controller for DIY van/RV hydronic heating systems. The controller manages multiple heating zones (domestic hot water, radiant floor, hot air) while treating the diesel hydronic heater as a "black box" — the heater handles combustion safety internally, while this controller handles zone logic, pumps, valves, and user interface.

### 1.2 Why This Exists

Commercial hydronic systems (Rixen, Timberline) charge premium prices for integration that can be replicated with off-the-shelf components. The diesel heaters themselves (Autoterm, Webasto, Espar, HLN) already handle all combustion safety. What's missing is an affordable, flexible zone controller with a modern interface.

### 1.3 Design Philosophy

- **Heater-agnostic:** Works with any diesel hydronic heater via simple relay or optional CANbus
- **Standalone operation:** Functions completely without WiFi or Home Assistant
- **Local-first UI:** Modern touchscreen interface for daily use
- **Distributed architecture:** Main controller with up to 6 mini "zone boards" connected via CAT5/6
- **Open source:** Hardware designs, schematics, and ESPHome configs published for community
- **Fail-safe:** System defaults to safe state if controller fails; heater's internal safety remains intact

---

## 2. System Architecture

### 2.1 High-Level Topology

```
┌───────────────────────────────────────────────────────────────┐
│                      MAIN CONTROLLER                          │
│  ┌─────────────────┐  ┌─────────────┐  ┌──────────────────┐   │
│  │  WT32-SC01 Plus │  │  4-Ch Relay │  │  CAN Transceiver │   │
│  │  (ESP32-S3 +    │  │  Module     │  │  (SN65HVD230)    │   │
│  │   Touchscreen)  │  │             │  │                  │   │
│  └────────┬────────┘  └──────┬──────┘  └────────┬─────────┘   │
│           │                  │                   │            │
│           │ I²C + 1-Wire     │ Local 12V         │ CANbus     │
│           │                  │                   │            │
│  ┌────────▼──────────────────▼───────────────────▼─────────┐  │
│  │                    MAIN CONTROLLER OUTPUTS              │  │
│  │  • Heater Enable (relay OR CANbus)                      │  │
│  │  • Main Circulation Pump                                │  │
│  │  • Loop Temp Sensors (DS18B20 × 3)                      │  │
│  │  • Zone Data Bus (I²C + 1-Wire via CAT5/6)              │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────┬───────────────────────────────────┘
							│
			┌───────────────┼───────────────┐
			│               │               │
		 CAT5/6          CAT5/6          CAT5/6
		(data only)     (data only)     (data only)
			│               │               │
	┌───────▼───────┐ ┌─────▼───────┐ ┌─────▼───────┐
	│   ZONE 1      │ │   ZONE 2    │ │   ZONE 3    │
	│   DHW         │ │   Floor     │ │   Hot Air   │
	│               │ │             │ │             │
	│ • PCF8574     │ │ • PCF8574   │ │ • PCF8574   │
	│ • 2-Ch Relay  │ │ • 2-Ch Relay│ │ • 2-Ch Relay│
	│ • DS18B20     │ │ • DS18B20   │ │ • DS18B20   │
	│ • Buck Conv.  │ │ • Buck Conv.│ │ • Buck Conv.│
	│               │ │             │ │             │
	│ ◄── 12V local │ │ ◄── 12V     │ │ ◄── 12V     │
	└───────────────┘ └─────────────┘ └─────────────┘
```

### 2.2 Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| **CAT5/6 carries data only** | No voltage drop over long runs; pumps get full 12V locally |
| **Local 12V at each zone** | Proper wire gauge for pump loads; safer; standard practice |
| **I²C for zone communication** | Addressable (up to 8 zones); low pin count on ESP32; ESPHome native support |
| **1-Wire for temperature** | Multiple sensors on single wire; each sensor has unique address |
| **Heater as black box** | Heater handles combustion safety; controller only requests heat |
| **Standalone operation** | System works without WiFi/HA; local display for daily use |

---

## 3. Hardware Specifications

### 3.1 Main Controller Components

| Component | Part | Qty | Purpose |
|-----------|------|-----|---------|
| **MCU + Display** | WT32-SC01 Plus | 1 | ESP32-S3 with 3.5" capacitive touchscreen |
| **CAN Transceiver** | SN65HVD230 module | 1 | Heater CANbus communication (optional) |
| **Relay Module** | 4-channel 5V trigger, opto-isolated | 1 | Heater enable, main circ pump, spares |
| **Temp Sensors** | DS18B20 waterproof probe | 3 | Loop supply, loop return, buffer tank |
| **Pull-up Resistors** | 4.7kΩ | 1 pack | 1-Wire bus pull-up |
| **Buck Converter** | 12V→5V 3A | 1 | Powers ESP32 from vehicle 12V |
| **RJ45 Jacks** | Panel-mount keystone | 4 | Zone connections (3 + spare) |
| **Enclosure** | IP65 ABS junction box | 1 | ~200×150×75mm with display cutout |

### 3.2 Zone Board Components (×3)

*Each zone board is identical; build 3 for DHW, Floor, and Hot Air zones*

| Component | Part | Qty | Purpose |
|-----------|------|-----|---------|
| **I²C Expander** | PCF8574 module | 1 (buy 5-pack) | 8-bit GPIO over I²C |
| **Relay Module** | 2-channel 5V trigger | 1 | Zone pump/valve control |
| **Buck Converter** | 12V→5V 3A | 1 | Powers PCF8574 and relay logic |
| **Temp Sensor** | DS18B20 waterproof probe | 1-2 | Zone supply/return temps |
| **RJ45 Breakout** | Screw terminal to RJ45 | 1 (buy 5-pack) | CAT5/6 termination |
| **Enclosure** | Small project box | 1 | Mount near zone equipment |

### 3.3 Interconnect & Prototyping

| Component | Part | Qty | Purpose |
|-----------|------|-----|---------|
| **Data Cable** | CAT5e (25-50ft) | 1 | Data runs to zones |
| **Breadboard** | 830-point | 1-2 | Desktop prototyping |
| **Power Module** | MB102 breadboard supply | 1 | Bench testing 5V/3.3V |
| **Jumper Wires** | M-M and M-F kit | 1 | Prototyping connections |
| **Power Adapter** | 12V 3A barrel jack | 1 | Bench testing |

---

## 4. Wiring Specifications

### 4.1 CAT5/6 Pinout (Data Only)

The CAT5/6 cable carries **signaling only** — no power for pumps or relays.

```
Pin 1: I²C SDA          ─┐
Pin 2: I²C SDA (pair)   ─┘ Blue twisted pair

Pin 3: I²C SCL          ─┐
Pin 4: I²C SCL (pair)   ─┘ Orange twisted pair

Pin 5: 1-Wire Data      ─┐
Pin 6: GND (signal ref) ─┘ Green twisted pair

Pin 7: Interrupt/Status ─┐
Pin 8: Spare            ─┘ Brown twisted pair
```

### 4.2 I²C Addressing

| Address | Zone | Function |
|---------|------|----------|
| 0x20 | Zone 1 | DHW (Domestic Hot Water) |
| 0x21 | Zone 2 | Radiant Floor |
| 0x22 | Zone 3 | Hot Air Fan Coil |
| 0x23 | Zone 4 | Expansion (shower heat recovery) |
| 0x24-0x27 | Reserved | Future expansion |

*PCF8574 base address is 0x20; set A0/A1/A2 jumpers for each zone*

### 4.3 Zone Board Internal Wiring

```
┌────────────────────────────────────────────────────────────┐
│  ZONE BOARD                                                │
│                                                            │
│  CAT5/6 RJ45 (Data In)                                     │
│    Pin 1,2 (SDA) ────────┬──► PCF8574 SDA                  │
│    Pin 3,4 (SCL) ────────┼──► PCF8574 SCL                  │
│    Pin 5 (1-Wire) ───────┼──► DS18B20 Data                 │
│    Pin 6 (GND) ──────────┴──► Signal Ground                │
│                                                            │
│  Local 12V Power Input                                     │
│    [+12V] ───────────────┬──► Buck Converter Input         │
│                          └──► Relay Module COM             │
│    [GND] ────────────────┴──► Common Ground                │
│                                                            │
│  Buck Converter (12V→5V)                                   │
│    5V Out ───────────────┬──► PCF8574 VCC                  │
│                          └──► Relay Module VCC (logic)     │
│                                                            │
│  PCF8574 GPIO Outputs                                      │
│    P0 ───────────────────────► Relay 1 IN (pump)           │
│    P1 ───────────────────────► Relay 2 IN (valve/spare)    │
│    P2-P7 ────────────────────► Unused (expansion)          │
│                                                            │
│  Relay Outputs (switching 12V to loads)                    │
│    Relay 1 NO ───────────────► Zone Pump (+)               │
│    Relay 2 NO ───────────────► Mixing Valve / Spare        │
│    Relay COM ────────────────► 12V Supply                  │
│                                                            │
│  Screw Terminals                                           │
│    [12V IN] [GND] [PUMP] [VALVE] [SENSOR]                  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 4.4 Main Controller GPIO Assignment (WT32-SC01 Plus)

| GPIO | Function | Notes |
|------|----------|-------|
| GPIO21 | I²C SDA | Zone bus master |
| GPIO22 | I²C SCL | Zone bus master |
| GPIO4 | 1-Wire Data | DS18B20 sensors (all on one bus) |
| GPIO5 | Relay 1 | Heater Enable |
| GPIO6 | Relay 2 | Main Circulation Pump |
| GPIO7 | Relay 3 | Spare |
| GPIO8 | Relay 4 | Spare |
| GPIO25 | CAN TX | Heater CANbus (optional) |
| GPIO26 | CAN RX | Heater CANbus (optional) |

---

## 5. Control Logic

### 5.1 Heater Interface

**Option A: Simple Relay Control (Default)**
- Single relay closure = heater ON request
- Works with any heater that has "external thermostat" input
- No status feedback (infer from loop temp changes)

**Option B: CANbus (If Heater Supports)**
- Bidirectional communication
- Read: heater state, faults, runtime, fuel level
- Write: on/off, target temp, power level
- Supported: Autoterm (well-documented), Webasto, Espar

### 5.2 Zone Control Logic

```
FOR each zone:
  IF zone_enabled AND (zone_temp < setpoint - hysteresis):
	zone_calling = TRUE
  ELSE IF zone_temp > setpoint + hysteresis:
	zone_calling = FALSE

IF any zone_calling:
  IF loop_temp >= minimum_useful_temp:
	// Heat available — use it
	FOR each calling zone:
	  activate_zone_pump()
  ELSE:
	// Need heat — request heater
	request_heater_on()
	FOR each calling zone:
	  activate_zone_pump()
ELSE:
  // No demand
  request_heater_off()
  all_zone_pumps_off()
```

### 5.3 Zone Independence

Each zone operates independently based on its own sensor readings. The controller does not enforce priority between zones — thermal distribution is determined by the physical piping and buffer tank sizing.

Example: if the floor temp sensor reads below setpoint, the controller activates the floor pump and requests the heater if loop temperature is insufficient. All zones follow this same pattern independently.

### 5.4 "Free Heat" Sources

The controller monitors loop temperature and uses available heat before requesting diesel:

- **Engine heat:** When driving, engine coolant heats the loop via plate HX
- **Shore power:** 120V immersion heater in buffer tank (when plugged in)
- **Residual heat:** Buffer tank thermal mass

**Logic:** If loop temp is above minimum useful threshold, activate zones WITHOUT requesting diesel heater.

**Note:** The main circulation pump must be able to run independently of the heater. When free heat is available (engine, shore power, residual), the controller enables the main loop pump to distribute that heat even if the diesel heater is not running.

### 5.5 Safety Features

| Feature | Trigger | Action |
|---------|---------|--------|
| **Overtemp Cutoff** | Loop supply > 95°C | Disable heater request; alarm |
| **Watchdog Timer** | ESP32 crash/hang | Hardware reset; safe default state |
| **Flow Verification** | Pump ON but no ΔT after 60s | Fault alarm; possible air lock |
| **Freeze Protection** | Outdoor temp < 2°C | Circulate loop to prevent freezing |
| **Anti-Short-Cycle** | Heater OFF | Minimum 3-minute delay before restart |

---

## 6. User Interface

### 6.1 Display Hardware

**WT32-SC01 Plus** provides:
- 3.5" IPS display (480×320)
- Capacitive touch (responsive, modern feel)
- ESP32-S3 integrated (no separate MCU board)
- USB-C programming

### 6.2 UI Framework

**LVGL (Light and Versatile Graphics Library)** via ESPHome:
- Modern widget toolkit
- Smooth animations
- Touch-optimized
- Active development and community

**Optional:** SquareLine Studio (Mac/Win/Linux) for drag-and-drop UI design, exports to LVGL code.

### 6.3 Screen Layouts

**Home Screen:**
```
┌─────────────────────────────────────────┐
│  🔥 Hydronic System            ⚙️      │
├─────────────────────────────────────────┤
│                                         │
│     Loop: 68°C  [══════════░░░]         │
│                                         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐    │
│  │  DHW    │ │  Floor  │ │   Air   │    │
│  │  45°C ✓ │ │  21°C → │ │   OFF   │    │
│  │ [BOOST] │ │ [▲ 22 ▼]│ │  [ON]   │    │
│  └─────────┘ └─────────┘ └─────────┘    │
│                                         │
│  Heater: RUNNING 🔥    Pump: ON         │
│                                         │
└─────────────────────────────────────────┘
```

**Zone Detail Screen (tap a zone):**
```
┌─────────────────────────────────────────┐
│  ← Radiant Floor                        │
├─────────────────────────────────────────┤
│                                         │
│  Current:     21.2°C                    │
│  Target:      [  ▼  22°C  ▲  ]          │
│                                         │
│  Supply:      38.5°C                    │
│  Return:      34.2°C                    │
│                                         │
│  Pump: ON ████████░░                    │
│                                         │
│  [ ZONE OFF ]      [ SCHEDULE ]         │
│                                         │
└─────────────────────────────────────────┘
```

**Diagnostics Screen:**
```
┌─────────────────────────────────────────┐
│  ← Diagnostics                          │
├─────────────────────────────────────────┤
│                                         │
│  I²C Bus Scan:                          │
│    0x20 ✓  0x21 ✓  0x22 ✓  0x23 ✗      │
│                                         │
│  Sensors:                               │
│    Loop Supply:   68.2°C  ✓            │
│    Loop Return:   64.1°C  ✓            │
│    Buffer Tank:   65.3°C  ✓            │
│    DHW Output:    45.1°C  ✓            │
│    Floor Supply:  38.5°C  ✓            │
│    Floor Return:  FAULT ⚠️              │
│                                         │
│  [ EXPORT LOGS ]    [ RESET FAULTS ]    │
│                                         │
└─────────────────────────────────────────┘
```

---

## 7. Software Architecture

### 7.1 Platform

**ESPHome** running on ESP32-S3:
- Native Home Assistant integration
- YAML-based configuration
- OTA updates
- Large community and documentation

### 7.2 Home Assistant Integration (Absolutely but not required)

- All sensors exposed automatically via ESPHome API
- Zone controls available as switches/climate entities
- Automations can override local control
- Dashboard cards for monitoring

**Critical:** System operates fully standalone. HA enhances but is not required.

---

## 8. Development Phases

### Phase 1: Desktop Proof-of-Concept

**Goal:** Validate hardware communication and basic control logic on bench.

**Steps:**
1. Wire ESP32 + one PCF8574 + relay on breadboard
2. Flash basic ESPHome config
3. Verify I²C communication (address scan)
4. Add DS18B20 sensors, verify 1-Wire reads
5. Control relay via PCF8574 GPIO
6. Add display, test basic LVGL rendering

**Success criteria:** Can read temps and toggle relay from touchscreen.

### Phase 2: Multi-Zone Testing

**Goal:** Prove full architecture with all 3 zones.

**Steps:**
1. Build 3 zone boards (can be breadboard/protoboard)
2. Connect via CAT5/6 (10+ ft runs)
3. Assign unique I²C addresses
4. Implement zone control logic
5. Test priority and interlock behavior
6. Stress test (simultaneous zone calls)

**Success criteria:** All 3 zones respond correctly; no I²C conflicts.

### Phase 3: Integration Testing

**Goal:** Test with actual hydronic components.

**Steps:**
1. Connect to real pumps (bench setup with bucket of water)
2. Test heater relay interface (dry contact)
3. If available: test CANbus with heater
4. Verify safety cutoffs work
5. Long-duration soak test (24+ hours)

**Success criteria:** Reliable operation controlling real loads.

### Phase 4: Van Installation

**Goal:** Final installation in vehicle.

**Steps:**
1. Install main controller in permanent location
2. Run CAT5/6 to zone locations
3. Install zone boards near equipment
4. Connect to actual pumps, valves, heater
5. Commission each zone
6. Document final wiring

**Success criteria:** System operates reliably in vehicle environment.

---

## 10. Open Source Deliverables

### 10.1 Hardware

- [ ] Main controller schematic (KiCad)
- [ ] Zone board schematic (KiCad)
- [ ] Bill of Materials with supplier links
- [ ] Enclosure recommendations / 3D print files
- [ ] Wiring diagrams

### 10.2 Software

- [ ] Complete ESPHome YAML configuration
- [ ] LVGL UI design files (SquareLine export)
- [ ] Home Assistant dashboard examples
- [ ] Calibration and commissioning guide

### 10.3 Documentation

- [ ] This specification document
- [ ] Installation guide with photos
- [ ] Troubleshooting guide
- [ ] Community contribution guidelines

---

*End of specification*
