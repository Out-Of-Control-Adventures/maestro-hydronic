# Maestro Hydronic — ESPHome Firmware

## WT32-SC01 Plus LVGL Touchscreen Controller

This is the ESPHome firmware for the Maestro Hydronic open-source zone controller.
See `esp32_hydronic_controller_project_spec.md` for the full hardware specification.

---

## Quick Start

### 1. Prerequisites
- ESPHome **2025.6.0+** (mipi_spi display driver merged natively)
- WT32-SC01 Plus connected via USB-C
- Python 3.11+

### 2. First Flash
```bash
# Copy secrets template
cp secrets.yaml.template secrets.yaml
# Edit with your values
nano secrets.yaml

# Compile and flash via USB (first time)
esphome run maestro-hydronic.yaml
```

### 3. Subsequent Updates
After first flash, OTA updates work over WiFi:
```bash
esphome run maestro-hydronic.yaml
```

---

## What's Implemented (v0.1.0)

### Display & Touch
- WT32-SC01 Plus display via `mipi_spi` platform (landscape, 480×320)
- FT6336 capacitive touch with correct landscape transform
- PSRAM enabled (octal mode, 80MHz)

### LVGL UI — 5 Pages
1. **Home** — Three zone cards (DHW, Floor, Hot Air) with:
   - Live temperature readout
   - Zone status (Disabled / Satisfied / Heating)
   - Setpoint adjustment (Floor & Air)
   - ON/OFF toggle per zone
   - "Details >" navigation
2. **DHW Detail** — Output temp, setpoint (mechanical TMV note), pump status
3. **Floor Detail** — Supply/return temps, setpoint +/- buttons, pump status
4. **Hot Air Detail** — Fan status, setpoint +/- buttons
5. **Diagnostics** — All sensor readouts, I²C bus status, WiFi, uptime, fault reset

### Always-On Status Bar (top_layer)
- System title
- Heater status with icon (STANDBY / RUNNING with color change)
- Loop supply temperature

### I²C Zone Board Communication
- PCF8574 expanders at 0x20, 0x21, 0x22 (second I²C bus on GPIO21/22)
- Relay outputs for pump + valve per zone

### Temperature Sensors
- 6× DS18B20 on 1-Wire bus (GPIO4)
- Sliding window averaging on primary loop sensors

### Global State
- Zone enable/disable (persisted across reboots)
- Setpoints (persisted across reboots)
- Heater tracking and zone calling flags

### Home Assistant Integration
- All sensors auto-exposed via ESPHome API
- OTA updates
- WiFi with fallback AP

---

## What's NOT Yet Implemented

### Control Logic (Phase 2)
The zone control algorithm from the spec (Section 5.2) is stubbed but not active:
- [ ] Zone calling logic (temp < setpoint − hysteresis → call)
- [ ] Heater request logic (any zone calling + loop below min useful temp)
- [ ] Priority handling (DHW > Floor > Air)
- [ ] "Free heat" detection (use available heat before diesel)
- [ ] Anti-short-cycle timer

### Safety Features (Phase 2)
- [ ] Overtemp cutoff (loop > 95°C → disable heater)
- [ ] Watchdog timer
- [ ] Flow verification (pump ON, no ΔT after 60s → alarm)
- [ ] Freeze protection (outdoor < 2°C → circulate)

### CAN Bus (Phase 3)
- [ ] SN65HVD230 transceiver wiring (GPIO25/26)
- [ ] Autoterm / Webasto protocol implementation
- [ ] Bidirectional heater status reads

### UI Enhancements
- [ ] Settings page (WiFi config, sensor address discovery)
- [ ] Boot splash screen with logo
- [ ] Screen dimming / burn-in protection
- [ ] Swipe navigation between pages

---

## Hardware Notes

### Pin Assignments (per spec)

| GPIO  | Function                    | Notes                          |
|-------|-----------------------------|--------------------------------|
| 47    | Display CLK (SPI octal)     | Managed by mipi_spi            |
| 9,46,3,8,18,17,16,15 | Display data   | 8-bit parallel bus             |
| 45    | Backlight enable            | Strapping pin — warning normal |
| 6     | Touch I²C SDA              | Also used by display touch     |
| 5     | Touch I²C SCL              | Also used by display touch     |
| 7     | Touch interrupt             |                                |
| 21    | Zone I²C SDA               | Second I²C bus for zone boards |
| 22    | Zone I²C SCL               | Second I²C bus for zone boards |
| 4     | 1-Wire data                 | DS18B20 sensors                |
| 16    | Relay 1 — Heater enable     | **VERIFY** — may conflict      |
| 15    | Relay 2 — Main circ pump    | **VERIFY** — may conflict      |
| 25    | CAN TX (future)             |                                |
| 26    | CAN RX (future)             |                                |

> **IMPORTANT:** GPIO 16 and 15 are listed in the spec for relay use BUT
> are also part of the display data bus on the WT32-SC01 Plus. You will
> need to use **different GPIOs** for the relay module or use the GPIO
> expander extension port on the board. Check the WT32-SC01 Plus pinout
> diagram for available GPIOs on the expansion header (typically GPIO10,
> GPIO11, GPIO12, GPIO13, GPIO14 on the extension connector).

### DS18B20 Sensor Discovery
On first boot with sensors connected, check the ESPHome logs for:
```
[D][dallas.sensor:084]: Found sensors:
[D][dallas.sensor:086]:   0x1234567890ABCDEF
```
Then update the `address:` fields in the YAML.

---

## File Structure
```
maestro-hydronic/
├── maestro-hydronic.yaml      # Main ESPHome config (this file)
├── secrets.yaml.template      # Template for WiFi/API secrets
├── secrets.yaml               # YOUR secrets (git-ignored)
└── README.md                  # This file
```

---

## References
- [ESPHome LVGL docs](https://esphome.io/components/lvgl/)
- [ESPHome mipi_spi display](https://esphome.io/components/display/mipi_spi/)
- [WT32-SC01 Plus community thread](https://community.home-assistant.io/t/wt32-sc01-plus-esp32-s3-esp-home/659661)
- Project spec: `esp32_hydronic_controller_project_spec.md`
