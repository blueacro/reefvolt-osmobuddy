# OsmoBuddy

OsmoBuddy is an open-source RO/DI (Reverse Osmosis / Deionization) monitor and controller for reef aquariums. It is built around an ESP32 and [ESPHome](https://esphome.io/), and feeds into [Home Assistant](https://www.home-assistant.io/) for dashboards, automations, and alerting.

For the full design story — including installation photos, parts sources, and discussion with other reefers — see the Reef2Reef thread:

**[osmoBuddy: RO/DI monitor and controller (reef2reef.com)](https://www.reef2reef.com/threads/osmobuddy-ro-di-monitor-and-controller.1078681/)**

## What it does

OsmoBuddy **monitors**:

- **Flow rate** from an inline impeller flow sensor (post-filter in the reference install)
- **Up to three pressures** from 0–5 V pressure transducers. In the reference install:
  - pre-filter (before the sediment / carbon blocks)
  - post-filter (membrane inlet)
  - RO storage tank line pressure
  — the display shows the pressure drop across the filter blocks, so it's obvious when they clog.
- **Up to three switches** (e.g. float switches on a salt-mix / ATO reservoir)

And **controls**:

- **Up to four 24 V valves or relays.** In the reference install these drive:
  - the **main water inlet valve** (a direct-acting non-diaphragm solenoid — this is the one that must not fail, so a reliable valve is used here)
  - a **flush valve** bridged across the flow restrictor to flush the RO membrane
  - (spare) a salt-mix reservoir refill solenoid
- **One on-board dry-contact relay** for signaling other controllers — for example, an **Apex** switch input used to gate automatic water change pumps (e.g. preventing AWC pumps from running after a salt-mix refill until salt has been added manually).

### Control logic

The stock firmware implements automatic pressure-based water production:

1. When storage-tank pressure drops below the **Turn On Pressure** setpoint, both the inlet and flush valves open — a **pre-flush** at startup.
2. After ~30 s the flush valve closes and the membrane starts producing water into the tank.
3. When tank pressure reaches the **Turn Off Pressure** setpoint, the flush valve re-opens for a **post-flush**.
4. After ~30 s the inlet valve closes; the flush valve stays open for a few more seconds to bleed off pressure, then closes.

It also supports a salt-mix auto-fill with a high-float safety cutoff, and a latching dry-contact signal out to an external controller.

> TDS measurement is **not** on this revision — it's planned for a future spin.

## Features

- ESP32-C3 Wi-Fi MCU running ESPHome
- ADS1115 16-bit ADC — 3 channels for 0–5 V pressure transducers
- Pulse-counter flow totalizer with persistent state across reboots
- PCA9554 I/O expander driving solenoid / relay outputs through a high-side driver
- On-board signal relay for dry-contact output to Apex or similar
- 1.3" SH1106 128×64 OLED for live local status (pressures, state, totalizer)
- Rotary encoder with push-button for local UI
- CAN transceiver (TCAN1044) for future expansion
- USB-C for flashing / debug
- 12 V DC input with on-board 5 V and 3.3 V rails

## Reference parts

| Part | Source |
|------|--------|
| 24 V flush valve | AliExpress |
| 24 V main inlet valve | McMaster-Carr [5489T413](https://www.mcmaster.com/5489T413/) |
| Pressure transducers | 1/8" NPT, 100 PSI, 0–5 V, stainless — Amazon |
| Flow sensor | Inline impeller type — Amazon |

## Repository layout

| Path | Contents |
|------|----------|
| `osmobuddy.yaml` | ESPHome firmware configuration |
| `osmobuddy-schematics.pdf` | Rendered schematic |
| `gerber/osmobuddy_v1/` | Gerber files for PCB fabrication |
| `bom/` | Bill of Materials (CSV) |
| `pnp/` | Pick-and-place files for assembly |
| `3d/` | Enclosure, wall mount, knob, and pump-switch add-on (Inventor, STL, 3MF) |
| `LICENSE` | Apache License 2.0 |

## Firmware

The firmware is a single ESPHome configuration (`osmobuddy.yaml`). To flash:

1. Install [ESPHome](https://esphome.io/).
2. Create a `secrets.yaml` alongside `osmobuddy.yaml` with your Wi-Fi credentials:
   ```yaml
   wifi_ssid: "your-ssid"
   wifi_password: "your-password"
   ```
3. Compile and upload:
   ```bash
   esphome run osmobuddy.yaml
   ```

Once connected, OsmoBuddy appears in Home Assistant via the ESPHome integration and exposes all sensors, switches, and the `Turn On Pressure` / `Turn Off Pressure` / `Force Off` controls.

## Manufacturing

The `gerber/`, `bom/`, and `pnp/` directories contain what's needed to order assembled boards from JLCPCB, PCBWay, or similar. The enclosure in `3d/` is designed for FDM 3D printing.

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.
