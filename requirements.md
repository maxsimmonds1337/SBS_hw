# BLDC Motor Driver Requirements
**Project:** SBS_hw v1.0  
**Baseline hardware:** VESC 6 MK5 (Benjamin Vedder)  
**Status:** Draft v0.2 — 2026-06-24

Unified requirements for a single hardware design that covers two target platforms. Per-platform requirements are defined first; the unified envelope takes the more demanding value in each row.

---

## Target Platforms

### Platform A — Robot Dog / Quadruped

Four-legged robot. One driver board per motor axis; four boards per robot coordinated over CAN bus.

**Motor:** Generic X4114-370 KV outrunner (24 poles: 22 magnets, 24 slots)  
**Drivetrain:** Motor + cycloidal gearbox (James Bruton v3 design, scaled)  
**Operating voltage:** 4S–6S LiPo (14.8–25.2 V)

### Platform B — Self-Balancing Unicycle (Onewheel)

Single-axis balance vehicle. One driver board per vehicle.

**Motor:** PHUB-188 10-inch brushless hub motor (rated 48 V / 800 W)  
**Drivetrain:** Direct drive (motor IS the wheel)  
**Operating voltage:** 48 V nominal → 12S–13S LiPo (50.4–54.6 V charged max)

---

## 1. Electrical — Input / Bus

| Parameter | Platform A (Dog) | Platform B (Onewheel) | **Unified** | Basis |
|---|---|---|---|---|
| Min operating voltage | 14.8 V (4S) | 36 V (9S cutoff est.) | **14.8 V** | Dog 4S floor |
| Nominal operating voltage | 22.2 V (6S) | 48 V | **48 V** | Onewheel nominal |
| Max operating voltage | 25.2 V (6S charged) | 54.6 V (13S charged) | **54.6 V** | Onewheel 13S |
| Absolute max voltage (board must survive) | 29 V (add headroom) | 58.8 V (14S charged) | **58.8 V** | Onewheel 14S worst-case |
| On-board bulk capacitance | ≥ 470 µF | ≥ 1000 µF | **≥ 1000 µF** | Onewheel regen transients |
| Bulk cap voltage rating | ≥ 50 V | ≥ 80 V (derated) | **≥ 80 V** | ≥ 30% above 58.8 V |

> **Note:** The dog never operates above 25.2 V. A single board design must tolerate 58.8 V, but within a dog system the bus sits at 6S. This is one hardware envelope with two operating points inside it.

---

## 2. Electrical — Output / Motor Phase

Platform A current numbers are **provisional** pending bench torque test on the X4114. Values are estimated from comparable 4108/4114-class motors at 6S.

| Parameter | Platform A (Dog) ⚠️ est. | Platform B (Onewheel) | **Unified** | Basis |
|---|---|---|---|---|
| Continuous phase current | ~20 A | ~17 A (800 W / 48 V) | **25 A** | Dog dynamic load est. |
| Peak phase current (≤ 2 s) | ~50 A (est.) | 60–80 A | **80 A** | Onewheel braking/accel |
| Absolute HW overcurrent trip | — | — | **200 A** | AD8418 + shunt full scale = 330 A; trip at 200 A |
| PWM switching frequency | 20–40 kHz | 20–40 kHz | **20–40 kHz** (configurable) | VESC firmware range |
| Number of phases | 3 | 3 | **3** | |

---

## 3. Power Stage (MOSFET / Gate Driver)

| Parameter | Requirement | Basis |
|---|---|---|
| Gate driver | DRV8301 | Required for VESC firmware SPI interface |
| MOSFET Vds rating | **≥ 80 V** | 58.8 V max bus + switching transients + 30% derating |
| MOSFET Id continuous | ≥ 100 A | 2× peak phase current for thermal margin |
| MOSFET Rds(on) | ≤ 5 mΩ per switch | Conduction loss < 10 W at 50 A |
| Topology | 3-phase synchronous half-bridge | 6 switches (3 high-side, 3 low-side) |
| Deadtime | 100–200 ns | Shoot-through prevention |
| Bootstrap capacitors | 220 nF | Reference value, one per high-side |
| Bootstrap diode Vr | ≥ 100 V | Must block full Vbus; see schematic_review.md §3.3 |

> ⚠️ **Current design uses IRF7749L1TRPBF (Vds = 40 V) — does not meet this requirement. See schematic_review.md §3.1.**

---

## 4. Current Sensing

| Parameter | Requirement | Basis |
|---|---|---|
| Method | Inline shunt + instrumentation amp | AD8418, gain = 20 V/V |
| Shunt value | 0.5 mΩ (0.0005 Ω) | Reference value |
| Measurable range | 0–330 A | ADC full scale / (gain × shunt) |
| Overcurrent trip range | 0–200 A (configurable) | Software + DRV8301 hardware |
| Number of sense channels | 3 (one per phase) | Full 3-phase sensing |

---

## 5. Other Sensing

| Sensor | Requirement | Notes |
|---|---|---|
| Bus voltage | Resistor divider → ADC, ≤ 0.2 V resolution | Overvoltage detection, regen management |
| Phase voltage (back-EMF) | Resistor divider → ADC, 3 channels | Sensorless FOC |
| PCB temperature | NTC thermistor, ≤ 5 °C resolution | Thermal derating |
| Motor temperature | NTC via motor connector (optional) | Configurable in VESC |
| IMU | BMI160, 6-axis (accel + gyro), ≥ 1 kHz ODR | Required for Platform B balance; useful for Platform A |
| Rotor position | Hall sensors (×3) — primary; encoder — optional | 5 V Hall supply from board |

---

## 6. Firmware / Control

| Requirement | Detail |
|---|---|
| Firmware | VESC open-source (v6.x) — must run unmodified |
| MCU | STM32F405RGTx — mandated by VESC firmware |
| Control algorithm | FOC mandatory; trapezoidal commutation optional |
| Motor types | PMSM, BLDC (outrunner and hub) |
| Operating modes | Current (torque), Speed, Position, Duty-cycle |
| Sensorless operation | Required (back-EMF zero-crossing for startup and high-speed) |
| Hall sensor operation | Required (low-speed / startup with sensors) |
| Encoder | Optional (not required for initial bring-up) |

---

## 7. Communications

| Interface | Standard | Use |
|---|---|---|
| CAN bus | ISO 11898, 1 Mbit/s | Multi-axis coordination (Platform A — all four boards) |
| USB | USB 2.0 Full-Speed (Micro-B) | Configuration, FW update, VESC Tool |
| UART | 3.3 V logic | App UART (Bluetooth UART adapter, custom controller) |
| PWM input | Servo signal 50–400 Hz | RC receiver or balancing controller output |
| SWD | ARM SWD | Programming and debug |
| NRF51822 wireless | Proprietary / Bluetooth | Remote control / telemetry (Platform B primary) |

---

## 8. Protection

| Protection | Trip Threshold | Response |
|---|---|---|
| Overcurrent (hardware) | 200 A (DRV8301 OCx pins) | Gate disable within 1 µs, FAULT asserted |
| Overcurrent (software) | Configurable default 80 A | Current limiting via FOC loop |
| Overvoltage (bus) | 60 V (software), board survives 58.8 V | Disable PWM, fault |
| Undervoltage (bus) | 12 V (software, configurable) | Soft stop |
| Over-temperature (PCB) | Derate above 60 °C, disable above 85 °C | Linear current ramp-down |
| Over-temperature (motor) | Configurable (VESC) | Current derating |
| Short-circuit | — | DRV8301 hardware latch, cleared by EN_GATE cycle |
| Shoot-through | — | Deadtime in DRV8301 + VESC PWM generation |

---

## 9. Physical / Environmental

Parameters marked **TBD** are not yet constrained. They should be set once a packaging design is confirmed — do not invent values.

| Parameter | Platform A (Dog) | Platform B (Onewheel) | **Unified** |
|---|---|---|---|
| Form factor | TBD — must fit leg link or torso | TBD — must fit wheel housing | TBD |
| Heatsink / cooling | Forced air or conduction to chassis | Conduction to enclosure | Copper baseplate or exposed thermal pads preferred |
| Operating temp (ambient) | -10 °C to +50 °C | -10 °C to +40 °C | **-10 °C to +50 °C** |
| Ingress protection | TBD | TBD | TBD — seal requirements depend on final enclosure design |
| Vibration | High (robot impacts, leg strikes) | Medium (road surface) | Design for robot dog vibration level |
| Connector current rating | ≥ 80 A peak (phase outputs) | ≥ 80 A peak | **≥ 80 A** phase connectors, ≥ 80 A battery input |

---

## 10. Open Questions

1. **MOSFET selection** — Which ≥ 80 V part replaces the IRF7749L1TRPBF? See schematic_review.md §3.1 for candidates. Decision needed before layout.
2. **X4114 current validation** — Platform A peak current is estimated. Bench test with the torque jig (see robot_dog_butler blog) to confirm before finalising current ratings.
3. **Onewheel battery cell count** — 12S or 13S? Sets max bus voltage and determines whether 60 V or 80 V MOSFETs suffice.
4. **Bootstrap Schottky** — Replace PMEG6020ER (20 V) with PMEG10020ER or equivalent ≥ 60 V part.
5. **Bulk cap** — Confirm 680 µF / 100 V fits the PCB footprint. If not, two 330 µF / 100 V in parallel.
6. **Platform B packaging** — Is the board inside a sealed housing on the unicycle? Determines whether on-board IP sealing is needed or the enclosure handles it.
7. **Four-quadrant operation** — Robot dog legs must decelerate and brake dynamically. Confirm VESC firmware regen braking is enabled and the bus can absorb regen energy (or a bleed resistor / supercap is used).
8. **NRF51822 status** — Blog notes the BLE module may be EOL. Confirm availability or identify a replacement (NRF52832 is a likely drop-in with firmware updates).
