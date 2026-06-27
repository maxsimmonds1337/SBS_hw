# Motor Controller — Board-Level Issues

Issues here are at the board/schematic level, not tied to a single component. Component-specific issues are in `../parts/{Vendor}/{MPN}/{MPN}.md`.

Status: `[OPEN]` / `[IN PROGRESS]` / `[RESOLVED]`

---

## Critical — Must Fix Before Layout

### MC-001 MOSFET Footprint Change [IN PROGRESS]
**All 6 MOSFETs (Q1, Q3-Q7) updated in schematic** to IRFS4115TRLPBF with `Package_TO_SOT_SMD:TO-263-2` footprint (2026-06-27). The symbol lib_id (`Transistor_FET:IRF7748L1`) remains — pin ordering (G=1, D=2, S=3) is compatible with TO-263-2.

**Still needed:**
- Add LCSC property `C2692945` to each MOSFET instance in KiCad
- Verify pin 1/2/3 assignment in the KiCad footprint matches D2PAK physical pinout (Gate=1, Drain tab=2, Source=3)
- PCB layout: each MOSFET needs ≥ 50 cm² copper pour on the tab pad. Tight half-bridge loop (< 1 cm²) to keep L_stray < 30 nH.

### MC-002 Bootstrap Diodes Missing/Wrong [PARTIALLY RESOLVED]
- BST_A: ~~PMEG6020ER (20V)~~ → DSS210 (100V) set in DRV8301.kicad_sch 2026-06-27. Footprint: `Diode_SMD:D_SOD-123F`.
- BST_B: **STILL MISSING** — must be added to DRV8301.kicad_sch in KiCad (requires placement)
- BST_C: **STILL MISSING** — must be added to DRV8301.kicad_sch in KiCad (requires placement)

Cathode of each diode → BST_x pin (DRV8301 pins 48/38/49 for A/B/C). Anode → VBOOST (+5V from DRV8301 buck). See `../parts/Infineon/DSS210/DSS210.md`.

### MC-003 Missing SUPPLY Bus Electrolytic Capacitors [OPEN]
VESC 6 MK5 reference uses 680 µF × 2 (1.36 mF total) at 63 V on SUPPLY. SBS_hw has only ~28 µF of MLCCs.

At 14S (58.8 V), the bus capacitors must be rated ≥ 100 V. Add to Power sheet or top-level:
- 2× 470 µF (min) or 680 µF 100 V electrolytic, directly across SUPPLY and GND
- Suggestion: EEEHB2A471P (Panasonic, 470 µF, 100 V, radial, LCSC C131403)
- Place as close as possible to the SUPPLY bus connector pads

### MC-004 DRV8301 Footprint Override Not Applied [RESOLVED 2026-06-27]
~~Footprint was blank.~~ Set to `Package_SO:Texas_HTSSOP-56_6.1x9.7mm_P0.65mm` via text edit.

---

## Important — Fix Before Fab

### MC-005 Schematic Unannotated [OPEN]
All component references are `C?`, `R?`, `Q?`, `U?`. Run **Tools → Annotate Schematic** in KiCad before generating BOM or sending for fab. Current netlist and BOM reflect unannotated refs (functionally correct but confusing).

### MC-006 TC2117 Value Field Blank [RESOLVED 2026-06-27]
~~U1 Value = `~`.~~ Set Value=`TC2117-3.3VDBTR`, Footprint=`SOT-223-3_TabPin2`. LCSC C98655 still needs to be set as a property in KiCad.

### MC-007 Crystal No Frequency Value [RESOLVED 2026-06-27]
~~Y? on MCU sheet has no frequency in the Value field.~~ Fixed: Value set to `8MHz`, footprint set to `Crystal:Crystal_SMD_3225-4Pin_3.2x2.5mm` (matches LCSC C5181477 SOSET 8MHz 20pF 10ppm, per BOM.md).

### MC-008 Q2 (SENS_SUPPLY) No Part Assigned [RESOLVED 2026-06-27]
~~Q2 had generic symbol with no part.~~ Set Value=`BSS138`, Footprint=`SOT-23`. LCSC C112739 still needs to be set as a property.

### MC-009 DRV8301 PVDD Margin [OPEN]
PVDD_max = 60V. At 14S (58.8V): 1.2V headroom.

Mitigation already present: 4.7µF 100V MLCCs on PVDD, bulk caps (MC-003) will reduce spikes.
Additional: add 60V TVS (SMAJ60CA, bidirectional) across SUPPLY bus for overvoltage protection.

### MC-010 Phase Sense Resistors Voltage Rating [OPEN]
R1/R3/R5 "39k 60V" — at 58.8V bus this 60V rating has no margin. Replace with 100V rated 39kΩ 0402 resistors (standard availability, negligible cost difference).

---

## Lower Priority

### MC-011 SPI3 Net Naming [OPEN]
SBS_hw MCU sheet labels SPI3 SDI/SDO as "wrongly named" (noted in schematic). The MCU sheet note says these will be copied from the original VESC schematic. Verify before layout that SPI3_MOSI connects to DRV8301 SDI and SPI3_MISO connects to DRV8301 SDO.

### MC-012 AVDD (STM32 Pin 13) Decoupling [OPEN → SEE MC-015]
Promoted to MC-015 (VDDA Not Connected) after direct-read analysis confirmed VDDA has no wire connection at all.

### MC-013 BOOT0/PC13 Wiring Error [OPEN]
Direct-read analysis of MCU.kicad_sch (2026-06-27) found:
- The hierarchical label `PC13` is placed at (132.08, 133.35), which matches **BOOT0 (pin 60)** at y_sym=+35.56, not the actual PC13 GPIO.
- The actual PC13 GPIO (pin 2, y_sym=-35.56, abs y=62.23) connects to GND via a wire and a 100n cap — wrong for a GPIO output.

**Needed in KiCad:**
1. Move the `PC13` hierarchical label from y=133.35 to y=62.23 (the actual PC13 GPIO pin position).
2. Fix BOOT0 (y=133.35): either tie directly to GND, or add a 10kΩ pull-down to GND (pull-down preferred — allows DFU entry via BOOT0 high).
3. Remove the GND direct-tie from the PC13 GPIO pin once the label is moved.

### MC-014 U2/U3/U4 Symbol Update [PARTIALLY RESOLVED 2026-06-27]
U2/U3/U4 Value set to `SN74LVC1G66DCKR` and Footprint set to `SOT-353_SC-70-5`.
lib_id NOT changed (stays as `Analog_Switch:MC74VHC1GT66`) — the MC74VHC1GT66 and SN74LVC1G66 have identical pinout in SOT-353/SC-70-5, so the symbol is functionally compatible. Changing lib_id in a text editor risks breaking wiring; do this in KiCad if the custom 74xGxx library is available.

Still needed: U8/U9/U10 on Power.kicad_sch need same Value/Footprint update.

### MC-015 VDDA (Pin 13) Not Connected [OPEN]
Direct-read analysis (2026-06-27): VDDA (STM32 pin 13, abs position (163.195, 143.51)) has no wire connection. All other bottom-side VDD pins (19, 32, 48, 64) have decoupling caps to GND, but VDDA has nothing.

**Needed in KiCad:**
1. Add wire from VDDA pin to the +3.3V supply rail (same net as the other VDD pins, or via the `Vcc` hierarchical label path).
2. Add 1µF decoupling cap + 10nF decoupling cap between VDDA and GND (standard STM32 application circuit). These are the `AVDD bypass caps` called out in the STM32 datasheet.
3. Optionally add a ferrite bead between VDD and VDDA for better analog isolation (VESC 6 MK5 does this).

---

## Resolved

### MC-R01 Power Sheet Incomplete ~~[OPEN]~~ → RESOLVED ✓
Phases B and C completed in commit `3adaedc`.

### MC-R02 CURR_FILTER_ON Routing ~~[OPEN]~~ → FALSE FINDING (RETRACTED)
U2/U3/U4 → SENS_FILTERED/PC13 is correct. CURR_FILTER_ON/PD2 → U8/U9/U10. Matches VESC 6 MK5.

### MC-R03 BMI160 Missing ~~[OPEN]~~ → RESOLVED ✓
BMI160 sub-sheet added in commit `3adaedc`. Connections match MK5.

### MC-R04 MOSFET Voltage Rating ~~[OPEN]~~ → PARTIALLY RESOLVED
Schematic values updated to IRFS4115TRLPBF (150V, D2PAK) — 2026-06-27.
BST diodes and PCB layout work still required (see MC-001, MC-002).
