# Contributing to Maestro Hydronic

Thanks for your interest in contributing! This project is built by the van life community, for the van life community.

## How to Get Involved

### 🐛 Report Issues
Found a bug or have a problem? [Open an issue](../../issues/new/choose) with as much detail as possible.

### 💡 Suggest Features
Have an idea? Open a feature request issue. We especially value suggestions from people with real-world hydronic experience.

### 🔧 Submit Code
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-improvement`)
3. Make your changes
4. Test on real hardware if possible
5. Submit a pull request

### 📝 Improve Documentation
Documentation improvements are always welcome. If something is unclear, fix it and submit a PR.

---

## Development Setup

### Prerequisites

- **ESPHome** installed ([guide](https://esphome.io/guides/installing_esphome.html))
- **USB-C cable** for WT32-SC01 Plus programming
- **12V power supply** (3A+) for bench testing
- **Hardware components** (see [Bill of Materials](docs/project-specification.md#11-shopping-list-summary))

### Getting Started

```bash
# Clone the repo
git clone https://github.com/Out-Of-Control-Adventures/maestro-hydronic.git
cd maestro-hydronic

# Install ESPHome (if not already installed)
pip install esphome

# Compile the firmware (once configs are ready)
esphome compile software/esphome/maestro.yaml

# Flash to device
esphome upload software/esphome/maestro.yaml
```

### Bench Testing Without Full Hardware

You can test most of the control logic with just:
- An ESP32 dev board (any ESP32 or ESP32-S3)
- One PCF8574 module + relay on a breadboard
- DS18B20 temperature sensor(s)
- 12V power supply

The full WT32-SC01 Plus display is only needed for UI development.

---

## Code Standards

### ESPHome YAML
- Use descriptive names for all entities
- Comment non-obvious configuration choices
- Group related components together
- Test with `esphome config` before committing

### Documentation
- Write for someone who's never seen this project before
- Include photos/diagrams where helpful
- Keep the spec document in sync with actual implementation

---

## Good First Issues

New to the project? These are great starting points:

- **Test with a different heater model** — We need validation across Autoterm, Webasto, Espar, and HLN heaters
- **Improve documentation** — Clarify anything that confused you as a newcomer
- **UI mockups** — Design touchscreen layouts in SquareLine Studio
- **I²C bus testing** — Validate reliability over various CAT5/6 cable lengths
- **Home Assistant dashboard** — Create example dashboard cards for monitoring

---

## Project Architecture

Understanding the code organization:

```
software/esphome/
├── maestro.yaml          # Main ESPHome configuration
├── zones/                # Zone-specific configs (future)
└── ui/                   # LVGL display layouts (future)

hardware/
├── schematics/           # KiCad schematics (future)
├── pcb/                  # PCB designs (future)
└── enclosures/           # 3D print files (future)

docs/
├── project-specification.md   # Complete technical spec
└── hydronic-system-design.md  # System architecture
```

---

## Communication

- **GitHub Issues** — Bug reports, feature requests, questions
- **GitHub Discussions** — General conversation (coming soon)
- **Pull Requests** — Code review and collaboration

---

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).
