# PyFusion — RP2040 Sensor Fusion Board

A compact, custom-fabricated two-layer sensor fusion PCB built around the Raspberry Pi RP2040. Integrates a 6-axis IMU, 3-axis magnetometer, and barometric pressure sensor on a single board — programmable natively in MicroPython.

---

## Hardware

| Component | Part | Interface | Address |
|---|---|---|---|
| Microcontroller | RP2040 (dual-core Cortex-M0+, 133MHz) | — | — |
| IMU | IAM-20680HV (6-axis accel + gyro) | I2C1 — GP2/GP3 | 0x68 |
| Magnetometer | IIS2MDCTR (3-axis AMR) | I2C0 — GP8/GP9 | 0x1E |
| Barometer | BMP581 (pressure + temperature) | SoftI2C — GP6/GP7 | 0x46 |
| Flash | W25Q128JVS (128Mbit QuadSPI) | QSPI | — |
| Crystal | ABM8-272-T3 (12MHz) | XIN/XOUT | — |
| Regulator | NCP1117-3.3 (5V → 3.3V LDO) | — | — |

---

## PCB Design

The board is a two-layer design captured in KiCad 9.0. Layout decisions were driven primarily by the sensitivity requirements of the three MEMS sensors.

### Layer Stack

- **Top layer** — signal routing and 3.3V power polygon pour
- **Bottom layer** — continuous ground plane for low-impedance return paths and shielding, with minimal routing tracks

### Sensor Placement

**IMU (IAM-20680HV)**
Placed on a mechanically stable region of the board, away from mounting holes and connectors. Requires a solid ground plane beneath it for noise reduction — copper fill is maintained under this device. The exposed ground pad is connected to the ground plane via multiple vias.

**Magnetometer (IIS2MDCTR)**
Highly sensitive to magnetic fields from nearby copper currents. Ground plane and copper pours are cleared on both layers in the region surrounding this device. High-current traces and switching components are kept at a distance.

**Barometer (BMP581)**
Isolated from large copper regions on both layers to prevent thermal gradients and mechanical stress from influencing pressure readings.

### Key Layout Decisions

**Decoupling** — All bypass capacitors placed within 1–2mm of their associated VDD pins, with short via stubs to the ground plane, in accordance with each sensor's datasheet.

**Crystal** — ABM8-272-T3 placed as close as practical to the RP2040 XIN/XOUT pins. Load capacitors are inline. A copper pour guard ring tied to GND prevents switching-noise coupling into the oscillator.

**USB differential pair** — USBDP/USBDM routed as a 90Ω differential pair (0.8mm track width, 0.15mm gap on a 1mm substrate) per the RP2040 hardware design guide.

**GPIO header** — 23 pins expose selected RP2040 GPIO signals for external interfacing via UART, SPI, or I2C, making the board extensible rather than a closed system.

**BOOTSEL and RESET** — dedicated push-buttons for bootloader entry and manual reset.

### KiCad Footprint Note

The IAM-20680HV KiCad footprint files were not available at the time of design. The **ICM-20602** footprint was used as a drop-in substitute — the two devices share an identical package and pin layout. The assembled board uses the IAM-20680HV component on the ICM-20602 footprint without modification.

### Design Verification

Before manufacture the following checks were completed in KiCad:
- Electrical Rules Check (ERC) — no unconnected pins or conflicting power nets
- Design Rules Check (DRC) — cleared against manufacturer minimums (trace width, clearance, via drill, annular ring)
- Footprint verification — each land pattern cross-checked against the manufacturer datasheet
- Gerber preview — layer stack-up, drill file, and copper patterns confirmed

---

## Firmware

All firmware is written in MicroPython and runs directly on the RP2040. The board was developed and flashed using Thonny IDE.

### File Overview
