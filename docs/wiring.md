# Wiring reference

Full net and pin map for schematic capture / point-to-point wiring.

## Power nets

| Net | Source | Feeds |
| :--- | :--- | :--- |
| `VBUS_IN` | USB-C connector VBUS | CH224K IN |
| `+12V` | CH224K OUT (negotiated, set to 12V) | WS2815 strip, LM2596 IN |
| `+5V` | LM2596 OUT (trim to 5.00V) | ESP32-S3 5V pin, LD2410B VIN |
| `+3V3` | ESP32-S3 onboard 3V3 regulator | MPR121 VDD, BH1750 VCC |
| `GND` | Common | All modules |

Add a ~2A fuse or ppt polyfuse between `VBUS_IN` and CH224K IN, a bulk 100–220µF
electrolytic on `+12V` near the strip header, and a 100nF decoupling cap at the
VDD/VIN pin of each IC.

## CH224K (USB-C PD trigger)

| Pin | Net |
| :--- | :--- |
| VBUS/IN | `VBUS_IN` |
| GND | `GND` |
| CC1, CC2 | USB-C connector CC1/CC2 |
| OUT+ | `+12V` |
| OUT− | `GND` |
| CFG pads | Bridge the board's labeled **12V** solder-jumper pad (varies by vendor — check your board's silkscreen) |

## LM2596 buck

| Pin | Net |
| :--- | :--- |
| IN+ | `+12V` |
| IN− | `GND` |
| OUT+ | `+5V` |
| OUT− | `GND` |
| Adjust pot | Trim by hand to 5.00V on OUT+ before connecting any load |

## ESP32-S3 DevKitC-1 (N16R8)

| Pin | Net / peripheral | Notes |
| :--- | :--- | :--- |
| 5V | `+5V` | Board input |
| GND | `GND` | Tie all GND pins together |
| 3V3 | `+3V3` | Powers MPR121 |
| GPIO4 | WS2815 DIN | Via inline 330–470Ω resistor |
| GPIO17 | LD2410B TX | ESP32 RX |
| GPIO18 | LD2410B RX | ESP32 TX |
| GPIO8 | MPR121 SDA | I2C data |
| GPIO9 | MPR121 SCL | I2C clock, add 4.7kΩ pull-ups to `+3V3` if not already on the MPR121 board |

> [!WARNING]
> Avoid GPIO33–37 on the N16R8 variant — reserved internally for octal PSRAM/flash.

## WS2815 strip (12V / GND / DIN)

| Pin | Net |
| :--- | :--- |
| 12V | `+12V` |
| GND | `GND` |
| DIN | GPIO4 (via 330–470Ω series resistor) |

## LD2410B

| Pin | Net |
| :--- | :--- |
| VIN | `+5V` |
| GND | `GND` |
| TX | GPIO17 |
| RX | GPIO18 |
| OUT | Optional spare GPIO — not required, ESPHome reads presence over UART |

## MPR121

| Pin | Net |
| :--- | :--- |
| VDD | `+3V3` |
| GND | `GND` |
| SDA | GPIO8 |
| SCL | GPIO9 |
| IRQ | Unconnected (polling mode, no interrupt needed) |
| ADDR | Tie to `GND` for I2C address `0x5A` |

## BH1750 (ambient light sensor)

Shares the same I2C bus as the MPR121 — no extra GPIOs needed, just wire it in
parallel.

| Pin | Net |
| :--- | :--- |
| VCC | `+3V3` |
| GND | `GND` |
| SDA | GPIO8 |
| SCL | GPIO9 |
| ADDR | Leave unconnected / tie to `GND` for I2C address `0x23` (matches the YAML config) |

<details>
<summary><b>Power budget notes</b></summary>

At 05 LEDs full white, the WS2815 strip can draw several amps. Size the 12V
wiring/traces for worst-case current even though typical desk-lamp brightness
levels draw far less. Keep the strip's 12V/GND runs short and thick, or feed
power at both ends of the strip if the arc is long, to avoid voltage sag and
color shift at the far end.

</details>
