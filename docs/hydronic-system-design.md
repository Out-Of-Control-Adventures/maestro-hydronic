# DIY Van Hydronic System â€“ Design Summary & Decisions

This document captures the **current agreed architecture, conclusions, and equipment selections** for the DIY van hydronic heating system discussed to date. It is intended as a living design reference for mechanical layout, control strategy, and future implementation.

---

## 1. Design Goals

- Hydronic system suitable for a **Sprinter 170-class van**
- Support **multiple heat sources** (diesel, engine heat, shore power)
- Support **multiple zones** (DHW, radiant floor, hot air)
- Heater treated as a **black box** for combustion and safety
- Zone logic handled externally via **ESP32 / ESPHome / Home Assistant**
- Avoid proprietary lock-in and excessive system markup
- Favor mechanical simplicity and predictable hydronic behavior

---

## 2. Core Architecture Overview

The system is built around a **glycol buffer tank** that acts as:
- Thermal mass
- Hydraulic separator
- Central heat distribution hub

All heat sources and loads interface with this buffer tank either directly or via heat exchangers.

---

## 3. Primary Glycol Loop

### Function
The primary loop maintains hot glycol (nominally **60â€“80â€¯Â°C**) and supplies heat to all zones.

### Components (in loop order)
- Diesel hydronic heater (5â€¯kW class)
- Engine coolant heat exchanger (plate HX)
- Buffer / expansion tank
- Primary circulation pump

### Key Points
- Single continuous loop
- All components have explicit **in/out flow**
- Heater internal pump may run, but **external pump control is required** to support:
  - engine-only heating
  - electric-only heating
  - circulation without firing the burner

HLN has confirmed that **external pumping through the heater when off is acceptable**.

---

## 4. Diesel Hydronic Heater

### Selected Class
- **5â€¯kW hydronic diesel heater**

### Candidates
- EberspÃ¤cher Hydronic S3 (same class used by Rixen / Timberline)
- Autoterm Flow 5
- HLN 5â€¯kW hydronic heater

### Final Position
- Heater remains **heater-agnostic**
- Control interface:
  - simple enable/start signal for baseline operation
  - CAN / UART integration considered a **bonus**, not a requirement

Heater is responsible for:
- combustion safety
- startup / shutdown sequencing
- internal temperature regulation

---

## 5. Buffer / Expansion Tank

### Why a Buffer Tank Is Required
- Reduces heater short-cycling
- Enables fast DHW response
- Allows electric and engine heat without firing diesel
- Provides hydraulic separation for secondary loops

### Target Tank Features
- Rectangular, stainless steel
- Ports for:
  - primary loop in/out
  - radiant floor loop supply/return
  - fill / service
  - pressure relief valve
  - air vent
  - drain
  - optional 120â€¯V immersion heater

### Internal Heat Exchangers (Desired)
- DHW internal coil or plate HX
- Optional secondary internal HX (future use)

Internal HXs are acceptable given low failure rates and efficiency benefits.

---

## 6. Engine Heat Integration

### Strategy
- Plate heat exchanger between engine coolant and house glycol
- HX located near engine bay to minimize engine hose length
- House loop hoses may be longer (acceptable when insulated)

### Behavior
- Engine running â†’ heat naturally transferred
- Engine off â†’ negligible transfer unless pumped
- Optional future feature: **engine preheat using house loop** (via dedicated pump)

---

## 7. Radiant Floor Heating (Zone 2)

### Architecture
- **Secondary loop** sharing the same glycol as the main loop
- Draws hot glycol directly from buffer tank
- Returns glycol back to tank

### Control
- Dedicated floor circulation pump
- **Fixed thermostatic mixing valve** (not electronically controlled)
- Floor loop target supply temperature:
  - nominal **30â€“35â€¯Â°C**

### Key Conclusions
- No glycol-to-glycol HX needed
- No dedicated expansion tank needed
- Pump off = loop hydraulically idle
- Fast warm-up and high efficiency

---

## 8. Domestic Hot Water (Zone 1)

### Architecture
- Potable water passes through **internal DHW heat exchanger** in buffer tank
- Cold water in â†’ heated â†’ thermostatic mixing valve â†’ fixtures

### Control
- No active control required
- DHW temperature set mechanically (â‰ˆ45â€¯Â°C)

### Notes
- Flow rate, not temperature logic, governs DHW performance
- Comparable systems achieve real-world shower performance with this approach

---

## 9. Hot Air Heating (Zone 3)

### Architecture
- Fan-coil heat exchanger connected directly to primary loop
- Always thermally ready when loop is hot

### Selected Class
- Autoterm CHX-style liquid-to-air heat exchanger
- 12â€¯V brushless fan

### Control
- Fan on/off or PWM speed control
- No additional mixing or pumping required

---

## 10. 120â€¯V Electric Heating (Shore Power)

### Purpose
- Maintain temperature
- Pre-warm system
- Reduce diesel runtime

### Implementation
- **Proper immersion heater** installed in buffer tank
- NOT a cartridge heater

### Target Specs
- 750â€“1000â€¯W @ 120â€¯V
- Stainless sheath
- Glycol rated
- Thermostat + manual high-limit cutoff
- GFCI protected circuit

Electric heat is treated as a **priority heat source** when shore power is available.

---

## 11. Control System Philosophy

### Heater Control
- Heater treated as a black box
- Controller only:
  - requests heat (on/off)
  - monitors loop temperature

### Zone Control
- ESP32-based controller
- Manages:
  - zone pumps
  - fans
  - relays
  - temperature sensors

### Logic Summary
- If zone calls for heat:
  - run relevant pump / fan
  - if loop temp sufficient â†’ no heater request
  - if loop temp low â†’ request heater

### UI
- Standalone display desired (Nextion or M5Stack-class)
- HA integration for monitoring and automation
- System must remain functional **without HA**

---

## 12. Key Conclusions

- The **heater itself is not the differentiator** between commercial systems
- Rixen / Timberline value comes from integration, not intelligence
- A buffer tank + simple pumps + mixers achieves the same mechanical behavior
- Smart zone control can be added **without interfering with heater safety**
- Estimated system cost savings vs commercial systems: **30â€“50%**

---

## 13. Open Items / Next Steps

- Finalize buffer tank fabrication approach
- Select specific immersion heater model (Canada-available)
- Confirm DHW real-world flow expectations
- Finalize ESP32 hardware platform and UI choice
- Validate CAN / UART access for selected heater (optional)

---

_End of current design snapshot_

