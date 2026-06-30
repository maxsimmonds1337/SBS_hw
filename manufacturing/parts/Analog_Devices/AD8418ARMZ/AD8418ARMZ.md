---
mpn: AD8418ARMZ
manufacturer: Analog Devices
description: Bidirectional zero-drift current shunt monitor, MSOP-8 — current sense amp
lcsc: C2680958
package: MSOP-8 (RM suffix)
footprint: Package_SO:MSOP-8_3x3mm_P0.65mm
datasheet_url: https://www.analog.com/media/en/technical-documentation/data-sheets/AD8418.pdf
kicad_datasheet: ${KIPRJMOD}/../../manufacturing/parts/Analog_Devices/AD8418ARMZ/AD8418ARMZ.pdf
used_as: U5 (Phase A), U6 (Phase B), U7 (Phase C) — current sense amplifiers
note: Verify actual package used in schematic — AD8418 also available in SOIC-8 (RZ suffix)
---

# AD8418ARMZ — Bidirectional Current Shunt Monitor

## Key Specs

| Parameter | Value |
|-----------|-------|
| Gain | Fixed at 20 V/V |
| Input common mode | −2V to +65V |
| Supply voltage | 3V to 12V (powered from +3.3V or +5V) |
| Input offset (max) | 35 µV |
| Bandwidth | 1.2 MHz |
| PSRR | 80 dB (typ) |
| Package | MSOP-8 (ARM) or SOIC-8 (ARZ) |

## Gain and Full-Scale Calculation

```
Shunt resistors: R9, R13, R16 = 0.5 mΩ (0.0005 Ω)

AD8418 gain = 20 V/V

Full-scale output (ADC ref = 3.3V):
  Vout_fs = 3.3V (rail-to-rail output, supply = 3.3V)

  I_fs = Vout_fs / (Gain × Rshunt)
       = 3.3V / (20 × 0.0005Ω)
       = 3.3V / 0.01Ω
       = 330 A

Full-scale current range: ±330 A (bidirectional, output = Vcc/2 at 0A)
```

### Useful Operating Range

STM32F405 ADC Vref = 3.3V, 12-bit resolution:

```
LSB current = 330A × 2 / 4096 = 161 mA/LSB

At 80A motor peak:
  Vout = 3.3V/2 + (80A × 0.0005Ω × 20) = 1.65 + 0.8 = 2.45V
  ADC code = 2.45 / 3.3 × 4096 = 3042 counts (74% of full scale — good)

At 40A continuous:
  Vout = 1.65 + 0.4V = 2.05V
  ADC code = 2548 counts (62% of full scale — good)
```

Full-scale of 330A provides adequate headroom above peak 80A without wasting ADC resolution.

## Current Sense RC Filter (Power Sheet)

Each phase has a 2-stage RC filter after the AD8418 output:

```
AD8418 OUT → R18 (1kΩ) → IC_A node → ADC input
                              ├── C17 (1nF, always connected, to GND)
                              └── U8 switch → C16 (15nF, switched)

Filter OFF (U8 open, CURR_FILTER_ON low):
  RC = 1kΩ × 1nF = 1µs
  f_-3dB = 1 / (2π × 1µs) = 159 kHz

Filter ON (U8 closed, CURR_FILTER_ON high):
  C_total = 1nF + 15nF = 16nF
  RC = 1kΩ × 16nF = 16µs
  f_-3dB = 1 / (2π × 16µs) = 9.95 kHz

Purpose: VESC uses the high-bandwidth mode (filter OFF) during PWM off-time to
sample current without RC settling delay. Filter ON mode reduces noise during
continuous measurement windows.
```

**CURR_FILTER_ON** (PD2 on MCU sheet) drives U8/U9/U10 ON/OFF pins on the Power sheet.

Note: This is separate from SENS_FILTERED (PC13) which controls U2/U3/U4 on the
DRV8301 sheet (phase voltage sense filter) — two different control signals.

## Input Common Mode Verification

```
Shunt locations: source of low-side MOSFETs (near GND)
Shunt voltage at 80A peak: V = 80A × 0.0005Ω = 40 mV
Shunt node voltage range: 0V to +40mV (above GND)

AD8418 input CM range: -2V to +65V ✓ (well within spec for bottom-side shunt)
```

The AD8418 can also measure across top-side shunts (up to 65V CM), but this design
places shunts at the low-side — simpler and preferred.

## Decoupling

Each AD8418 requires 100nF bypass cap between VS and GND (C7/C12/C15 in schematic, 2.2µF in current design — slightly over-spec but fine).

## LCSC Note

AD8418ARMZ (MSOP-8) may have limited LCSC stock. Alternative: AD8418ARZ (SOIC-8, C2682498).
Verify which package is placed in the schematic footprint before ordering.
