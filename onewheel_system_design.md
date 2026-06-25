# Onewheel System Design
**Project:** SBS V1.0 — Self-Balancing Unicycle  
**Last updated:** 2026-06-24  
**Status:** Architecture defined; chassis geometry TBD

---

## 1. System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        POWER SYSTEM                             │
│                                                                 │
│  ┌──────────┐    ┌─────────┐   XT90-S   ┌──────────────────┐   │
│  │ Charger  │───▶│   BMS   │◀──────────▶│  Battery Pack    │   │
│  │ 58.8V/5A │    │ 14S 100A│            │  14S2P Molicel   │   │
│  └──────────┘    └────┬────┘            │  P42A  (~58.8V)  │   │
│   GX16 port           │ 58.8V           └──────────────────┘   │
└───────────────────────┼─────────────────────────────────────────┘
                        │ 58.8V  (XT90-S antispark)
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SBS_hw DRIVER BOARD                          │
│                                                                 │
│   DRV8301 ──── 3-phase bridge (6× MOSFET ≥80V)                 │
│   STM32F405 ── BMI160 IMU (balance control)                     │
│   VESC 6.x  ── Balance App                                      │
│                                                                 │
│   Inputs:  Footpad ADC × 2    Hall sensors A/B/C                │
│            NRF51822 wireless  CAN (unused on onewheel)          │
│   Outputs: Phase A/B/C ──────────────────────────────┐          │
│            NeoPixel data ──────────────┐             │          │
│            5V/3.3V to sensors          │             │          │
└────────────────────────────────────────┼─────────────┼──────────┘
                                         │             │
                              ┌──────────┴──┐   ┌──────┴──────┐
                              │  LED strips │   │ PHUB-188    │
                              │  (WS2812B)  │   │ Hub motor   │
                              │  Front/Rear │   │ + Hall sens.│
                              └─────────────┘   └─────────────┘
                   ┌──────────────────┐
                   │  Footpad sensors │
                   │  FSR × 2 (front │
                   │  and rear pads) │
                   └─────────┬────────┘
                             │ analog
                             ▼
                       COMM header
                       (ADC_15, AN_IN)
```

---

## 2. Component Decisions

### 2.1 Motor — PHUB-188

| Parameter | Value |
|---|---|
| Type | Brushless hub motor, 10-inch |
| Rated voltage | 48 V |
| Rated power | 800 W |
| Tire size | 10 × 6 × 5.5 inch |
| Interface | 3-phase + 3-wire Hall sensors |
| Source | peipeiscooter.com |

The motor IS the wheel — direct drive, no gearbox, no chain. Simplicity is the main advantage.

---

### 2.2 Battery Pack

See `battery_design.md` for full detail, safety, and build procedure.

| Parameter | Value |
|---|---|
| Configuration | 14S2P |
| Cells | Molicel P42A (21700) |
| Nominal voltage | 14 × 3.6V = 50.4V |
| Max charge voltage | 14 × 4.2V = **58.8V** |
| Cutoff voltage | 14 × 2.8V = 39.2V |
| Capacity | 2 × 4.2Ah = 8.4Ah |
| Energy | ~434 Wh |
| Max continuous discharge | 2 × 45A = **90A** |
| Construction | Spot welded, pure nickel strip 0.2mm |
| Physical layout | Split across two foot well cavities, ~14 cells per cavity |

---

### 2.3 BMS — Daly Smart 14S 100A

| Parameter | Value |
|---|---|
| Cell count | 14S |
| Continuous discharge | 100A |
| Per-cell overvoltage cutoff | 4.20V |
| Pack undervoltage cutoff | 2.80V/cell (39.2V) |
| Short-circuit response | < 500 µs |
| Balancing | Passive (built-in) |
| Monitoring | Bluetooth app (Daly Smart) |
| NTC inputs | 1–2 (mount on cell surface in pack) |

The BMS sits between the battery and the XT90-S main connector. Its discharge path carries full motor current; its charge path connects to the GX16 charge port.

> **Wiring note:** In many DIY builds the BMS is wired in a "common port" configuration — one path for both charge and discharge. An alternative is "separate port" — distinct charge and discharge connectors. Separate port is safer (prevents charging through the discharge path and vice versa) and is preferred here.

---

### 2.4 Antispark — XT90-S Connector

Plugging a 58.8V battery into the 1000µF bulk capacitor bank on the driver board without pre-charge creates a large inrush current spike that can weld/damage connector contacts.

**Solution:** Use an **XT90-S** (antispark variant) connector as the main battery disconnect. The XT90-S has a built-in pre-charge resistor (~5.6Ω) in a shorter pin that makes contact first, limiting inrush to ~10A, before the main contacts engage.

- Cost: ~£3/pair
- No extra circuitry required
- This is the standard approach in the DIY onewheel community
- Pair with a "loop key" (short pigtail with male XT90-S) that plugs in to arm the board

---

### 2.5 Charger

| Parameter | Value |
|---|---|
| Output voltage | 58.8V (14 × 4.2V) |
| Output current | 5A (0.6C charge rate — safe, not slow) |
| Charge time | ~2 hours (0% → 100%, 8.4Ah / 5A) |
| Connector | GX16-3 (3-pin aviation connector) — one pin charge+, one GND, one NTC (optional) |
| Type | CC/CV lithium charger |

**Sourcing:** Search "58.8V 5A charger" or "14S lithium charger" — these are standard e-bike chargers. Verify output voltage with a multimeter before connecting to the pack for the first time.

> **Do not use a generic "60V" charger** without confirming the CV (constant voltage) phase cuts off at exactly 58.8V. Overcharging past 4.2V/cell, even briefly, degrades cells and is a fire risk.

---

### 2.6 Charge Port — GX16-3

GX16 aviation connectors are the de-facto standard for DIY e-vehicle charging ports. They are:
- Keyed (can't insert wrong orientation)
- Rated for 5A continuous
- IP54 with the dust cap fitted
- Small enough to mount flush in a panel

Mount on the underside of the board or rear bumper, with the dust cap retained on a lanyard.

---

### 2.7 Motor Driver — SBS_hw Board

The custom VESC 6 MK5 derivative. See `requirements.md` and `schematic_review.md` for full detail.

Key connections to the rest of the system:

| Signal | Connector on board | Wire spec |
|---|---|---|
| Battery in (SUPPLY) | XT90-S female | 8AWG silicone, as short as possible |
| Phase A/B/C out | 3× MR30 or solder lugs | 8AWG silicone (peak 80A, <500ms) |
| Hall A/B/C + 5V + GND | JST-PH 1.0mm 5-pin | Standard hub motor hall cable |
| Motor NTC (optional) | JST-PH 1.0mm 2-pin | From motor body |
| Footpad sensor A | COMM header ADC_15 | Thin signal wire |
| Footpad sensor B | COMM header AN_IN | Thin signal wire |
| NeoPixel data | GPIO via COMM header | Single data wire + GND |
| SWD | 4-pin debug header | For firmware flashing only |
| USB | Micro-B | VESC Tool configuration |

**VESC firmware target:** `60_mk5` (confirmed in VESC repo — matches STM32F405, DRV8301, 0.5mΩ shunts)

---

### 2.8 Footpad Sensors

The footpad sensors are a **dead man's switch** — the board only attempts to balance when both feet are detected. If either foot lifts (e.g. falling off), the motor stops. This is the primary active safety mechanism.

**Chosen type: Force Sensitive Resistors (FSR)**

Two sensors, one under each foot area (front pad and rear pad).

| Property | Value |
|---|---|
| Part | Interlink FSR 402 or FSR 406 (long strip) |
| Resistance | ~1 MΩ unloaded → ~10 kΩ at light foot pressure → ~200 Ω at full body weight |
| Interface | Voltage divider: FSR in series with a fixed resistor (10 kΩ) → ADC input |
| Supply | 3.3V from board |
| ADC pins | ADC_15, AN_IN (COMM connector on SBS_hw) |

**Circuit per sensor:**
```
3.3V ──── FSR ──── ADC_pin
                       │
                    10kΩ
                       │
                      GND
```
When foot is off: FSR ≈ 1 MΩ, V_ADC ≈ 3.27V (high)  
When foot is on: FSR ≈ 200 Ω, V_ADC ≈ 0.065V (low)

In VESC Balance App: configure ADC threshold to detect "foot on" at V_ADC < 0.5V. Both channels must be below threshold for the balance loop to activate.

**Alternative: Capacitive sensing** (used by commercial Onewheel XR)  
More reliable in wet conditions (no mechanical pressure needed), but requires a dedicated capacitive sensing IC (e.g. FDC2212 or a simple 555-based circuit). Worth considering for a v2 if FSR proves unreliable outdoors.

---

### 2.9 Lighting — WS2812B NeoPixels

Per the original blog: front and rear individually addressable RGB LED strips.

| Parameter | Value |
|---|---|
| LED type | WS2812B (5050 package, integrated driver) |
| Control | Single-wire, 800kHz data signal |
| Supply | 5V (from SBS_hw board) |
| Control source | STM32F405 GPIO via LispBM script on VESC |
| Suggested count | 8–12 LEDs front, 8–12 rear |
| Current | ~60mA/LED at full white; in practice 5–10mA average in normal colour modes |
| Total at 12 LEDs/end, average: | ~240mA — within 5V rail budget |

VESC's LispBM environment has native NeoPixel support (`ws2812-set-brightness`, `ws2812-set-color`, etc.). Lighting patterns (idle, riding, braking, fault) can be scripted without recompiling firmware.

**Mounting:** Recess into underside of nose/tail bumpers, facing forward/backward. Use a diffuser (frosted PETG strip, or 3M diffusion film) for even illumination.

---

### 2.10 Wireless Remote — NRF51822

Already on the SBS_hw board (NRF51822 EYSGJNZWY). Used with the VESC NRF remote (small handheld puck with a trigger). Primarily useful for:
- Remote speed limiting during initial testing (confidence mode)
- On/off from pocket before mounting

> **EOL risk noted:** Verify NRF51822 availability before PCB order. NRF52832 is a pin-compatible upgrade if needed (requires updated NRF firmware, VESC host side already supports it). For the dog, wireless is not needed.

---

## 3. Power Flow

```
Wall AC
  │
  ▼ 58.8V / 5A
Charger ──GX16──▶ BMS charge port
                       │
                  BMS (monitors
                  cell voltages,
                  balances, protects)
                       │
              Battery 14S2P
                  (50.4V nom)
                       │
                  BMS discharge port
                       │
                  XT90-S (antispark loop key)
                       │ 58.8V–39.2V (full to cutoff)
                       ▼
               SBS_hw SUPPLY input
                  │
          ┌───────┴────────┐
          │                │
   3-phase bridge     DRV8301 buck
   (motor drive)      → 5V → 3.3V
          │           (logic supply)
          ▼
     PHUB-188
     hub motor
```

---

## 4. Signal / Control Flow

```
BMI160 IMU
  │ pitch angle, rate (I2C, 1kHz)
  ▼
STM32F405
  ├── Balance App PID loop (1kHz)
  │     ├── Input: pitch error + rate + footpad state
  │     └── Output: motor current setpoint
  │
  ├── FOC current loop (20–50kHz)
  │     └── Drives DRV8301 → MOSFET gates → Phase A/B/C
  │
  ├── Hall sensors (A/B/C) ──▶ rotor position
  │
  ├── Footpad ADC (ADC_15, AN_IN) ──▶ ride enable/disable
  │
  ├── LispBM script ──▶ NeoPixel lighting
  │
  └── NRF51822 ──▶ wireless remote input
```

**Balance control summary:**  
1. IMU measures board pitch (forward tilt)  
2. Both footpads must be active (rider mounted)  
3. PID loop converts pitch error → target current  
4. Positive pitch (nose up) → forward current → motor accelerates forward → board levels  
5. Negative pitch → reverse current → decelerates/reverses  
6. Rider controls speed by leaning; the board follows to stay level  

---

## 5. Wiring Specification

| Connection | Wire gauge | Type | Max length |
|---|---|---|---|
| Battery → BMS → XT90-S | 8 AWG | Silicone, 200°C | As short as possible |
| XT90-S → SBS_hw SUPPLY | 8 AWG | Silicone, 200°C | < 150mm |
| Phase A/B/C → motor | 8 AWG | Silicone, 200°C | < 300mm |
| Hall sensor cable | 28 AWG × 5 | Standard hall cable (comes with motor) | < 500mm |
| Footpad sensors | 28 AWG × 2 | Thin hookup wire | < 600mm per sensor |
| NeoPixel power (5V) | 22 AWG | Standard | < 500mm |
| NeoPixel data | 28 AWG | Single wire | < 500mm |
| Charge cable (GX16 → BMS) | 20 AWG | Silicone | < 300mm |

All high-current connections (≥8 AWG) should be as short as possible to minimise inductance (reduces switching transients on the MOSFET drain).

---

## 6. Chassis / Enclosure — TBD

No mechanical design has been started. The following constraints are known:

| Constraint | Value | Source |
|---|---|---|
| Wheel diameter | 254mm (10 inch) | PHUB-188 spec |
| Wheel width | 152mm | PHUB-188 (10×6×5.5 tire) |
| Foot well depth | ~40–50mm target | Battery cell geometry (§2.2) |
| Foot well length | ~200mm per side | Cell count estimate |
| Min foot pad width | ~130mm | Cell count estimate |
| Rider weight capacity | TBD — design for ≥100kg | |

**Open design decisions:**
- Deck material: aluminium extrusion vs. steel vs. CNC plate vs. carbon fibre
- Frame style: monocoque vs. separate axle plate + deck
- Bumper material: TPU 3D print vs. rubber vs. UHMWPE
- Footpad grip surface: grip tape vs. rubber moulding
- Enclosure sealing: gasket + screws vs. potting vs. rely on outer housing

---

## 7. Open Questions

| # | Question | Blocks |
|---|---|---|
| 1 | What is the chassis geometry? Measure/design the foot well to confirm 14S2P fits | Battery pack build, PCB dimensions |
| 2 | ≥80V MOSFET selection for SBS_hw (see schematic_review.md §3.1) | PCB layout |
| 3 | Bootstrap diode replacement (PMEG10020ER) | PCB layout |
| 4 | Footpad sensor: FSR or capacitive? | Wiring, software config |
| 5 | NRF51822 availability check | BOM / ordering |
| 6 | Phase wire connector: MR30 or solder lug direct to PCB? | PCB layout, mechanical |
| 7 | How is the SBS_hw board mechanically fixed and thermally coupled to chassis? | Mechanical design |
| 8 | Charge port location on chassis | Mechanical design |
