# Manufacturing Folder — Structure and Context

## What This Folder Contains

This folder holds all design review, component documentation, and issue tracking for the SBS_hw project. It is intended to be the single source of truth for manufacturing readiness, and is structured so that a fresh Claude session (or engineer) can orient quickly.

## Folder Structure

```
manufacturing/
  CLAUDE.md           ← this file: system overview + session context
  BOM.md              ← full bill of materials with LCSC part numbers and prices
  SBS_hw.net          ← KiCad-exported netlist (regenerate with kicad-cli after schematic changes)
  SBS_hw_schematic.pdf ← exported schematic PDF
  parts/              ← per-component documentation and design calculations
    {Manufacturer}/
      {MPN}/
        {MPN}.md      ← part info, design calcs, issues specific to this part
        {MPN}.pdf     ← datasheet PDF (download and place here; link KiCad Datasheet field to this)
  motor_controller/
    ISSUES.md         ← board-level design issues (not component-specific)
  ../schematic_review.md ← full schematic review document (in repo root)
```

## Issue Tracking Philosophy

Issues are tracked at the level where they live:
- **Component issue** (wrong voltage rating, wrong package, needs calc verification) → in `parts/{Vendor}/{MPN}/{MPN}.md` under an "Issues / Action Required" section.
- **Motor controller board issue** (missing bulk caps, net routing, annotation, footprint mismatch) → `motor_controller/ISSUES.md`
- **Systemic / architectural issue** (design intent, target voltage, topology choices) → `schematic_review.md` or noted at the top of this file

## Design Overview

**SBS_hw** is a custom VESC 6 MK5 derivative designed for a 14S LiPo (58.8V) Onewheel-style e-skateboard and potentially also for a robot dog butler application.

Key differences from VESC 6 MK5:
- Target voltage: **14S (58.8V)** vs MK5 design intent of ~12S (50.4V max). This requires component upgrades throughout.
- Added **CONN_SW / footpad detection** latch circuit (SHUTDOWN/EN_BUCK/Zener) not present in MK5.
- **TCAN1051VDQ1** CAN transceiver (3.3V I/O compatible) replacing TJA1051.
- **BMI160** IMU on PA15/PB2 software I2C — matches MK5.

## Review Trigger (2026-06-27)

This review was triggered with the following design requirements:

> Check every component for appropriate value, voltage rating, package size, and LCSC part number. Create the `manufacturing/parts/{vendor}/{mpn}/` folder structure containing a markdown file with all relevant info, a PDF datasheet (linked so the 'D' key opens it in KiCad), and all design calculations in context of this design. Confirm pinouts (schematic and footprint) against datasheets. Understand and sanity-check the higher-level architecture (60V LiPo input, 3-phase bridge, etc.). Compare this design to the VESC 6 MK5. Verify suitability for PSU input and X4114 motor (robot dog butler). Generate the netlist to review connectivity. Update MOSFETs to 80V-rated ones with all relevant calcs: worst-case thermal, switching losses, snubbers if needed, flyback diode adequacy. All component content lives in `manufacturing/parts/`. Run additional checks including SPICE if warranted.

## Key Decisions Made This Session

1. **MOSFET replacement:** IRF7749L1TRPBF (40V DirectFET L8) → IRFS4115TRLPBF (150V D2PAK). Minimum 100V required for 14S due to inductive transient spikes. Full calcs in `parts/Infineon/IRFS4115TRLPBF/`. Schematic updated 2026-06-27.

2. **Bootstrap diodes:** PMEG6020ER (20V, BST_A only) → DSS210 (100V, SOD-123FL, all 3 phases). Full analysis in `parts/Infineon/DSS210/`.

3. **Schematic review §3.5 retracted:** U2/U3/U4 (phase voltage sense filter) connecting to SENS_FILTERED/PC13 is CORRECT per VESC 6 MK5. Previously flagged as a wiring error — confirmed false finding.

4. **SPICE skipped:** No SPICE simulator installed (`ngspice`, `ltspice`, `xyce` all absent). Install ngspice if simulation is needed.

## Open Items (see motor_controller/ISSUES.md for full tracked list)

Critical blockers before PCB layout:
- Bootstrap diodes: BST_B and BST_C not yet placed in DRV8301.kicad_sch; BST_A still PMEG6020ER
- Missing SUPPLY bus electrolytic capacitors (470-680µF 100V × 2)
- DRV8301 footprint override not yet applied (still wrong in schematic)
- Schematic unannotated (all R?, Q?, U?)

## How to Regenerate the Netlist

```bash
/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli sch export netlist \
  --output manufacturing/SBS_hw.net \
  --format kicadxml \
  design/SBS_hw.kicad_sch
```

## How to Run the Schematic Analyzer

```bash
python3 ~/.claude/skills/kicad/scripts/analyze_schematic.py \
  design/SBS_hw.kicad_sch \
  --analysis-dir analysis/
```
