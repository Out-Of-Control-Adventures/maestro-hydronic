# ESP32 Hydronic Zone Controller
## Complete Project Specification

**Version:** 1.0  
**Date:** February 5, 2026  
**Status:** Hardware Selection Complete / Ready for Prototyping

---

## 1. Project Overview

### 1.1 What This Is

An open-source ESP32-based zone controller for DIY van/RV hydronic heating systems. The controller manages multiple heating zones (domestic hot water, radiant floor, hot air) while treating the diesel hydronic heater as a "black box" â€” the heater handles combustion safety internally, while this controller handles zone logic, pumps, valves, and user interface.

### 1.2 Why This Exists

Commercial hydronic systems (Rixen, Timberline) charge premium prices ($3,000â€“$6,000+) for integration that can be replicated with off-the-shelf components. The diesel heaters themselves (Autoterm, Webasto, Espar, HLN) already handle all combustion safety. What's missing is an affordable, flexible zone controller with a modern interface.

**Estimated savings:** 30â€“50% vs commercial integrated systems.

### 1.3 Design Philosophy

- **Heater-agnostic:** Works with any diesel hydronic heater via simple relay or optional CANbus
- **Standalone operation:** Functions completely without WiFi or Home Assistant
- **Local-first UI:** Modern touchscreen interface for daily use
- **Open source:** Hardware designs, schematics, and ESPHome configs published for community
- **Fail-safe:** System defaults to safe state if controller fails; heater's internal safety remains intact

---

## 2. System Architecture

### 2.1 High-Level Topology

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                      MAIN CONTROLLER                            â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”‚
â”‚  â”‚  WT32-SC01 Plus â”‚  â”‚  4-Ch Relay â”‚  â”‚  CAN Transceiver â”‚    â”‚
â”‚  â”‚  (ESP32-S3 +    â”‚  â”‚  Module     â”‚  â”‚  (SN65HVD230)    â”‚    â”‚
â”‚  â”‚   Touchscreen)  â”‚  â”‚             â”‚  â”‚                  â”‚    â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â”‚
â”‚           â”‚                  â”‚                   â”‚              â”‚
â”‚           â”‚ IÂ²C + 1-Wire     â”‚ Local 12V         â”‚ CANbus       â”‚
â”‚           â”‚                  â”‚                   â”‚              â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
â”‚  â”‚                    MAIN CONTROLLER OUTPUTS               â”‚   â”‚
â”‚  â”‚  â€¢ Heater Enable (relay OR CANbus)                      â”‚   â”‚
â”‚  â”‚  â€¢ Main Circulation Pump                                 â”‚   â”‚
â”‚  â”‚  â€¢ Loop Temp Sensors (DS18B20 Ã— 3)                      â”‚   â”‚
â”‚  â”‚  â€¢ Zone Data Bus (IÂ²C + 1-Wire via CAT5/6)              â”‚   â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                            â”‚
            â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
            â”‚               â”‚               â”‚
         CAT5/6          CAT5/6          CAT5/6
        (data only)     (data only)     (data only)
            â”‚               â”‚               â”‚
    â”Œâ”€â”€â”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â–¼â”€â”€â”€â”€â”€â”€â”€â”
    â”‚   ZONE 1      â”‚ â”‚   ZONE 2    â”‚ â”‚   ZONE 3    â”‚
    â”‚   DHW         â”‚ â”‚   Floor     â”‚ â”‚   Hot Air   â”‚
    â”‚               â”‚ â”‚             â”‚ â”‚             â”‚
    â”‚ â€¢ PCF8574     â”‚ â”‚ â€¢ PCF8574   â”‚ â”‚ â€¢ PCF8574   â”‚
    â”‚ â€¢ 2-Ch Relay  â”‚ â”‚ â€¢ 2-Ch Relayâ”‚ â”‚ â€¢ 2-Ch Relayâ”‚
    â”‚ â€¢ DS18B20     â”‚ â”‚ â€¢ DS18B20   â”‚ â”‚ â€¢ DS18B20   â”‚
    â”‚ â€¢ Buck Conv.  â”‚ â”‚ â€¢ Buck Conv.â”‚ â”‚ â€¢ Buck Conv.â”‚
    â”‚               â”‚ â”‚             â”‚ â”‚             â”‚
    â”‚ â—„â”€â”€ 12V local â”‚ â”‚ â—„â”€â”€ 12V    â”‚ â”‚ â—„â”€â”€ 12V    â”‚
    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 2.2 Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| **CAT5/6 carries data only** | No voltage drop over long runs; pumps get full 12V locally |
| **Local 12V at each zone** | Proper wire gauge for pump loads; safer; standard practice |
| **IÂ²C for zone communication** | Addressable (up to 8 zones); low pin count on ESP32; ESPHome native support |
| **1-Wire for temperature** | Multiple sensors on single wire; each sensor has unique address |
| **Heater as black box** | Heater handles combustion safety; controller only requests heat |
| **Standalone operation** | System works without WiFi/HA; local display for daily use |

---

## 3. Hardware Specifications

### 3.1 Main Controller Components

| Component | Part | Qty | Est. Cost | Purpose |
|-----------|------|-----|-----------|---------|
| **MCU + Display** | WT32-SC01 Plus | 1 | $22-35 | ESP32-S3 with 3.5" capacitive touchscreen |
| **CAN Transceiver** | SN65HVD230 module | 1 | $3-5 | Heater CANbus communication (optional) |
| **Relay Module** | 4-channel 5V trigger, opto-isolated | 1 | $12-15 | Heater enable, main circ pump, spares |
| **Temp Sensors** | DS18B20 waterproof probe | 3 | $10-15 (5-pack) | Loop supply, loop return, buffer tank |
| **Pull-up Resistors** | 4.7kÎ© | 1 pack | $5 | 1-Wire bus pull-up |
| **Buck Converter** | 12Vâ†’5V 3A | 1 | $5-8 | Powers ESP32 from vehicle 12V |
| **RJ45 Jacks** | Panel-mount keystone | 4 | $8-10 | Zone connections (3 + spare) |
| **Enclosure** | IP65 ABS junction box | 1 | $15-20 | ~200Ã—150Ã—75mm with display cutout |

**Main Controller Subtotal: ~$80-115 CAD**

### 3.2 Zone Board Components (Ã—3)

*Each zone board is identical; build 3 for DHW, Floor, and Hot Air zones*

| Component | Part | Qty | Est. Cost | Purpose |
|-----------|------|-----|-----------|---------|
| **IÂ²C Expander** | PCF8574 module | 1 (buy 5-pack) | $8-12 total | 8-bit GPIO over IÂ²C |
| **Relay Module** | 2-channel 5V trigger | 1 | $4-5 each | Zone pump/valve control |
| **Buck Converter** | 12Vâ†’5V 3A | 1 | $3-4 each | Powers PCF8574 and relay logic |
| **Temp Sensor** | DS18B20 waterproof probe | 1-2 | (from 5-pack) | Zone supply/return temps |
| **RJ45 Breakout** | Screw terminal to RJ45 | 1 (buy 5-pack) | $8-12 total | CAT5/6 termination |
| **Enclosure** | Small project box | 1 | $5-7 each | Mount near zone equipment |

**Per Zone Board: ~$20-25 CAD**  
**All 3 Zone Boards: ~$60-75 CAD**

### 3.3 Interconnect & Prototyping

| Component | Part | Qty | Est. Cost | Purpose |
|-----------|------|-----|-----------|---------|
| **Data Cable** | CAT5e (25-50ft) | 1 | $8-12 | Data runs to zones |
| **Breadboard** | 830-point | 1-2 | $10-15 | Desktop prototyping |
| **Power Module** | MB102 breadboard supply | 1 | $6-8 | Bench testing 5V/3.3V |
| **Jumper Wires** | M-M and M-F kit | 1 | $8-10 | Prototyping connections |
| **Power Adapter** | 12V 3A barrel jack | 1 | $12-15 | Bench testing |

**Interconnect Subtotal: ~$45-60 CAD**

### 3.4 Total Project Cost

| Category | Cost |
|----------|------|
| Main Controller | $80-115 |
| Zone Boards (Ã—3) | $60-75 |
| Interconnect & Prototyping | $45-60 |
| **TOTAL** | **$185-250 CAD** |

---

## 4. Wiring Specifications

### 4.1 CAT5/6 Pinout (Data Only)

The CAT5/6 cable carries **signaling only** â€” no power for pumps or relays.

```
Pin 1: IÂ²C SDA          â”€â”
Pin 2: IÂ²C SDA (pair)   â”€â”˜ Blue twisted pair

Pin 3: IÂ²C SCL          â”€â”
Pin 4: IÂ²C SCL (pair)   â”€â”˜ Orange twisted pair

Pin 5: 1-Wire Data      â”€â”
Pin 6: GND (signal ref) â”€â”˜ Green twisted pair

Pin 7: Interrupt/Status â”€â”
Pin 8: Spare            â”€â”˜ Brown twisted pair
```

### 4.2 IÂ²C Addressing

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
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  ZONE BOARD                                                 â”‚
â”‚                                                             â”‚
â”‚  CAT5/6 RJ45 (Data In)                                     â”‚
â”‚    Pin 1,2 (SDA) â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â–º PCF8574 SDA                  â”‚
â”‚    Pin 3,4 (SCL) â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â–º PCF8574 SCL                  â”‚
â”‚    Pin 5 (1-Wire) â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â–º DS18B20 Data                 â”‚
â”‚    Pin 6 (GND) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â–º Signal Ground                â”‚
â”‚                                                             â”‚
â”‚  Local 12V Power Input                                      â”‚
â”‚    [+12V] â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â–º Buck Converter Input         â”‚
â”‚                          â””â”€â”€â–º Relay Module COM             â”‚
â”‚    [GND] â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â–º Common Ground                â”‚
â”‚                                                             â”‚
â”‚  Buck Converter (12Vâ†’5V)                                   â”‚
â”‚    5V Out â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â–º PCF8574 VCC                  â”‚
â”‚                          â””â”€â”€â–º Relay Module VCC (logic)     â”‚
â”‚                                                             â”‚
â”‚  PCF8574 GPIO Outputs                                       â”‚
â”‚    P0 â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º Relay 1 IN (pump)           â”‚
â”‚    P1 â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º Relay 2 IN (valve/spare)    â”‚
â”‚    P2-P7 â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º Unused (expansion)          â”‚
â”‚                                                             â”‚
â”‚  Relay Outputs (switching 12V to loads)                    â”‚
â”‚    Relay 1 NO â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º Zone Pump (+)               â”‚
â”‚    Relay 2 NO â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º Mixing Valve / Spare        â”‚
â”‚    Relay COM â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–º 12V Supply                  â”‚
â”‚                                                             â”‚
â”‚  Screw Terminals                                            â”‚
â”‚    [12V IN] [GND] [PUMP] [VALVE] [SENSOR]                  â”‚
â”‚                                                             â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 4.4 Main Controller GPIO Assignment (WT32-SC01 Plus)

| GPIO | Function | Notes |
|------|----------|-------|
| GPIO21 | IÂ²C SDA | Zone bus master |
| GPIO22 | IÂ²C SCL | Zone bus master |
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
    // Heat available â€” use it
    FOR each calling zone:
      activate_zone_pump()
  ELSE:
    // Need heat â€” request heater
    request_heater_on()
    FOR each calling zone:
      activate_zone_pump()
ELSE:
  // No demand
  request_heater_off()
  all_zone_pumps_off()
```

### 5.3 Zone Priority

When multiple zones call simultaneously:

1. **DHW (Priority 1)** â€” Always satisfied first; people notice cold water immediately
2. **Radiant Floor (Priority 2)** â€” Slow response; benefits from early activation
3. **Hot Air (Priority 3)** â€” Fast response; can wait

### 5.4 "Free Heat" Sources

The controller monitors loop temperature and uses available heat before requesting diesel:

- **Engine heat:** When driving, engine coolant heats the loop via plate HX
- **Shore power:** 120V immersion heater in buffer tank (when plugged in)
- **Residual heat:** Buffer tank thermal mass

**Logic:** If loop temp is above minimum useful threshold, activate zones WITHOUT requesting diesel heater.

### 5.5 Safety Features

| Feature | Trigger | Action |
|---------|---------|--------|
| **Overtemp Cutoff** | Loop supply > 95Â°C | Disable heater request; alarm |
| **Watchdog Timer** | ESP32 crash/hang | Hardware reset; safe default state |
| **Flow Verification** | Pump ON but no Î”T after 60s | Fault alarm; possible air lock |
| **Freeze Protection** | Outdoor temp < 2Â°C | Circulate loop to prevent freezing |
| **Anti-Short-Cycle** | Heater OFF | Minimum 3-minute delay before restart |

---

## 6. User Interface

### 6.1 Display Hardware

**WT32-SC01 Plus** provides:
- 3.5" IPS display (480Ã—320)
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
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  ðŸ”¥ Hydronic System            âš™ï¸      â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                         â”‚
â”‚     Loop: 68Â°C  [â•â•â•â•â•â•â•â•â•â•â–‘â–‘â–‘]         â”‚
â”‚                                         â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
â”‚  â”‚  DHW    â”‚ â”‚  Floor  â”‚ â”‚   Air   â”‚   â”‚
â”‚  â”‚  45Â°C âœ“ â”‚ â”‚  21Â°C â†’ â”‚ â”‚   OFF   â”‚   â”‚
â”‚  â”‚ [BOOST] â”‚ â”‚ [â–² 22 â–¼]â”‚ â”‚  [ON]   â”‚   â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â”‚                                         â”‚
â”‚  Heater: RUNNING ðŸ”¥    Pump: ON        â”‚
â”‚                                         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Zone Detail Screen (tap a zone):**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â† Radiant Floor                        â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                         â”‚
â”‚  Current:     21.2Â°C                    â”‚
â”‚  Target:      [  â–¼  22Â°C  â–²  ]          â”‚
â”‚                                         â”‚
â”‚  Supply:      38.5Â°C                    â”‚
â”‚  Return:      34.2Â°C                    â”‚
â”‚                                         â”‚
â”‚  Pump: ON â–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–ˆâ–‘â–‘                    â”‚
â”‚                                         â”‚
â”‚  [ ZONE OFF ]      [ SCHEDULE ]         â”‚
â”‚                                         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Diagnostics Screen:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â† Diagnostics                          â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                         â”‚
â”‚  IÂ²C Bus Scan:                          â”‚
â”‚    0x20 âœ“  0x21 âœ“  0x22 âœ“  0x23 âœ—      â”‚
â”‚                                         â”‚
â”‚  Sensors:                               â”‚
â”‚    Loop Supply:   68.2Â°C  âœ“            â”‚
â”‚    Loop Return:   64.1Â°C  âœ“            â”‚
â”‚    Buffer Tank:   65.3Â°C  âœ“            â”‚
â”‚    DHW Output:    45.1Â°C  âœ“            â”‚
â”‚    Floor Supply:  38.5Â°C  âœ“            â”‚
â”‚    Floor Return:  FAULT âš ï¸              â”‚
â”‚                                         â”‚
â”‚  [ EXPORT LOGS ]    [ RESET FAULTS ]    â”‚
â”‚                                         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 7. Software Architecture

### 7.1 Platform

**ESPHome** running on ESP32-S3:
- Native Home Assistant integration
- YAML-based configuration
- OTA updates
- Large community and documentation

### 7.2 Key ESPHome Components

```yaml
# IÂ²C bus for zone boards
i2c:
  sda: GPIO21
  scl: GPIO22
  scan: true

# 1-Wire for temperature sensors
one_wire:
  - pin: GPIO4

# PCF8574 expanders (one per zone)
pcf8574:
  - id: zone1_gpio
    address: 0x20
  - id: zone2_gpio
    address: 0x21
  - id: zone3_gpio
    address: 0x22

# Temperature sensors
sensor:
  - platform: dallas_temp
    address: 0x1234567890ABCDEF
    name: "Loop Supply Temp"
  # ... additional sensors

# Zone relays via PCF8574
switch:
  - platform: gpio
    name: "DHW Pump"
    pin:
      pcf8574: zone1_gpio
      number: 0
      mode: OUTPUT

# CAN bus (optional)
canbus:
  - platform: esp32_can
    tx_pin: GPIO25
    rx_pin: GPIO26
    can_id: 4
    bit_rate: 250kbps

# LVGL display
display:
  - platform: ili9xxx
    # ... display config

lvgl:
  # ... UI widgets
```

### 7.3 Home Assistant Integration (Optional)

- All sensors exposed automatically via ESPHome API
- Zone controls available as switches/climate entities
- Automations can override local control
- Dashboard cards for monitoring

**Critical:** System operates fully standalone. HA enhances but is not required.

---

## 8. Installation Notes

### 8.1 Main Controller Placement

- Mount where display is accessible (eye level preferred)
- Near main electrical panel for 12V supply
- Central location minimizes CAT5/6 run lengths
- IP65 enclosure for moisture protection

### 8.2 Zone Board Placement

- Mount near the equipment each board controls
- Short 12V power runs (< 6 ft ideal)
- CAT5/6 data runs can be longer (tested to 50+ ft)
- Small enclosure at each location

### 8.3 Wiring Best Practices

- **Ferrules** on all stranded wire terminations
- **Fused circuits** for each zone (5-10A depending on pump)
- **Cable glands** for wire entry to enclosures
- **Labeling** on all cables and terminals
- **Service loops** for future maintenance

### 8.4 Vibration and Moisture

- Use locking connectors (not friction-fit Dupont)
- Hot glue or thread-lock on screw terminals
- Conformal coating on PCBs (optional but recommended)
- Desiccant packs in enclosures

---

## 9. Development Phases

### Phase 1: Desktop Proof-of-Concept

**Goal:** Validate hardware communication and basic control logic on bench.

**Steps:**
1. Wire ESP32 + one PCF8574 + relay on breadboard
2. Flash basic ESPHome config
3. Verify IÂ²C communication (address scan)
4. Add DS18B20 sensors, verify 1-Wire reads
5. Control relay via PCF8574 GPIO
6. Add display, test basic LVGL rendering

**Success criteria:** Can read temps and toggle relay from touchscreen.

### Phase 2: Multi-Zone Testing

**Goal:** Prove full architecture with all 3 zones.

**Steps:**
1. Build 3 zone boards (can be breadboard/protoboard)
2. Connect via CAT5/6 (10+ ft runs)
3. Assign unique IÂ²C addresses
4. Implement zone control logic
5. Test priority and interlock behavior
6. Stress test (simultaneous zone calls)

**Success criteria:** All 3 zones respond correctly; no IÂ²C conflicts.

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

## 11. Shopping List Summary

### Amazon.ca Search Terms

**Main Controller:**
- `"WT32-SC01 Plus"` or `"WT32-SC01 Plus ESP32"`
- `"SN65HVD230 CAN module"` or `"CAN bus transceiver module"`
- `"4 channel relay module 5V opto isolated"`
- `"DS18B20 waterproof temperature sensor"` (5-pack)
- `"12V to 5V buck converter 3A"`
- `"RJ45 keystone jack panel mount"`
- `"4.7k ohm resistor"`

**Zone Boards:**
- `"PCF8574 I2C IO expander module"` (5-pack)
- `"2 channel relay module 5V"`
- `"12V to 5V buck converter"` (5-pack)
- `"RJ45 breakout screw terminal"` (5-pack)
- `"small project enclosure"` or `"ABS junction box"`

**Prototyping:**
- `"breadboard jumper wire kit"` or `"ELEGOO breadboard kit"`
- `"MB102 breadboard power supply"`
- `"12V 3A power adapter DC barrel jack"`
- `"Cat5e ethernet cable 50ft"`

---

## 12. Reference Documents

- `diy_van_hydronic_system_design_summary_decisions.md` â€” Hydronic system architecture
- `van_electrical_system_high_level_architecture_decisions.md` â€” Van electrical design
- `Hydronicdesignopt2.pdf` â€” System diagram (note: shows HX for floor; actual design uses direct glycol)

---

## 13. Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-05 | Initial complete specification |

---

*End of specification*
