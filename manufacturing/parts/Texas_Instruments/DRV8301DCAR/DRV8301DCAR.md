---
mpn: DRV8301DCAR
manufacturer: Texas Instruments
description: 3-phase gate driver with dual current sense amp and buck converter, HTSSOP-56
lcsc: C98969
package: HTSSOP-56
footprint: Package_SO:Texas_HTSSOP-56_6.1x9.7mm_P0.65mm
datasheet_url: https://www.ti.com/lit/gpn/DRV8301
kicad_datasheet: ${KIPRJMOD}/../../manufacturing/parts/Texas_Instruments/DRV8301DCAR/DRV8301DCAR.pdf
used_as: U? (DRV8301 sheet) — main gate driver IC
---

# DRV8301DCAR — 3-Phase Gate Driver + Buck Converter

## Key Specs

| Parameter | Value |
|-----------|-------|
| PVDD operating range | 6 V to 60 V |
| PVDD absolute max | 62 V |
| Gate drive peak source | 1.7 A (typ) |
| Gate drive peak sink | 2.3 A (typ) |
| Bootstrap voltage | 4.5 V to 17 V |
| Buck converter | 5V/2A internal, EN_BUCK controlled |
| Current sense amps | 2× integrated (gain 10V/V or 40V/V) |
| SPI interface | 16-bit, up to 10 MHz |
| Package | HTSSOP-56 (6.1 × 9.7 mm, 0.65 mm pitch) |
| Tj max | 150 °C |

## Design Voltage Check

```
PVDD = SUPPLY (battery bus) = 58.8V max (14S)
DRV8301 PVDD max = 60V

Margin: 60 - 58.8 = 1.2V  ← VERY TIGHT

At 14S with 4.2V/cell = 58.8V, the DRV8301 is within spec by only 1.2V.
Any overvoltage transient on the bus (regenerative braking, inductive kick)
risks exceeding PVDD_abs_max = 62V.
```

> **Risk:** Ensure the 4.7µF 100V MLCC bulk caps (CGA6M3X7S2A475KT0Y3S) are placed directly
> at PVDD pins. Consider a TVS or varistor at the SUPPLY input if regenerative braking
> is used aggressively.

## Footprint Issue (Critical)

**The current schematic assigns the wrong footprint.**

The DRV8301DCAR (DCAR = HTSSOP-56) is a 56-pin HTSSOP package. The KiCad schematic
shows a custom symbol `3_Phase_BLDC_Driver_56_Pin_HTSSOP_DRV8301` but the footprint
filter previously resolved to `PVQFN-40` (completely wrong package).

**Action required:** Override the footprint in the schematic to:
`Package_SO:Texas_HTSSOP-56_6.1x9.7mm_P0.65mm`

This footprint has 56 pins at 0.65mm pitch, 6.1×9.7mm body — matches the datasheet.

## Pinout Verification Notes

The DRV8301 HTSSOP-56 pin assignments (from TI datasheet):

| Pin | Name | Function | Connected to |
|-----|------|----------|-------------|
| 29, 53, 54 | PVDD | Power supply | SUPPLY bus |
| 55 | EN_BUCK | Buck enable | EN_BUCK power latch net |
| 47 | GH_A | High-side A gate | GH_A → Power sheet |
| 45 | GL_A | Low-side A gate | GL_A → Power sheet |
| 43 | SL_A | Source A (low-side) | SL_A → Power sheet |
| 44 | SH_A | Source A (high-side) | SH_A → Power sheet |
| 48 | BST_A | Bootstrap A | BST_A diode cathode |
| 42 | GH_B | High-side B gate | GH_B → Power sheet |
| 40 | GL_B | Low-side B gate | GL_B → Power sheet |
| 41 | SH_B | Source B (high-side) | SH_B → Power sheet |
| 38 | BST_B | Bootstrap B | BST_B diode cathode |
| 37 | GH_C | High-side C gate | GH_C → Power sheet |
| 35 | GL_C | Low-side C gate | GL_C → Power sheet |
| 36 | SH_C | Source C (high-side) | SH_C → Power sheet |
| 9 | SDI | SPI data in | SPI3_MOSI (MCU) |
| 10 | SDO | SPI data out | SPI3_MISO (MCU) |
| 11 | SCLK | SPI clock | SPI3_SCK (MCU) |
| 8 | CS | SPI chip select | SPI3_CS (MCU) |
| 6 | FAULT | Fault output | nFAULT → MCU |
| 7 | EN_GATE | Gate enable | EN_GATE → MCU |
| 31 | VCAP_1 | Internal reg cap | 2.2µF to GND |
| 47 | VCAP_2 | Internal reg cap | 2.2µF to GND |

## Internal Buck Converter

The DRV8301 has an internal non-synchronous buck converter (PVDD → +5V, up to 1.5A):
- EN_BUCK pin controls enable — must be asserted HIGH for +5V to be available
- The power latch circuit in SBS_hw.kicad_sch drives EN_BUCK via the footpad/CONN_SW circuit
- +5V output powers the VBOOST for bootstrap and feeds the TC2117 LDO for +3.3V

```
Power tree:
  SUPPLY (58.8V) → DRV8301 buck → +5V → TC2117 LDO → +3.3V (MCU, NRF, BMI160)
                                        → VBOOST (bootstrap diode supply)
```

EN_BUCK pull-up: add 100kΩ from EN_BUCK to PVDD so the buck starts on power-on
(before the MCU takes control). Currently EN_BUCK is driven by the power latch
circuit and is not directly driven by an MCU GPIO — this is acceptable for the
Onewheel use case but means there is no software sequencing.

## Current Sense Amplifier (Internal)

DRV8301 has 2 integrated CSAs (for phases A and B). The SBS_hw uses external AD8418
for all 3 phases — this is consistent with the VESC 6 MK5 design which also uses
external AD8418 on all 3 phases (DRV8301 internal CSAs are not used).

## Decoupling

Per TI datasheet recommendations:
- PVDD bypass: 100nF (C1/C2/C3 in schematic) ✓ + 4.7µF 100V MLCC (added by user) ✓
- VCAP_1/VCAP_2: 2.2µF ✓ (internal LDO bypass — must be within 10mm of pins)
- AVDD: 100nF bypass to AGND

> AVDD (pin 24) shows as unconnected in the analyzer (PP-001 finding). Verify in
> schematic that AVDD is connected to VCC (3.3V) with 100nF bypass cap.
