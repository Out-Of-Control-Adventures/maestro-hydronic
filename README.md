# 🎵 Maestro Hydronic

**An open-source ESP32 zone controller for DIY van/RV hydronic heating systems.**

Like a conductor who coordinates the orchestra without playing an instrument, Maestro coordinates your heating zones without touching combustion safety — your diesel heater handles that.

---

## The Problem

Commercial hydronic systems (Rixen, Timberline) charge **$3,000–$6,000+** for integration that wraps around the same diesel heaters anyone can buy. The heaters themselves (Autoterm, Webasto, Espar, HLN) already handle all combustion safety internally. What's missing is an affordable, flexible zone controller with a modern interface.

**Maestro fills that gap.**

## What It Does

- Controls **multiple heating zones** (domestic hot water, radiant floor, hot air) from a single touchscreen
- Treats your diesel heater as a **black box** — Maestro requests heat, the heater handles safety
- Works with **any** diesel hydronic heater via simple relay or optional CANbus
- Runs **100% standalone** — no WiFi, no cloud, no Home Assistant required
- Optionally integrates with **Home Assistant** for dashboards and automation
- Uses **wired zone boards** (I²C over CAT5/6) — no wireless dependencies

**Estimated savings: 30–50% vs commercial integrated systems.**

---

## 🏗️ Project Status

| Milestone | Status |
|-----------|--------|
| System architecture | ✅ Complete |
| Component selection | ✅ Complete |
| Wiring specifications | ✅ Complete |
| Project specification | ✅ Complete |
| Desktop proof-of-concept | 🔄 In Progress |
| Multi-zone testing | ⏳ Next |
| Integration testing | ⏳ Planned |
| Van installation | ⏳ Planned |

---

## 🎨 System Architecture

```
Main Controller (ESP32-S3 + Touchscreen)
├─ Heater Control (relay or CANbus)
├─ Main Circulation Pump
└─ Zone Bus (I²C + 1-Wire over CAT5/6)
    │
    ├─ Zone 1: Domestic Hot Water
    │   ├─ PCF8574 GPIO expander
    │   ├─ 2-ch relay (pump/valve)
    │   └─ DS18B20 temp sensors
    │
    ├─ Zone 2: Radiant Floor
    │   ├─ PCF8574 GPIO expander
    │   ├─ 2-ch relay (pump/mixer)
    │   └─ DS18B20 temp sensors
    │
    └─ Zone 3: Hot Air Fan Coil
        ├─ PCF8574 GPIO expander
        ├─ 2-ch relay (fan control)
        └─ DS18B20 temp sensors
```

**Key design decision:** CAT5/6 carries data only. Each zone gets local 12V power for pumps/relays — no voltage drop, safer, standard practice.

---

## 🔧 Hardware Overview

| Component | Part | Purpose |
|-----------|------|---------|
| **MCU + Display** | WT32-SC01 Plus | ESP32-S3 with 3.5" capacitive touchscreen |
| **Zone Expanders** | PCF8574 modules | I²C GPIO for distributed zone control |
| **Temp Sensors** | DS18B20 (waterproof) | Loop, zone, and buffer tank monitoring |
| **Zone Relays** | 2-ch relay modules | Pump and valve control per zone |
| **CAN Transceiver** | SN65HVD230 | Optional heater CANbus integration |

**Total hardware cost: ~$185–250 CAD** for a complete 3-zone system.

See [`docs/project-specification.md`](docs/project-specification.md) for full hardware specs, wiring diagrams, and bill of materials.

---

## 📁 Repository Structure

```
maestro-hydronic/
├── docs/                          # Documentation
│   ├── project-specification.md   # Complete technical spec
│   └── hydronic-system-design.md  # Overall system architecture
├── hardware/                      # Schematics and PCB designs
│   └── README.md                  # Hardware documentation status
├── software/                      # Firmware and configuration
│   └── esphome/                   # ESPHome YAML configs
│       └── README.md              # Software documentation status
├── CONTRIBUTING.md                # How to get involved
├── LICENSE                        # MIT License
└── README.md                      # You are here
```

---

## 🚀 Getting Started

### For Builders

1. Read the [Project Specification](docs/project-specification.md) to understand the system
2. Check the [Bill of Materials](docs/project-specification.md#11-shopping-list-summary) for parts
3. Follow the [Development Phases](docs/project-specification.md#9-development-phases) to build incrementally

### For Developers

1. Read [CONTRIBUTING.md](CONTRIBUTING.md) for development setup
2. Check [open issues](../../issues) for things to work on
3. The ESPHome configs in `software/esphome/` are the main codebase

---

## 🤝 Contributing

We're actively looking for contributors! This project benefits from people who have:

- **Diesel hydronic heaters** (any brand) to test with
- **ESPHome / ESP32 experience** for firmware development
- **LVGL UI design** skills for the touchscreen interface
- **KiCad experience** for PCB/schematic design
- **Van/RV builds** to validate real-world installation

See [CONTRIBUTING.md](CONTRIBUTING.md) for full details on how to get involved.

---

## 📜 License

This project is licensed under the [MIT License](LICENSE) — open source hardware and software.

---

## ⚠️ Safety Notice

This project controls heating equipment. While the diesel heater handles all combustion safety internally, proper installation and testing is critical.

- Follow all manufacturer guidelines for your heater
- Use appropriate wire gauges and fuses
- Test thoroughly before deployment
- This is not a replacement for professional HVAC work — use at your own risk

---

## 🙏 Acknowledgments

- Inspired by commercial systems from [Rixen](https://rixenheating.com/) and [Timberline](https://www.timberlinevanheat.com/)
- Built on the excellent [ESPHome](https://esphome.io/) framework
- LVGL graphics library for the touchscreen UI

---

**Built with ❤️ for the van life community**
