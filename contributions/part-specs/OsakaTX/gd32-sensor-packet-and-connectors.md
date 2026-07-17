# GD32 Sensor Status Packet — Updated Encoder & Connector Details

> **Source:** codetiger/VacuumTiger SENSORSTATUS.md, constants.rs (Dec 2025 – Jan 2026)
> **Last updated:** July 16, 2026

## Purpose

This file supplements `vacuumtiger-verified-specs.md` with updated information
from the VacuumTiger project's December 2025 – January 2026 code updates. The
encoder L/R offset assignment was corrected, and the full sensor packet layout
is now documented.

## Encoder Offset Correction (Dec 10, 2025)

The VacuumTiger commit "Fix wheel encoder assignment and lidar scan wrap
detection" (3f56cd4, Dec 10, 2025) corrected the left/right encoder offsets:

| Field | Offset | Previous (wrong) | Corrected |
|-------|--------|-------------------|-----------|
| Left wheel encoder | 0x10–0x11 | 0x18 | **0x10** |
| Right wheel encoder | 0x18–0x19 | 0x10 | **0x18** |

> **Note:** The current offsets (left = 0x10, right = 0x18) are the corrected
> assignment. An earlier VacuumTiger revision had the two swapped; commit 3f56cd4
> fixed it. These match the values in the companion `vacuumtiger-verified-specs.md`.

## Full Sensor Status Packet (CMD 0x15, 96-byte payload)

From `SENSORSTATUS.md` (codetiger/VacuumTiger, Dec 15, 2025):

```
Packet: [0xFA 0xFB] [LEN] [0x15] [PAYLOAD 96 bytes] [CRC_H] [CRC_L]
Total: 102 bytes
Frequency: ~110 Hz (every ~9ms at 115200 baud)
```

### Key Offsets (from constants.rs)

| Offset | Size | Field | Type |
|--------|------|-------|------|
| 0x01 | 1 | Bumper flags | u8 bitfield |
| 0x03 | 1 | Cliff flags | u8 bitfield |
| 0x04 | 1 | Dustbox flags | u8 bitfield |
| 0x07 | 1 | Charging flags | u8 bitfield |
| 0x08 | 1 | Battery voltage (raw÷10=volts) | u8 |
| **0x10–0x11** | 2 | **Left wheel encoder** | **u16 LE** |
| **0x18–0x19** | 2 | **Right wheel encoder** | **u16 LE** |
| 0x28–0x29 | 2 | Gyro X (Yaw rate, raw) | i16 LE |
| 0x2A–0x2B | 2 | Accel X | i16 LE |
| 0x2C–0x2D | 2 | Gyro Y (Pitch rate, raw) | i16 LE |
| 0x2E–0x2F | 2 | Accel Y | i16 LE |
| 0x30–0x31 | 2 | Gyro Z (Roll rate, raw) | i16 LE |
| 0x32–0x33 | 2 | Accel Z | i16 LE |
| 0x34–0x35 | 2 | Tilt X (LP-filtered gravity) | i16 LE |
| 0x36–0x37 | 2 | Tilt Y (LP-filtered gravity) | i16 LE |
| 0x38–0x39 | 2 | Tilt Z (LP-filtered gravity) | i16 LE |
| 0x3A–0x3B | 2 | Start button | u16 LE |
| 0x3E–0x3F | 2 | Dock button | u16 LE |
| 0x46 | 1 | Water tank level (0-100) | u8 |

### Flag Bit Definitions

**Bumper (0x01):** bit 1 = right bumper, bit 2 = left bumper
**Cliff (0x03):** bit 0 = left side, bit 1 = left front, bit 2 = right front, bit 3 = right side
**Charging (0x07):** bit 0 = dock connected, bit 1 = charging
**Dustbox (0x04):** bit 2 = dustbox attached

### Battery Voltage

```
BATTERY_VOLTAGE_MIN = 13.5V (0%)
BATTERY_VOLTAGE_MAX = 15.5V (100%)
percentage = (voltage - 13.5) / (15.5 - 13.5) * 100
```

## J25/J26 Connector Architecture (Updated)

From codetiger/VacuumRobot Component_Diagram.md (Oct 2025):

### J25 — Left Wheel (16-pin, 1.0mm SHD)

Contains signals for:
- Left wheel encoder (2-4 pins for A/B channels)
- Dustbox power (2 pins)
- Left cliff sensor (IR, 2-3 pins)
- Left bumper sensor (1-2 pins)

> Status: ❓ HYPOTHESIS — needs physical pinout analysis (pin-by-pin mapping
> to STM32F103 GPIOs still requires multimeter continuity tracing)

### J26 — Right Wheel (16-pin, 1.0mm SHD)

Contains signals for:
- Right wheel encoder (2-4 pins)
- Sweeper (side brush) motor power (2 pins)
- Right cliff sensor (IR, 2-3 pins)
- Right bumper sensor (1-2 pins)

> Status: ❓ HYPOTHESIS — needs physical pinout analysis

### Motor Power Connectors

- **J24** (left motor, 2-pin 2.0mm PH): motor power only
- **J27** (right motor, 2-pin 2.0mm PH): motor power only

## Robot Platform Clarification

The VacuumTiger/VacuumRobot research documents the **3irobotix CRL-200S**, which
uses a GD32F103VCT6 motor controller. The oomwoo BOM specifies **Roborock S5**
drive wheel assemblies, which use an **STM32F103VCT6** motor controller (see
`roborock-s5-mainboard-ics.md`). Both are pin-compatible Cortex-M3 MCUs, but the
firmware protocols and mainboard layouts differ.

The calibrated odometry constants (ticks_per_meter=4464, wheel_base=0.233m) are
from the CRL-200S. The Roborock S5 may have different values — these need
independent calibration on the Roborock platform.

## Sources

- VacuumTiger SENSORSTATUS.md: https://github.com/codetiger/VacuumTiger/blob/main/sangam-io/src/devices/crl200s/SENSORSTATUS.md
- VacuumTiger constants.rs: https://github.com/codetiger/VacuumTiger/blob/main/sangam-io/src/devices/crl200s/constants.rs
- VacuumTiger encoder fix commit (3f56cd4): https://github.com/codetiger/VacuumTiger/commit/3f56cd4bd7f0a53fde654b2a1a34788f6f079a39
- VacuumRobot Component_Diagram.md: https://github.com/codetiger/VacuumRobot/blob/main/Research/Motherboard/Component_Diagram.md
- VacuumTiger config.rs: https://github.com/codetiger/VacuumTiger/blob/main/sangam-io/src/devices/mock/config.rs
- VacuumTiger dhruva.toml: https://github.com/codetiger/VacuumTiger/blob/main/dhruva-nav/dhruva.toml
