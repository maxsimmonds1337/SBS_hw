# Schematic Review — SBS_hw v1.0
**Reviewer:** Claude (automated review against VESC 6 MK5 reference)  
**Reference:** VESC 6 MK5 by Benjamin Vedder (VESC_6_mk5.pdf, 6 sheets)  
**Date:** 2026-06-24

---

## Summary

The SBS_hw design is a close derivative of the VESC 6 MK5 schematic. Sheet structure and topology are identical. Component substitutions are minor and mostly acceptable. One critical issue exists (MOSFET voltage rating) and several items warrant verification before layout.

**Status:** Not cleared for layout. Resolve §3.1 (MOSFET Vds) first.

---

## Sheet-by-Sheet Review

### Sheet 1 — Top Level

Hierarchical structure matches reference: DRV8301, MCU, CAN, Power, BMI160 sub-sheets, with NRF51822 and peripheral connectors at top level.

**Differences from reference:**
- BMI160 IMU added — good addition, essential for balance application. Verify I2C pull-ups are present (2.2–4.7 kΩ to 3.3 V) on SDA/SCL.
- TCAN1051VDQ1 replaces TJA1051TK3 — see §3.2.
- NRF51822 module variant EYSGJNZWY — matches reference. ✓

**Items to verify:**
- EN_BUCK pull-up/pull-down behaviour at power-on. Reference sequence: supply → DRV8301 PVDD → EN_BUCK floats high → buck enabled → 5V → LDO → MCU → MCU asserts EN_GATE. Confirm your design follows the same safe power-on order.
- Roll-to-start / CONN_SW circuit: carried over from reference. Fine.

---

### Sheet 2 — CAN

TJA1051TK3 replaced with TCAN1051VDQ1. Decoupling capacitors (2.2 µF each side) retained. ✓

No issues found.

---

### Sheet 3 — MCU

STM32F405RGTx in LQFP64. Matches reference exactly. ✓

**Items to verify:**
- Crystal: 8 MHz with 15 pF load caps. Confirm crystal load capacitance matches selected crystal datasheet (reference uses 8 pF caps + PCB stray, check your crystal's C_L).
- Bypass capacitors: 2.2 µF on VBAT, VDDA, VCAP1/VCAP2 (100 nF on VCAP nodes). These are critical for the internal voltage regulator; do not omit.
- All VDD and VSS pins must be individually decoupled. Reference shows 2.2 µF bulk + distributed 100 nF. ✓

---

### Sheet 4 — DRV8301

**Critical issue: §3.1 (MOSFET voltage rating) relates to components instantiated from this sheet.**

Bootstrap capacitors: 220 nF — matches reference. ✓  
Phase sense resistors: 39 kΩ / 60 V rating — voltage rating is the sense circuit limit, not the bus. ✓  
MC74VHC1GT66 analogue switches for current filter: matches reference. ✓  
SENS_SUPPLY MOSFET and resistor divider: matches reference. ✓  
LDO (TC2117): provides 3.3 V VCC. ✓  
TVS on 5 V rail (SMAJ5.0A): appropriate for ESD/transient protection. ✓

**Items to verify:**
- Bootstrap Schottky diodes (PMEG6020ER, 20 V, 2 A): see §3.3.
- Gate resistors on INH/INL: reference uses 4.7 Ω. Confirm these are present. Missing gate resistors cause oscillation and parasitic turn-on risk.
- DRV8301 SPI CS line — confirm chip-select is properly driven and not left floating.
- Fault output (FAULT pin, active low): confirm pull-up to 3.3 V and connection to MCU GPIO with interrupt capability.

---

### Sheet 5 — BMI160

BMI160 connected via I2C (SDA→PB2, SCL→PA15 based on top-level sheet). Decoupling caps (2.2 µF ×2) present.

**Addition over reference.** This sheet is new.

**Items to verify:**
- I2C pull-ups: must be present somewhere (top-level or this sheet). 2.2 kΩ to 3.3 V recommended at 400 kHz.
- INT1/INT2 lines pulled to defined state (not floating).
- CSB tied to VCC for I2C mode (vs SPI).

---

### Sheet 6 — Power

Three half-bridge phases (A, B, C), each with:
- High-side + low-side N-MOSFET (IRF7749L1TRPBF) driven via DRV8301
- 4.7 Ω gate resistors
- 0.5 mΩ shunt resistors for current sensing
- AD8418 current sense amplifiers

Bulk capacitors: 2 × 680 µF (voltage rating not visible in KiCad value field — see §3.4)  
NTC thermistor for PCB temperature sensing. ✓

**Critical issue:** §3.1 — MOSFET voltage rating.

---

## Critical Issues

### 3.1 MOSFET Voltage Rating — MUST FIX BEFORE LAYOUT

| Item | Value |
|---|---|
| Component | IRF7749L1TRPBF |
| Vds absolute max | **40 V** |
| Target max bus voltage | **58.8 V** (14S LiPo) |
| Onewheel motor rated voltage | **48 V** (PHUB-188) → fully charged 13S ≈ 54.6 V |

A 40 V MOSFET on a 48–55 V bus will fail. Switching transients add L·di/dt on top of the DC bus rail, pushing Vds above Vbus on every switching edge. Even at 36 V (9S LiPo), 40 V leaves no margin.

**Action:** Replace IRF7749L1TRPBF with a ≥80 V, low Rds(on) part in a compatible package. Candidate parts (verify availability at time of order):

| Part | Vds | Id (25°C) | Rds(on) typ | Package | Notes |
|---|---|---|---|---|---|
| CSD18563Q5A (TI) | 60 V | 100 A | 5.6 mΩ | PowerPAK 5×6 | Minimum viable for 13S |
| NVMFS5C430NLT1G (ON Semi) | 40 V | — | — | — | Do NOT use — same issue |
| BSC093N08NS5 (Infineon) | 80 V | 100 A | 9.3 mΩ | TSDSON-8 | Good headroom |
| IPT111N10N5 (Infineon) | 100 V | 120 A | 1.1 mΩ | TO-263-7 | Low Rds but large package |
| IRFS4115 (Infineon) | 150 V | 104 A | 10 mΩ | D2PAK | Conservative, easier layout |

**Recommendation:** Target Vds ≥ 80 V for 58.8 V max bus (gives 36% derating headroom). If finalising at 13S (54.6 V max), 60 V parts are mathematically sufficient but leave only 10% margin against transients — 80 V is safer.

---

## Non-Critical Issues / Warnings

### 3.2 CAN Transceiver Substitution

| Reference | TCAN1051VDQ1 (TI) | TJA1051TK3 (NXP) |
|---|---|---|
| Speed | 1 Mbit/s | 1 Mbit/s |
| Supply | 4.5–5.5 V | 4.5–5.5 V |
| Footprint | SOIC-8 | SOIC-8 |

Functionally equivalent drop-in. TCAN1051HDRQ1 is the AEC-Q100 variant if automotive-grade reliability is needed. No schematic changes required. ✓

### 3.3 Bootstrap Schottky Diodes (PMEG6020ER — 20 V, 2 A)

The PMEG6020ER has Vr = 20 V. In a bootstrap circuit the diode must block the full bus voltage during the high-side switch-on phase. At Vbus > 20 V this part is out of specification.

**Action:** Replace with a ≥ 100 V Schottky. Suggested: **PMEG10020ER** (100 V, 2 A, same SOD-123 package). Alternatively STPS2L60A (60 V, 2 A, SOD-123) if finalising at 13S.

### 3.4 Bulk Electrolytic Capacitor Voltage Rating

The schematic value field shows "680u" without a voltage rating suffix. The reference uses 680 µF / 63 V. With the onewheel bus reaching 54.6 V (13S) or 58.8 V (14S), 63 V rating has only 7–15% margin — below the 20% minimum derating for electrolytics.

**Action:** Specify 680 µF / 100 V (e.g. Panasonic EEU-FR2A681L or equivalent). Confirm this fits within the PCB footprint (typically D16 × L20 mm for 100 V / 680 µF radial).

### 3.5 Shunt Resistor Power Dissipation

0.5 mΩ shunts at 200 A peak: P = I² × R = 200² × 0.0005 = **20 W** per shunt instantaneously.  
At 50 A continuous: P = 50² × 0.0005 = **1.25 W** per shunt.

Most 2512-package shunts are rated 1–3 W. At 50 A continuous the shunt is near its thermal limit. Parallel two shunts per phase (0.25 mΩ each, 2 W shared), or use a shunt rated for higher power (e.g. Isabellenhutte PBV-R0005-F1-0.5 rated 3 W in 2512).

At normal operating currents (20–30 A continuous) a single 1 W shunt is fine. Flag this if targeting >40 A continuous.

### 3.6 AD8418 Gain / ADC Range Check

AD8418 gain: 20 V/V fixed. Shunt: 0.5 mΩ.  
ADC full scale: 3.3 V → max measurable current = 3.3 / (20 × 0.0005) = **330 A** ✓  
At 50 A: output = 50 × 0.0005 × 20 = **0.5 V** — only 15% of ADC range. Resolution is fine for FOC at typical currents; at 330 A the range saturates cleanly.

No issue.

---

## Items Not Yet Visible / To Confirm

- [ ] Gate resistor values populated (expected 4.7 Ω per gate)
- [ ] Bootstrap cap value 220 nF confirmed in schematic
- [ ] Bulk cap voltage rating — add "100V" suffix to schematic value field
- [ ] BMI160 I2C pull-up resistors present
- [ ] BMI160 CSB tied to VCC for I2C mode
- [ ] INT1/INT2 of BMI160 pulled to defined state
- [ ] All DRV8301 decoupling caps placed per datasheet (PVDD1/2, AVDD, DVDD, VDD_SPI)
- [ ] FAULT and EN_GATE signal integrity — short traces, defined pull resistors
- [ ] PCB: high-current return paths reviewed (power GND star point or solid plane)
- [ ] PCB: bootstrap cap placement — must be within 5 mm of BST/SH pins
