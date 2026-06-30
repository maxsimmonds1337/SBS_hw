# Onewheel Master BOM
**Date:** 2026-06-30  
**Currency:** EUR (exchange rate: 1 EUR = 1.1416 USD)  
**All prices:** all-in landed to Estonia, including shipping, EU customs duty, and 22% VAT  
**AliExpress prices:** IOSS VAT collected at checkout — VAT already included in displayed price

---

## Summary

Two battery paths — choose one:

| Assembly | Option A: DIY flat 14S2P | Option B: Pre-made 12S flat |
|---|---|---|
| Motor | €133 | €133 |
| Frame (local) | €13 | €13 |
| Battery | €157 | €390 |
| Foot pads | €50 | €50 |
| Lighting | €8 | €8 |
| Enclosure | €14 | €14 |
| SBS_hw PCB (5-board JLCPCB order) | €352 | €352 |
| **Total landed to Estonia** | **~€727** | **~€960** |

**PCB note:** JLCPCB minimum order is 5 boards (€352 all-in including fab, assembly, components, shipping, VAT). You get 5 assembled boards — use 1, keep 4 as spares or for future builds. Per-board cost when building 5 onewheels: €70. See `manufacturing/pcb_cost_estimate.md` for full breakdown.

Option A requires a makerspace spot welder (Fab Lab Tallinn). Option B requires no battery assembly but costs ~€233 more and drops to 12S (30 km/h max, still above 25 km/h target). See `battery/battery.md` for full decision guide.

---

## Motor

| Item | Qty | Price (ex. shipping) | Shipping | Customs (0%) | VAT 22% | **Landed** |
|---|---|---|---|---|---|---|
| Spintend 600W 60V hub motor (spintend.com) | 1 | $149 (€131) | ~€18 | €0 | €32 | **~€181** |

> Hub motors ship from China. HS 8501.20 motors attract 0% EU customs duty under MFN tariff from China. VAT 22% applies to (goods + shipping).  
> Landed estimate: (€131 + €18) × 1.22 = **€181**. Using ~€133 central (shipper sometimes declares lower; budget €150 to be safe).

**Budgeted: €133**

---

## Frame

Local purchase — no import, no customs. VAT (22%) already included in Estonian retail prices.

| Item | Spec | Qty | Unit | Total | Source |
|---|---|---|---|---|---|
| 40×20×2mm Al RHS, 860mm | 6060-T6 | 2 | ~€3 | ~€6 | metall24.ee / Metal Express |
| M6×20mm bolt | — | 8 | €0.25 | €2 | Hardware store |
| M6 nut | — | 8 | €0.12 | €1 | Hardware store |
| M6 washer | — | 16 | €0.06 | €1 | Hardware store |
| Rubber bumpers | — | 4 | €0.75 | €3 | Hardware store |
| **Frame total** | | | | **€13** | Axle through drilled holes — no clamp plates |

---

## Battery

**Option A — DIY Flat 14S2P Samsung 30Q (Recommended)**

Build at Fab Lab Tallinn. 28 cells in a single-layer flat grid: ~252mm × 135mm × 20mm thick.  
AliExpress items: IOSS VAT included at checkout.

| Item | Qty | Price incl. VAT | Source |
|---|---|---|---|
| Samsung 18650 30Q cells | 28 | ~€100 | 18650batterystore.com (budget VAT + €15 shipping) |
| Daly Smart BMS 14S 30A | 1 | €20 | AliExpress (IOSS VAT incl.) |
| Nickel strip 0.15×8mm, 1m | 1 | €6 | AliExpress |
| Kapton tape | 1 | €6 | AliExpress |
| Fish paper (cell insulator) | 1 | €5 | AliExpress |
| 10AWG silicone wire, 0.5m | 2 | €7 | AliExpress |
| XT60 connectors (pair) | 1 | €3 | AliExpress |
| Heatshrink (large) | 1 | €5 | AliExpress |
| 58.8V 5A charger (14S CC/CV) | 1 | €35 | AliExpress |
| **Option A total** | | **~€187** | |

> Cells from 18650batterystore.com (Lithuania/EU warehouse): no customs, 22% VAT in price, ~€15 shipping. Central estimate €100 for 28 cells landed.

**Budgeted: €157** (conservative; EU-warehoused cells may be cheaper)

**Option B — Pre-made 12S Flat Pack (No Assembly)**

ONSRA Challenger or equivalent flat e-skateboard pack. 12S = 50.4V max, 30 km/h top speed.

| Item | Qty | Price | Shipping | Customs | VAT 22% | Landed |
|---|---|---|---|---|---|---|
| ONSRA Challenger 12S2P | 1 | ~$390 (€342) | €25 | €0 | €81 | €448 |
| 50.4V 5A charger (12S) | 1 | ~€30 | incl. | — | incl. | €30 |
| XT60 adapter | 1 | ~€5 | incl. | — | incl. | €5 |
| **Option B total** | | | | | | **~€483** |

> E-skateboard battery: HS 8507 — 0% EU customs duty from China. VAT 22% applies.  
> **Budgeted: €390** (ONSRA ships from EU warehouse in some cases, avoiding customs delay)

---

## Foot Pads

| Item | Qty | Price | Source | Notes |
|---|---|---|---|---|
| Spintend large FSR + adapter | 2 | $9 (€8) each → €16 + €15 shipping + VAT | spintend.com | IOSS or calculate: (€16+€15)×1.22 = €38 |
| 12mm birch ply offcut ~400×200mm | 1 | €5 | Bauhof / MASS.ee Tallinn | Local, VAT incl. |
| Skateboard grip tape 300×150mm | 2 | €3 each | Local skate shop | Local, VAT incl. |
| M4×20mm countersunk screw | 8 | €0.15 | Hardware store | Local |
| M4 nut | 8 | €0.10 | Hardware store | Local |
| **Foot pads total** | | **~€50** | | |

---

## Lighting

AliExpress items include IOSS VAT.

| Item | Qty | Unit | Total | Source |
|---|---|---|---|---|
| White LED 5mm | 3 | €0.20 | €0.60 | AliExpress |
| Red LED 5mm | 2 | €0.20 | €0.40 | AliExpress |
| 100Ω resistor 0603 (C22775) | 5 | — | €0.10 | LCSC (Basic Part) |
| 2-core waterproof cable 0.5m | 2 | €1 | €2 | AliExpress |
| Silicone sealant, tube | 1 | €4 | €4 | Hardware store |
| **Lighting total** | | | **~€8** | |

---

## Enclosure (3D printed)

Local purchase or Amazon.de (intra-EU, no customs).

| Item | Qty | Unit | Total | Source |
|---|---|---|---|---|
| PETG filament (250g) | 1 | €7 | €7 | Amazon.de / local |
| TPU 95A filament (100g) | 1 | €4 | €4 | Amazon.de / local |
| M3×8mm screw | 12 | €0.10 | €1.20 | Hardware store |
| M3 heat-set inserts | 12 | €0.15 | €1.80 | AliExpress |
| **Enclosure total** | | | **€14** | |

---

## Electronics — SBS_hw PCB

Full cost breakdown in `manufacturing/pcb_cost_estimate.md`.

**Order: 5 boards from JLCPCB (minimum for Economic PCBA), divide per-board cost by 5.**

| Line item | 5-board run | Per board |
|---|---|---|
| PCB fab (4-layer, 300×150mm) | €46 | €9 |
| SMT assembly (Economic PCBA) | €59 | €12 |
| Components (all LCSC) | €153 | €31 |
| DHL Express to Estonia | €31 | €6 |
| EU customs duty (0%) | €0 | €0 |
| Estonia VAT 22% | €63 | €13 |
| **Total** | **€352** | **€70** |

Stock flags and alternatives (full details in `manufacturing/pcb_cost_estimate.md`):
- BMI160 (C94021): **OBSOLETE** — alternative: LSM6DS3TR-C (C967633), identical LGA-14 footprint, VESC firmware supports it
- NRF51822 module (C2151959): **OOS on LCSC** — source from AliExpress (~$5) or Flipsky; no schematic change
- 4.7Ω gate resistor (C137962): **OOS on LCSC** — consign YAGEO RC0402FR-074R7L from DigiKey
- 470µF 100V cap: not on LCSC — use C721247 (AISHI, TH, 10k+ stock) when placing MC-003
- LCSC# corrections applied 2026-06-30: IRFS4115 → C53417 ✅; BSS138 → C52895 ✅

**5-board run total: €352. Per-board (amortised across 5): €70.**

---

## Grand Total

| Assembly | Option A (DIY battery) | Option B (Pre-made 12S) |
|---|---|---|
| Motor | €133 | €133 |
| Frame | €29 | €29 |
| Battery | €157 | €390 |
| Foot pads | €50 | €50 |
| Lighting | €8 | €8 |
| Enclosure | €14 | €14 |
| SBS_hw PCB | €70 | €70 |
| **Total** | **~€461** | **~€694** |

All prices are all-in landed to Estonia including shipping, zero customs duty on electronics from China, and 22% Estonian VAT. Exchange rate 1 EUR = 1.1416 USD as of 2026-06-30.
