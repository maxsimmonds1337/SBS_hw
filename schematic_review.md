# Schematic Review — SBS_hw v1.0
**Source:** Actual KiCad files + generated netlist (manufacturing/SBS_hw.net)  
**Date:** 2026-06-25  
**Method:** PDF review of all 5 sheets + netlist analysis for connectivity

---

## Summary

The SBS_hw is a custom VESC 6 MK5 derivative. The schematic is a **work in progress** — the Power sheet is incomplete and several critical wiring errors exist across multiple sheets. The design cannot be sent for layout in its current state.

**Status:** Not cleared for layout. See Critical Issues §3 below.

---

## Sheet-by-Sheet Review

### Sheet 1 — Top Level (SBS_hw.kicad_sch)

Contains: NRF51822 module, USB micro-B, peripheral connectors (SERVO, HALL_ENC, COMM, SWD, CAN), EN_BUCK control circuit, CONN_SW footpad detection, TC2117 LDO.

**Issues found:**

- **TC2117 (U1) has blank value field** — the Value is set to `"~"`. This will show as `~` in the BOM. Add `TC2117-3.3VDBTR` to the Value field.
- **BOOT0 tied directly to GND** (pin 60 of STM32F405) — no pull-down resistor. This works for normal operation (stays in flash boot mode), but prevents entering ISP/DFU mode in the field without reworking the board. Low risk for now; add a 10 kΩ resistor if ISP entry is needed.
- **Two 1 kΩ PTC resistors** on SHUTDOWN/CONN_SW for footpad detection — this is the correct approach. ✓
- EN_BUCK control: 39 kΩ/56 kΩ divider + Schottky on SH_A — intended to hold EN_BUCK high during power-on before MCU takes control. Logic is reasonable but verify the MCU GPIO that drives EN_BUCK (see §3.4).

---

### Sheet 2 — MCU (MCU.kicad_sch)

Contains: STM32F405RGTx LQFP-64, crystal, ferrite bead, VCAP caps, SPI routing to DRV8301.

**Issues found:**

- **BMI160 entirely absent** — the IMU is not on the MCU sheet or any other sheet. This is the primary sensor for the VESC Balance App (pitch angle). It **must** be added before the design is complete. Connect via I2C: SDA → PB7, SCL → PB6. Add 2.2 kΩ pull-ups to 3.3 V on both lines.
- **Crystal has no frequency value** — the crystal symbol `Y?` has no frequency in its Value field. Add `8MHz`.
- **SPI3 net naming error on PB3/PB4** — schematic labels these as SDI/SDO but the pin assignments are wrong relative to STM32F405 AF6 mapping. A note in the schematic reads: *"These are wrongly named, but I will copy them from the original vesc schematic"* — this needs to be resolved before layout. Correct mapping for SPI3 AF6: SCK=PB3, MISO=PB4, MOSI=PB5 (or PC10/PC11/PC12 alternate pins).
- **VCAP_1/VCAP_2 have 2.2 µF** ✓ — correct for STM32F405 internal regulator.
- **Ferrite bead** FCM1608KF-601T05 on power rail ✓

---

### Sheet 3 — DRV8301 (DRV8301.kicad_sch)

Contains: DRV8301 gate driver, bootstrap diodes, phase sense resistors, MC74VHC1GT66 analog switches (U2/U3/U4), SENS_SUPPLY transistor Q2, bulk caps.

**Issues found:**

- **DRV8301 footprint mismatch** — the symbol is the 56-pin HTSSOP version, but the footprint filter in the library is `Texas*S*PVQFN*N40*3.52x2.62mm*`. The placed instance inherits a PVQFN-40 footprint. These are completely different packages — the HTSSOP-56 footprint must be assigned manually. LCSC C98969 (DRV8301DCAR, HTSSOP-56) is the correct part.
- **Only one bootstrap diode placed** — one PMEG6020ER at BST_A only. BST_B and BST_C need diodes too (total: 3×). Additionally, PMEG6020ER is rated 20 V — insufficient for a 58.8 V bus. **Replace all three with DSS210 or equivalent ≥100 V Schottky in SOD-123 package.** (See §3.2)
- **Q2 has no part assigned** — Q2 is a generic `Q_NMOS_GSD` symbol with no value or LCSC part. This controls SENS_SUPPLY. Assign BSS138 (LCSC C112739) or 2N7002 (LCSC C8545).
- **CURR_FILTER_ON routing error** — the ON_OFF control pins of U2/U3/U4 (MC74VHC1GT66) connect to `PC13` via net `SENS_FILTERED`. However, on the MCU sheet, `CURR_FILTER_ON` is driven from `PD2` — a completely separate net. The filter enable signal from the MCU never reaches the analog switches. **Fix: connect U2/U3/U4 ON_OFF pins to the `CURR_FILTER_ON` net (PD2), not PC13.** (See §3.5)
- **EN_BUCK not driven by MCU GPIO** — the `EN_BUCK` net connects only to DRV8301 pin 55. The MCU sheet has no GPIO driving this net. The DRV8301 buck converter enable is effectively floating from software's perspective. **Fix: connect EN_BUCK to an appropriate MCU GPIO** (see §3.4).
- **Bulk capacitor C6 = 100 µF, no voltage rating** — for a 58.8 V bus, minimum 100 V rating required. Specify `100µF 100V`. C4/C5 = 2.2 µF (PVDD decoupling) ✓
- **Phase sense resistors R1/R3/R5 = "39k 60v"** — the `60V` notation likely refers to the resistor's voltage rating, not a net label. At 58.8 V bus the 60 V rated resistors are marginal. Use 100 V rated 0402 resistors.
- **MC74VHC1GT66 (U2/U3/U4)** placed ✓, but see CURR_FILTER_ON issue above.

---

### Sheet 4 — CAN (CAN.kicad_sch)

Contains: TCAN1051VDQ1 CAN transceiver.

**No issues found.**

- TCAN1051VDQ1: VCC → +5V, VIO → +3.3V ✓
- S pin → 10 kΩ to GND (normal mode, not silent mode) ✓
- 220 Ω series resistors on TXD/RXD (protection, not bus termination) ✓
- 4.7 µF + 100 nF decoupling on both rails ✓
- CAN bus termination (120 Ω) is expected at the cable connector, not on-board — confirm this in the connector spec.

---

### Sheet 5 — Power (Power.kicad_sch)

Contains: Phase A half-bridge (Q1 high-side, Q3 low-side), gate resistors R10/R11, shunt R9 (0.5 mΩ), AD8418 current sense amp U5, 100 nF bypass cap C7.

**This sheet is a work in progress. Phases B and C are not yet drawn.**

**Issues found from netlist analysis:**

- **Q1 drain (pin 1) is unconnected** — `unconnected-(Q1-D-Pad1)` net. The high-side MOSFET drain has no electrical connection in the netlist. Wire must reach the exact pin endpoint.
- **Q3 drain and source unconnected** — low-side MOSFET has floating drain and source in the netlist (`unconnected-(Q3-D-Pad1)`, `unconnected-(Q3-S-Pad3)`).
- **R11 pin 1 unconnected** — gate resistor for Q3 has a floating end in the netlist.
- **Gate signals GH_A/GL_A not connected to any MOSFET** — these signals come from DRV8301 but only terminate at the hierarchical label; no MOSFET gate pin is on the net.
- **Only one AD8418 placed (U5, Phase A)** — the MCU sheet expects three current sense outputs: `CURRENT_1`, `CURRENT_2`, `CURRENT_3`. U6 and U7 must be added for Phases B and C.
- **MOSFETs still use IRF7749L1TRPBF (40 V Vds)** — must be replaced before layout. See §3.1.

**What Phase B and C each need (template for completion in KiCad):**
- 2× N-MOSFET (high-side + low-side): same part as Phase A (after replacement)
- 2× 4.7 Ω gate resistors (R_Small, 0402)
- 1× 0.5 mΩ shunt (R9-style, 2512 package)
- 1× AD8418 current sense amp (SOIC-8)
- 1× 100 nF bypass cap (C_Small, 0603)
- Hierarchical labels: `GH_B`/`GL_B`/`SL_B` (or C) — connect to same-named labels in DRV8301 sheet
- Net labels: `C_B` / `C_C` for current sense outputs to MCU sheet
- Connect drain of high-side FET to `SUPPLY` power net
- Connect source of low-side FET to `GND` via shunt
- Phase output node between high-side source and low-side drain → hierarchical label `PH_B` / `PH_C`

---

## Critical Issues

### 3.1 MOSFET Voltage Rating — MUST FIX BEFORE LAYOUT

| Item | Value |
|---|---|
| Current part | IRF7749L1TRPBF |
| Vds absolute max | **40 V** |
| Max bus voltage | **58.8 V** (14S fully charged) |
| Transient headroom needed | Vbus + L·di/dt — easily exceeds 40 V |

**Action:** Replace all 6 MOSFETs with ≥80 V, low Rds(on) parts. BOM recommends IRFS4115TRLPBF (D2PAK, 150 V, 104 A, 10 mΩ, LCSC C2692945) — conservative choice with easy layout. The DirectFET_L8 footprint currently assigned must be changed to D2PAK (TO-263-2).

---

### 3.2 Bootstrap Diode Voltage Rating — MUST FIX

| Item | Value |
|---|---|
| Current part | PMEG6020ER |
| Vr | **20 V** |
| Bus voltage | **58.8 V** |

Bootstrap diode must block the full bus voltage. PMEG6020ER will fail.

**Action:** Replace 3× bootstrap diodes with DSS210 (SOD-123FL, 100 V, 2 A, LCSC C511868) or equivalent ≥100 V Schottky.

---

### 3.3 Power Sheet Incomplete

Phases B and C do not exist. This is the primary blocker for PCB layout. See Sheet 5 review above for the exact list of what to add per phase.

---

### 3.4 EN_BUCK Not Driven by MCU

`EN_BUCK` has no MCU GPIO connection. DRV8301 must have EN_BUCK asserted to enable its internal buck converter (which powers 5 V → logic supply). Without MCU control, either:
- Float it high via a pull-up (always enabled, no software control) — acceptable for now
- Connect to an MCU GPIO (e.g. PA8 or similar) — allows safe sequencing

**Action:** At minimum, add a 100 kΩ pull-up to PVDD on EN_BUCK so the buck converter starts at power-on. Add MCU GPIO drive if software sequencing is desired.

---

### 3.5 CURR_FILTER_ON Routing Error

MC74VHC1GT66 switches (U2/U3/U4) control the phase voltage sense filter. Their ON_OFF pins currently connect to `PC13` (`SENS_FILTERED` net). The correct signal is `CURR_FILTER_ON` from `PD2`. These are different nets — the filter is never enabled by the MCU.

**Action:** In DRV8301.kicad_sch, change the net on U2/U3/U4 ON_OFF pins from `SENS_FILTERED`/`PC13` to `CURR_FILTER_ON`. Verify `PD2` on the MCU sheet drives this net.

---

## Non-Critical Issues

### 4.1 BMI160 Missing

BMI160 is required for the VESC Balance App. Must be added to a sheet (new sub-sheet or directly on MCU sheet). Connections: SDA→PB7, SCL→PB6, INT1→MCU GPIO, CSB→VCC (I2C mode), 2.2 kΩ pull-ups on SDA/SCL. Supply decoupling: 100 nF + 10 µF on VDDIO.

### 4.2 DRV8301 Footprint

Symbol is 56-pin HTSSOP; footprint filter resolves to PVQFN-40. Override the footprint on the placed instance to `Package_SO:Texas_HTSSOP-56_6.1x9.7mm_P0.65mm` or equivalent.

### 4.3 Q2 Part Not Assigned

SENS_SUPPLY gate transistor Q2 has no part. Use BSS138 (SOT-23, N-channel, LCSC C112739) or 2N7002.

### 4.4 Shunt Resistor Power Rating

0.5 mΩ shunt at 50 A continuous: P = 50² × 0.0005 = 1.25 W per shunt. Most 2512 shunts are rated 1–2 W — marginal at continuous high current. Consider paralleling two shunts per phase (2× 1 mΩ → 0.5 mΩ effective, 2 W shared) if targeting >40 A continuous.

### 4.5 Phase Sense Resistors Voltage Rating

R1/R3/R5 "39k 60v" — at 58.8 V bus, a 60 V rated resistor has no margin. Use 100 V rated 0402 parts (standard spec in this value, negligible cost difference).

### 4.6 Annotation

All component references are `C?`, `R?`, `U?`, etc. — schematic is unannotated. Run KiCad's **Tools → Annotate Schematic** before generating any manufacturing outputs. The current netlist and BOM are based on unannotated refs.

---

## Items Verified ✓

- STM32F405RGTx LQFP-64 ✓
- SPI3 routed to DRV8301 (PC10/PB4/PB5 area) ✓ (net naming needs cleanup)
- VCAP_1/VCAP_2 = 2.2 µF ✓
- CAN transceiver TCAN1051VDQ1 wired correctly ✓
- NRF51822 EYSGJNZWY: single instance, correct module ✓
- AD8418 gain = 20 V/V, shunt 0.5 mΩ → 330 A full-scale, adequate resolution ✓
- Gate resistors 4.7 Ω on Phase A ✓
- 100 nF bypass cap C7 on Phase A ✓
- CONN_SW 1 kΩ PTC resistors for footpad detection ✓
- TC2117 LDO for 3.3 V ✓ (value field needs populating)
