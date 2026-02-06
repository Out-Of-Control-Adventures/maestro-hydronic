# Software — ESPHome Configurations

This directory will contain the ESPHome YAML configurations for the Maestro controller.

## Planned Contents

- `maestro.yaml` — Main ESPHome configuration
- `zones/` — Zone-specific configuration includes
- `ui/` — LVGL display layout definitions
- `secrets.yaml.example` — Template for WiFi/HA credentials (actual secrets.yaml is gitignored)

## Current Status

Software development begins after Phase 1 hardware validation. See the [Development Phases](../../docs/project-specification.md#9-development-phases) in the project specification.

## Getting Started

```bash
# Install ESPHome
pip install esphome

# Compile (once configs exist)
esphome compile maestro.yaml

# Flash via USB
esphome upload maestro.yaml
```
