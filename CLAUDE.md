# SBS_hw — Claude Session Briefing

This file is the single entry point for any Claude session on this project. Read it fully before touching any files.

---

## What This Project Is

**SBS_hw** is a custom BLDC motor controller PCB derived from the VESC 6 MK5 open-source reference. It targets two platforms:

| Platform | Motor | Battery | Max voltage |
|---|---|---|---|
| DIY Onewheel (e-skateboard) | PHUB-188 hub motor | 14S LiPo | 58.8V nominal, ~68V spike |
| Robot dog butler | X4114-370KV | 6S LiPo | 25.2V |

The **14S Onewheel is the critical sizing case** for all component voltage ratings. The 6S robot dog case is comfortably within limits once 14S is satisfied.

The design runs **VESC 6.x firmware** (target: `60_mk5` hardware). Do not assume firmware compatibility without checking `vesc_firmware_notes.md`.

Key differences from VESC 6 MK5:
- MOSFETs upgraded to IRFS4115TRLPBF (150V D2PAK) for 14S margin
- Bootstrap diodes upgraded to DSS210 (100V SOD-123F) for 14S margin
- TCAN1051VDQ1 CAN transceiver (3.3V I/O compatible) replacing TJA1051
- CONN_SW / footpad detection latch circuit (not in MK5)
- BMI160 IMU on PA15/PB2 software I2C (matches MK5)
- Voltage derating target: **≥1.5× nominal** for all passives on high-voltage rails

---

## File Map

```
design/                      ← KiCad schematic sheets (6 sheets)
  SBS_hw.kicad_sch           ← top-level: NRF51822, USB, connectors, LDO, EN_BUCK
  MCU.kicad_sch              ← STM32F405RGT6, crystal, decoupling
  DRV8301.kicad_sch          ← gate driver, analog switches, bootstrap diodes
  CAN.kicad_sch              ← TCAN1051VDQ1 transceiver
  BMI160.kicad_sch           ← IMU
  Power.kicad_sch            ← 3-phase bridge (Q2–Q7), current sense (AD8418), shunts

manufacturing/
  CLAUDE.md                  ← session context, folder structure, key decisions
  BOM.md                     ← full BOM with LCSC numbers and unit prices
  SBS_hw.net                 ← KiCad-exported netlist (regenerate after schematic changes)
  SBS_hw_schematic.pdf       ← exported schematic PDF
  motor_controller/
    ISSUES.md                ← board-level design issue tracker (MC-001..MC-015)
  parts/
    {Manufacturer}/{MPN}/
      {MPN}.md               ← design calcs, pinout verification, issues for this part
      {MPN}.pdf              ← datasheet PDF (link KiCad Datasheet field here)

schematic_review.md          ← full sheet-by-sheet review vs VESC 6 MK5
requirements.md              ← per-platform + merged requirements
battery_design.md            ← cell count, spot welding, BMS, fire safety
vesc_firmware_notes.md       ← VESC 6.x features, bring-up checklists

analysis/                    ← output of analyze_schematic.py (auto-generated, gitignored)
```

---

## Ground Rules for All Sessions

### KiCad files
- **Always run `pgrep -fil kicad` before editing any `.kicad_sch` file.** KiCad auto-saves and overwrites — editing while it's open corrupts the file.
- Commit each schematic sheet separately after modifying it.
- Never amend published commits; always create new ones.

### Issue tracking — two-tier system
- **Component issue** (voltage rating, package, calc verification) → `manufacturing/parts/{Vendor}/{MPN}/{MPN}.md`
- **Board-level issue** (missing nets, annotation, wiring errors, footprint mismatch) → `manufacturing/motor_controller/ISSUES.md`
- Reference issues as MC-NNN. The current highest is MC-015.

### Design calculations
- Every non-trivial component gets a folder: `manufacturing/parts/{Manufacturer}/{MPN}/`
- The `.md` file contains: part info, design calcs in context of this board, pinout verification, any issues
- The `.pdf` datasheet goes in the same folder and is linked in KiCad's Datasheet property
- At the end of the design phase, all these `.md` files are collated into a single `design_calcs.md` document — do not do this prematurely; individual files are the working copies

---

## LCSC Part Numbers — Workflow

Every schematic component must have an LCSC property (`Cxxxxxx`). This is required for JLCPCB assembly quoting.

### Adding LCSC properties via script (preferred for bulk)

KiCad `.kicad_sch` files are text. LCSC properties are inserted into each symbol block by UUID. The pattern used in this project:

```python
LCSC_PROP = (
    '\t\t(property "LCSC" "{lcsc}"\n'
    '\t\t\t(at 0 0 0)\n'
    '\t\t\t(effects\n'
    '\t\t\t\t(font\n'
    '\t\t\t\t\t(size 1.27 1.27)\n'
    '\t\t\t\t)\n'
    '\t\t\t\t(hide yes)\n'
    '\t\t\t)\n'
    '\t\t)\n'
)
```

Algorithm: for each UUID, `text.rfind('\t(symbol\n', 0, uuid_pos)` → depth-match `()`→ find `sym_end` → check `'"LCSC"' in block` (skip if exists) → insert before `text[sym_end-2:sym_end] == '\t)'`. See `/tmp/add_lcsc.py` for the complete working implementation (can be reconstructed from git history).

### Finding LCSC numbers

**Standard 0603 passives (JLCPCB Basic Parts — no assembly fee):**

| Value | LCSC | Value | LCSC |
|---|---|---|---|
| 0Ω | C21189 | 22Ω | C23345 |
| 100Ω | C22775 | 220Ω | C22962 |
| 1kΩ | C21190 | 2.2kΩ | C4190 |
| 3.3kΩ | C22978 | 10kΩ | C25804 |
| 15kΩ | C22809 | 18kΩ | C25810 |
| 39kΩ | C23153 | 56kΩ | C23206 |
| 220kΩ | C22961 | | |
| 15pF | C1644 | 1nF | C1588 |
| 6.8nF | C1631 | 22nF | C21122 |
| 100nF | C14663 | 220nF | C21120 |
| 2.2µF | C23630 | 4.7µF | C19666 |

Note: "2k2" and "2.2k" in schematics both mean 2.2kΩ → C4190. "2kΩ" is C22975 (different part). Do not confuse them.

**For non-standard parts:** Use the `lcsc` skill (`/lcsc` or invoke the LCSC skill). The jlcsearch API at `jlcsearch.tscircuit.com` is a community tool and goes down intermittently — if it fails, fall back to WebSearch + LCSC website WebFetch.

### Stock checking

Before locking a BOM for ordering, check current stock on LCSC for every part:
- Use the `lcsc` skill to check stock levels
- Flag any part with stock < 10× required quantity as a risk
- For parts with low stock, identify and document an alternative part number (same value/package, different manufacturer)
- The `bom` skill coordinates this across all parts — use it for full BOM stock sweeps

---

## Schematic Review Methodology

When reviewing this schematic:
1. Run `analyze_schematic.py` on the top-level sheet (it recurses into sub-sheets)
2. Cross-reference every IC pinout against its datasheet — the analyzer can produce plausible-looking but wrong pin-to-net mappings
3. Compare against VESC 6 MK5 PDF reference — SBS_hw is a derivative; deviations are intentional (documented above) or bugs
4. Voltage derating: all passives and semiconductors on SUPPLY/PVDD must be rated ≥ 100V (14S = 58.8V nominal, allow for inductive spikes)
5. Every component must have: Value, Footprint, Datasheet, LCSC properties populated

**Tools:**
```bash
# Check KiCad is closed first
pgrep -fil kicad

# Run schematic analyzer
python3 ~/.claude/skills/kicad/scripts/analyze_schematic.py \
  design/SBS_hw.kicad_sch --analysis-dir analysis/

# Regenerate netlist
/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli sch export netlist \
  --output manufacturing/SBS_hw.net --format kicadxml design/SBS_hw.kicad_sch
```

---

## Current State (as of 2026-06-29)

### Completed
- All 6 schematic sheets: footprints assigned to all components
- All 6 sheets: **136 LCSC part numbers** populated via UUID script (commit `9ac1957`)
- MOSFET replaced: IRFS4115TRLPBF (150V D2PAK), LCSC C2692945, all 6 instances
- Bootstrap diode BST_A: DSS210 (100V), LCSC C511868
- DRV8301 footprint: HTSSOP-56 (`Package_SO:Texas_HTSSOP-56_6.1x9.7mm_P0.65mm`)
- BMI160 sub-sheet added and connected
- Crystal: 8MHz, LCSC C5181477
- BSS138 (Q1 level shifter): LCSC C112739
- TC2117-3.3VDBTR: LCSC C98655
- Full design review completed: `schematic_review.md`
- Component calcs written for: IRFS4115TRLPBF, DSS210, DRV8301, AD8418, BMI160, TC2117

### Open blockers (must fix before PCB layout)

| ID | Issue | Action needed | Where |
|---|---|---|---|
| MC-002 | BST_B + BST_C bootstrap diodes missing | Place 2× DSS210 in DRV8301.kicad_sch | KiCad GUI |
| MC-003 | Bus electrolytics missing | Add 2× 470µF 100V (EEEHB2A471P, C131403) | KiCad GUI |
| MC-013 | BOOT0/PC13 wiring error | Move PC13 label to correct pin; fix BOOT0 pull-down | KiCad GUI |
| MC-015 | VDDA not connected | Wire VDDA + add 1µF + 10nF decoupling | KiCad GUI |

### Open — fix before fab

| ID | Issue | Action |
|---|---|---|
| MC-005 | Schematic unannotated (all R?, Q?, U?) | Run Annotate Schematic in KiCad |
| MC-009 | DRV8301 PVDD margin (1.2V headroom at 14S) | Add 60V TVS (SMAJ60CA) across SUPPLY bus |
| MC-010 | Phase sense 39kΩ resistors: 60V rated, no margin at 58.8V | Replace with 100V rated 39kΩ 0402 |
| MC-011 | SPI3 SDI/SDO net naming unclear | Verify SPI3_MOSI→DRV SDI, SPI3_MISO→DRV SDO |
| MC-014 | U2/U3/U4 lib_id still MC74VHC1GT66 | Update lib_id to SN74LVC1G66DCKR in KiCad |

### Components still without LCSC numbers (~19 parts)

These need manual lookup — jlcsearch and WebSearch were unable to confirm numbers during the 2026-06-29 session:

| Ref | Value | Package | Sheet |
|---|---|---|---|
| C18 | 120pF | 0603 | DRV8301 |
| C19, C21 | 15nF | 0603 | DRV8301 |
| C23 | 100µF | 1210 | DRV8301 |
| C33, C34 | 2.2µF | 0805 | DRV8301 |
| C35 | 100µF | electrolytic | DRV8301 |
| C52, C53, C54 | 15nF | 0603 | Power |
| R14, R15 | 1kΩ PTC | 0603 | SBS_hw |
| J6 | 1.27mm 1×5 SWD header | — | SBS_hw |
| JP* / J7, J8 | solder jumper / single-pin placeholders | — | SBS_hw |

---

## End-of-Design Collation

When all design work is complete:
1. Concatenate all `manufacturing/parts/**/*.md` files into `manufacturing/design_calcs.md`
2. Update `manufacturing/BOM.md` to reflect final component selections and current stock/pricing
3. Export schematic PDF: `kicad-cli sch export pdf design/SBS_hw.kicad_sch`
4. Regenerate netlist
5. Run `analyze_schematic.py` one final time and confirm no new findings
6. All open ISSUES.md items must be RESOLVED or explicitly deferred with rationale

Do not collate design_calcs.md prematurely — individual `.md` files are the working documents.

---

## Key LCSC Numbers (quick reference)

| Part | LCSC | Package |
|---|---|---|
| STM32F405RGT6 | C15742 | LQFP-64 |
| DRV8301DCAR | C98969 | HTSSOP-56 |
| AD8418ARMZ | C99534 | MSOP-8 |
| TCAN1051VDQ1 | C133788 | SOIC-8 |
| TC2117-3.3VDBTR | C98655 | SOT-223 |
| BMI160 | C94021 | LGA-12 |
| IRFS4115TRLPBF | C2692945 | D2PAK |
| DSS210 (bootstrap) | C511868 | SOD-123F |
| SN74LVC1G66DCKR | C113518 | SC-70-5 |
| BSS138 | C112739 | SOT-23 |
| EYSGJNZWY (NRF51822) | C2151959 | module |
| SMAJ5.0A TVS | C87074 | SMA |
| FCM1608KF-601T (ferrite) | C133937 | 0603 |
| 8MHz crystal | C5181477 | 3.2×2.5mm |
| 0.5mΩ shunt | C5375456 | 2512 |
| 4.7Ω gate resistor | C137962 | 0402 |
| 10kΩ NTC (Murata NCP18XH103F03RB) | C13564 | 0603 |
| 4.7µF 100V MLCC (TDK CGA6M3X7S2A475KT0Y3S) | C338088 | 1210 |
| 470µF 100V electrolytic (EEEHB2A471P) | C131403 | radial |
| USB Micro-B | C10418 | — |
| JST-PH 4P | C157926 | — |
| JST-PH 6P | C157920 | — |
| JST-PH 3P | C157929 | — |
| 2.54mm 1×40 header | C2337 | — |
