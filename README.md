<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/images/banner-dark.png">
    <source media="(prefers-color-scheme: light)" srcset="docs/images/banner-light.png">
    <img alt="Arc Lamp banner" src="docs/images/banner-light.png" width="100%">
  </picture>
</p>

<h1 align="center">Arc Lamp</h1>

<p align="center">
  A presence-aware, touch-dimmable LED arc lamp built on the ESP32-S3 — powered entirely over USB-C PD.
</p>

<p align="center">
  <img alt="Build" src="https://img.shields.io/badge/build-passing-brightgreen?style=flat-square">
  <img alt="Firmware" src="https://img.shields.io/badge/firmware-ESPHome-blue?style=flat-square">
  <img alt="Platform" src="https://img.shields.io/badge/platform-ESP32--S3-orange?style=flat-square">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square">
  <img alt="PRs" src="https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square">
</p>

Most desk lamps are either a dumb switch or a cloud-locked "smart" bulb. Arc
Lamp auto-adjusts to presence, dims via touch, and runs its full feature set
completely offline — the ESP32-S3 handles sensing and light logic locally, with
Home Assistant as an optional bonus layer rather than a dependency.

## Table of contents

- [Features](#features)
- [Demo](#demo)
- [Hardware](#hardware)
- [Installation](#installation)
- [Usage](#usage)
- [Wiring](#wiring)
- [Contributing](#contributing)
- [License](#license)

## Features

| Feature | Description |
| :--- | :--- |
| Addressable RGBW arc | WS2815 strip inside a diffused silicone channel for continuous, soft glow |
| Presence-aware | LD2410B mmWave radar detects stationary presence, not just motion — auto on/off |
| Touch dimming | MPR121 capacitive pads set brightness in five steps, no mechanical switch |
| USB-C powered | CH224K negotiates 12V PD from any compatible charger or hub |
| Works fully offline | All automations run on-device; no hub or internet required |
| Home Assistant ready | Auto-discovered over the native ESPHome API when Wi-Fi is available |

## Demo

<p align="center">
  <img alt="Arc Lamp demo" src="docs/images/demo.gif" width="640">
</p>

<sub>Touch dimming and presence auto-on shown above. Replace with your own capture once assembled.</sub>

## Hardware

| Part | Role |
| :---: | :--- |
| ESP32-S3 DevKitC-1 (N16R8) | Main controller |
| WS2815 RGBW LED strip, 12V, 60/m, 2m | Light source |
| Silicone diffuser channel | Housing / light diffusion |
| CH224K | USB-C PD trigger, negotiates 12V |
| LM2596 | Buck converter, 12V → 5V |
| HLK-LD2410B | 24GHz mmWave presence radar |
| MPR121 | 5-pad capacitive touch controller |

Full net-by-net pin map lives in [`docs/wiring.md`](docs/wiring.md).

## Installation

```bash
pip install esphome
git clone https://github.com/yourusername/arc-lamp.git
cd arc-lamp
```

Create a `secrets.yaml` in the project root:

```yaml
wifi_ssid: "your-network"
wifi_password: "your-password"
```

Flash the firmware:

```bash
esphome run arc_lamp.yaml
```

<details>
<summary><b>Standalone / no Wi-Fi at hand</b></summary>

If the lamp can't find the configured network on boot, it opens a local
`ArcLamp Fallback` access point instead of blocking. Every onboard automation
— touch dimming, presence detection, auto-brightness — keeps running with zero
functional loss. Wi-Fi only adds remote control and Home Assistant discovery
on top.

</details>

<details>
<summary><b>Advanced: changing the LED count or GPIO pins</b></summary>

Edit the relevant fields directly in `arc_lamp.yaml`:

```yaml
light:
  - platform: esp32_rmt_led_strip
    pin: GPIO4          # change if rewiring the data line
    num_leds: 120        # match to your trimmed strip length
```

If you trim the strip while building the housing, `num_leds` must match the
physical count or the tail LEDs will read stale data.

</details>

## Usage

Once flashed, the lamp exposes a `light` entity plus presence and touch
binary sensors over the ESPHome native API:

```yaml
# Example Home Assistant automation: notify when the lamp turns on
automation:
  - alias: "Arc Lamp turned on"
    trigger:
      - platform: state
        entity_id: light.arc_lamp
        to: "on"
    action:
      - service: notify.mobile_app
        data:
          message: "Arc Lamp is on"
```

No Home Assistant instance is required for the lamp itself to function — touch
pads and presence detection work standalone out of the box.

## Wiring

| Peripheral | ESP32-S3 pin |
| :--- | :---: |
| WS2815 data | GPIO4 |
| LD2410B UART RX / TX | GPIO17 / GPIO18 |
| MPR121 I2C SDA / SCL | GPIO8 / GPIO9 |

```
USB-C PD source → CH224K (12V) ──┬─→ WS2815 strip (12V)
                                  └─→ LM2596 buck → 5V → ESP32-S3, LD2410B
ESP32-S3 3V3 → MPR121
```

Full power/signal net map, decoupling recommendations, and per-module pin
tables: [`docs/wiring.md`](docs/wiring.md).

## Contributing

Issues and pull requests are welcome — this is a personal desk project, so
response time may vary, but fixes, wiring corrections, and firmware
improvements are genuinely appreciated.

1. Fork the repo
2. Create a branch (`git checkout -b fix/thing`)
3. Commit your changes
4. Open a PR describing what changed and why

## License

[MIT](LICENSE) — free to use, modify, and build on.
