# Maestro Water Add-On Direction

**Status:** Direction accepted; detailed design deferred  
**Recorded:** 2026-08-03  
**Parent project:** Maestro Hydronic  
**Implementation gate:** Real pumps, valves, sensors, and protected output hardware must be available for bench testing before hardware or firmware architecture is finalized.

## 1. Decision

Water-system monitoring and automation should become a **Maestro add-on**, rather than a collection of Home Assistant-only automations or a separate proprietary pump controller.

The working name is **Maestro Water**.

Maestro Water should share the local-first philosophy, diagnostics, interface conventions, and reusable ESPHome packages developed for Maestro Hydronic. It does not need to use the same physical ESP32 as the hydronic touchscreen. The likely architecture is a dedicated water-bay controller near the pumps, valves, and analog sensors, with the Maestro touchscreen and Home Assistant acting as user interfaces.

This document records the direction and known starting information. It is not a final schematic, component selection, wiring design, or authorization to control the selected pumps with PWM.

## 2. Why This Belongs in Maestro

The plumbing and hydronic systems overlap at domestic hot water, shower reheating, hot-water priming, temperature sensing, pump control, and fault handling. A shared Maestro platform avoids creating separate control systems with different behavior and diagnostics.

The intended division of responsibility is:

- **Maestro Water controller:** Executes local pump and valve state machines, validates sensor inputs, applies timeouts, and moves outputs to safe states.
- **Maestro touchscreen:** Provides a central local interface for water and hydronic status, manual requests, setpoints, and faults.
- **Home Assistant:** Provides dashboards, history, notifications, and higher-level routines.
- **Physical controls:** Preserve local Prime, Shower Mode, Stop, and manual-service behavior where required.
- **Mechanical protection:** Pump pressure switches, relief protection, thermostatic mixing, fuses, manual valves, and other hard protections remain authoritative.

Loss of Home Assistant, Wi-Fi, or the reComputer must not leave a pump running or a powered valve in an unsafe state.

## 3. Known Starting Context

The current water-system direction includes:

- Two matching **SEAFLO 42-Series SFDP1-030-055-42** 12 V diaphragm pumps:
  - one for general freshwater pressure;
  - one for the recirculating shower loop.
- A hot-water priming return line that sends cool water from the hot line back to the freshwater tank until hot water reaches the remote fixture.
- A recirculating shower with fresh-water, recirculation, fresh-rinse, purge, and dump functions still to be finalized.
- A Home Assistant host on the van reComputer.
- Maestro Hydronic running ESPHome/LVGL on the WT32-SC01 Plus, with standalone operation as a core requirement.
- A preference for a compact, serviceable plumbing installation.

The selected SEAFLO pump rating and portal specifications are planning inputs. Actual running current, starting current, pressure behavior, cycling, acoustic behavior, thermal behavior, and compatibility with external PWM control must be measured before selecting control hardware.

## 4. Proposed Logical Architecture

```text
Pressure / flow / temperature / tank / leak sensors
                         |
                         v
              Maestro Water ESP32
           local state machines and faults
              |                   |
              v                   v
     Protected output stage    Home Assistant
     pumps and powered valves  dashboard/history/alerts
              |
              v
  Fresh pump / recirc pump / primer valve /
  shower routing / purge / dump functions

Optional local data link:
Maestro Water <-> Maestro touchscreen
(CAN, RS-485, or a proven ESPHome node-to-node transport)
```

### Physical direction

A separate controller located near the water equipment is currently preferred because it:

- keeps pressure-transducer and flow-sensor wiring short;
- reduces the number of long analog and I2C runs;
- keeps protected pump and valve outputs close to their loads;
- allows the hydronic display and water I/O hardware to evolve independently;
- avoids making one touchscreen controller a single point of failure.

The final communications method is deferred. Native ESPHome API is sufficient for Home Assistant supervision, but safety-critical operation must stay inside the water controller. A wired link may later be selected for direct Maestro touchscreen status and control.

## 5. Initial Functional Scope

### 5.1 Pressure monitoring

The first implementation should **measure and record pressure**, not immediately attempt variable-speed pressure regulation.

Useful initial observations include:

- freshwater manifold pressure;
- recirculating shower pump discharge pressure;
- pressure before and after the recirculating filter stack;
- pump start/stop thresholds and pressure recovery time;
- pressure decay with all legitimate outlets closed;
- rapid cycling or accumulator problems;
- pump-on with no pressure rise;
- flow with all commanded outlets closed.

A four-channel ADS1115 pressure-monitor arrangement is a strong starting point.

### 5.2 Hot-water priming

The hot-water primer should be a local Maestro Water state machine.

Proposed sequence:

1. Receive a Prime request from a physical control, Maestro touchscreen, or Home Assistant.
2. Reject the request if a conflicting shower, dump, purge, fault, or service mode is active.
3. Validate the primer return temperature sensor and required valve feedback.
4. Open the normally closed primer return valve.
5. Allow the pressure-demand freshwater pump to start naturally, or enable the pump if a future master-control arrangement requires it.
6. Confirm flow and plausible system pressure.
7. Monitor primer return temperature.
8. Close the primer valve after the adjustable ready-temperature threshold has been satisfied for a defined confirmation period.
9. Return to Idle and publish the result.

The controller must close the primer valve and stop any commanded pump on:

- no verified flow;
- implausible or missing pressure;
- failed or disconnected temperature sensor;
- no meaningful temperature rise;
- maximum runtime;
- leak detection;
- low-water inhibit, if required by the final tank-sensor design;
- controller restart, brownout, or watchdog recovery.

The ready temperature, hysteresis, confirmation period, and timeout must be established through bench and installed-system testing.

### 5.3 Recirculating shower

The likely user-facing states are:

- Idle;
- Fresh Wet-Down;
- Recirculating;
- Fresh Rinse;
- End Shower;
- Purge;
- Dump;
- Fault / Service.

The local controller should own all valve sequencing, pump interlocks, timeouts, and safe-state recovery. Home Assistant may request a state or run a convenience routine, but it should not individually orchestrate the safety-critical output sequence.

### 5.4 Water safety and diagnostics

Potential later functions include:

- leak-triggered main-water isolation;
- pump maximum-runtime lockout;
- dry-run or loss-of-prime detection;
- unexpected-flow detection;
- filter differential-pressure warning;
- fresh and Green Tank level monitoring;
- freeze-risk alerts for exposed tanks and lines;
- pump runtime, cycle count, current, and fault history;
- valve-command versus valve-position disagreement.

## 6. Candidate Inputs and Outputs

### Candidate inputs

- Freshwater pressure transducer.
- Recirculating pump discharge pressure transducer.
- Recirculating filter inlet and outlet pressure transducers.
- Freshwater flow meter.
- Recirculating shower flow meter.
- Hot-water primer return temperature sensor.
- Optional hot-water source or heat-exchanger outlet temperature.
- Freshwater tank level.
- Green Tank level.
- Leak sensors at the water bay and shower.
- Fresh and recirculating pump current sensing.
- Powered-valve position feedback or end switches.
- Physical Prime, Shower Mode, Stop, and Service controls.

### Candidate outputs

- Freshwater pump master enable, if the final design needs one.
- Recirculating shower pump enable.
- Hot-water primer return valve.
- Fresh/recirculating shower source-selection valves.
- Shower drain routing valve.
- Spin-down filter purge valve.
- Green Tank dump valve or pump.
- Main water shutoff valve.
- Local fault indicator or buzzer.

Every output requires a defined normal state, power-loss state, manual override, feedback strategy, fuse/protection path, and controller-restart behavior.

## 7. Pressure Regulation Direction

The useful idea from the IRVWPC family is closed-loop pump regulation using a pressure transducer, along with dry-run, long-run, low-flow, and rapid-cycling safeguards.

Maestro Water may eventually reproduce those useful behaviors while providing native integration and combined shower/primer control. That is explicitly a later stage.

Before attempting variable-speed control:

- verify that the exact SEAFLO 42-Series pump motor is suitable for external PWM;
- measure starting and running current;
- characterize pressure, flow, noise, and temperature across the intended duty range;
- select an automotive-appropriate protected motor driver with adequate transient and inrush margin;
- establish flyback, surge, EMI, grounding, and failure behavior;
- preserve a safe local fallback;
- confirm that PWM does not interfere with the pump's internal pressure switch or bypass behavior.

The ESP32 must never drive either pump directly. A relay suitable only for on/off operation must not be treated as a variable-speed motor driver.

## 8. GitHub Starting References

### Directly reusable candidate

- [dferg/esphome-water-pressure](https://github.com/dferg/esphome-water-pressure)
  - ESPHome-based four-channel pressure monitor.
  - ADS1115 acquisition, filtering, voltage diagnostics, PSI calibration, and Home Assistant entities.
  - The pressure-sensor package is the most relevant starting module.
  - MIT licensed.
  - Importing or adapting the pressure module is preferable to adopting its complete display/board configuration.

### Architecture and behavior references

- [samuelolteanu/esphome-water-pressure-switch](https://github.com/samuelolteanu/esphome-water-pressure-switch)
  - Combines pressure, flow, dry-run detection, and pump start/stop thresholds.
  - Useful for studying fault cases.
  - Not suitable for direct adoption: its logic depends too heavily on Home Assistant state and requires substantial cleanup.
  - No reusable license was confirmed during the initial review.

- [phamaralbr/esp32-pressure-pump-controller](https://github.com/phamaralbr/esp32-pressure-pump-controller)
  - Simple local debounce, cooldown, maximum-runtime lockout, and Wi-Fi-independent pump behavior.
  - Useful as a state-machine reference only.
  - It is not an ESPHome/Home Assistant implementation, and no reusable license was confirmed during the initial review.

- [ThisSmartHouse/rvc2hass](https://github.com/ThisSmartHouse/rvc2hass)
  - Useful example of separating protocol definitions, vehicle profiles, entities, and Home Assistant discovery.
  - Its RV-C implementation is not directly applicable unless Maestro later adopts RV-C.

- [dubberville/van-canbus-ha](https://github.com/dubberville/van-canbus-ha)
  - Useful local CAN-to-MQTT/Home Assistant pattern for RV systems.
  - Device-specific Timberline and Volta decoding is not directly reusable for this water controller.

- [EvilForge/ESPHome-ESP32-Camper-RV-Controller](https://github.com/EvilForge/ESPHome-ESP32-Camper-RV-Controller)
  - Demonstrates ESPHome node-to-node packet transport for a camper controller and remote display.
  - Potential reference if Maestro Water later publishes directly to the Maestro touchscreen without depending on Home Assistant.

## 9. Bench-Testing Gate

Detailed design should resume only when representative hardware is available.

Minimum useful bench set:

- both selected SEAFLO 42-Series pumps;
- representative accumulator and plumbing restriction;
- at least one candidate pressure transducer and reference pressure gauge;
- candidate ADS1115 or other ADC input stage;
- representative flow meter;
- candidate primer valve and at least one shower-routing valve;
- representative temperature sensor and wetted mounting method;
- protected relay/output module for on/off testing;
- candidate current-sensing method;
- test reservoir, hoses, relief path, and safe drainage;
- variable-speed motor driver only if PWM testing is intentionally authorized.

### Measurements and failure tests

Record:

- pump running and startup current;
- pressure switch cut-in/cut-out behavior;
- pressure recovery time and cycling frequency;
- flow versus pressure through representative restrictions;
- accumulator effect;
- flow-meter pressure loss;
- pressure-transducer noise and calibration stability;
- primer valve opening/closing time and power behavior;
- hot-water temperature rise and sensor response delay;
- pump and valve temperatures during repeated operation;
- electrical noise effects on the ESP32, ADC, and temperature sensors.

Test safe behavior for:

- Home Assistant unavailable;
- Wi-Fi unavailable;
- controller reboot during every active state;
- temperature sensor disconnected or shorted;
- pressure sensor disconnected or implausible;
- flow sensor stuck at zero or non-zero;
- valve fails to move;
- pump fails to build pressure;
- low supply voltage and brownout;
- watchdog reset;
- maximum-runtime expiry;
- physical Stop or manual override.

## 10. Decisions Intentionally Deferred

Do not select or finalize the following from this direction document alone:

- final Maestro Water MCU or carrier board;
- final communications bus;
- final pressure-transducer range or signal type;
- final flow-meter model;
- final valve models and fail positions;
- final current sensor;
- final pump driver or PWM strategy;
- final temperature thresholds and timeouts;
- final enclosure, connector, or harness;
- final fuse, wire, grounding, and transient-protection details;
- final touchscreen page layout;
- final diagram topology.

Those decisions require real measurements, installation geometry, component manuals, and failure testing.

## 11. Likely Future Repository Shape

A possible future structure is:

```text
software/esphome/
  maestro-hydronic.yaml
  maestro-water.yaml
  packages/
    maestro-core/
    hydronic/
    water/
      pressure.yaml
      flow.yaml
      primer.yaml
      shower-state-machine.yaml
      faults.yaml
```

This is a direction, not a required refactor. The repository should only be reorganized when the next bench-backed design stage begins.

## 12. Restart Point

When work resumes:

1. Confirm the exact hardware on the bench.
2. Pull current component specifications from the Van Build Portal.
3. Create a bench plumbing and electrical test diagram.
4. Establish measurement and failure-test procedures.
5. Characterize the pumps and candidate sensors.
6. Select protected output hardware from measured requirements.
7. Define the Maestro Water state model.
8. Implement a small bench firmware configuration.
9. Test loss-of-HA, loss-of-network, sensor-fault, and reboot behavior.
10. Only then update the production plumbing/electrical diagrams and procurement list.

Until those gates are met, **Maestro Water remains an accepted add-on direction, not a completed control-system design.**
