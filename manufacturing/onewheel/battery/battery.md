# Battery Sub-Assembly

## Voltage Target

SBS_hw is designed for **14S Li-ion = 58.8V nominal (60.2V full, 42V cutoff)**.

Primary path: DIY flat 14S2P cylindrical pack built at a makerspace.
Fallback: pre-made 12S flat e-skateboard pack (no assembly required).

---

## Form Factor Constraint

The battery mounts **under one footpad zone**, between the two L-rails.

Available space per footpad zone:
- Length: ~340mm (footpad zone length)
- Width: ~130mm (between L-rails, inside dimension)
- Height: ~72mm (rail bottom to ground — must leave ~35mm ground clearance for grass/curbs)

**E-bike bottle packs (e.g. EM3ev Rhino: 81mm tall) do not fit.** The correct form factor is a flat single-layer cylindrical pack or a flat e-skateboard pack, both ~18–30mm thick.

---

## Option A — DIY Flat 14S2P (Recommended)

### Cell: Samsung INR18650-30Q

The **30Q** is Samsung's high-drain 18650 cell:
- **18650** = size: 18mm diameter, 65mm long
- **30** = 3000mAh capacity
- **Q** = high-drain series
- Discharge: 15A continuous
- Internal resistance: ~23mΩ
- Per-cell PTC protection (resettable thermal fuse in top cap — last-resort backstop if BMS fails)

Buy from a reputable supplier (18650batterystore.com, liionwholesale.com) — not AliExpress. Genuine Samsung cells can be verified with a capacity test; fakes fail immediately.

### Pack Configuration: 14S2P

- 14 groups in series (14S = 58.8V nominal)
- 2 cells in parallel per group (2P)
- Total cells: **28**

### Flat Layout

Cells arranged in a single layer, long axis oriented across the board width:

```
Board length →
┌─────────────────────────────────────────────────────┐
│ ○ ○ ○ ○ ○ ○ ○  (row 1: 14 cells, one per group)    │  65mm
│ ○ ○ ○ ○ ○ ○ ○  (row 2: 14 cells, parallel to row 1)│  65mm
└─────────────────────────────────────────────────────┘
  ←────── 14 × 18mm = 252mm ──────→
```

Pack dimensions (assembled with nickel + wrap):
- Length: 252mm (along board length direction)
- Width: ~135mm (2 × 65mm cell length + 5mm wrap)
- Thickness: **~20mm** (one cell diameter + nickel + heatshrink)

Fits comfortably in the footpad zone (252mm < 340mm, 135mm ≈ 130mm rail spacing).

### Pack Specs

| Parameter | Value |
|---|---|
| Voltage nominal | 50.4V (14 × 3.6V) |
| Voltage full charge | 58.8V (14 × 4.2V) |
| Voltage cutoff | 39.2V (14 × 2.8V) |
| Capacity | 6Ah (2 × 3Ah) |
| Energy | 353Wh |
| Max discharge | 30A (2 × 15A) |
| Weight (cells only) | ~490g |
| Estimated range | ~3–5 km |

Range is modest at 2P — adequate for recreational sessions. Upgrade to 14S3P (42 cells, 3 rows) if more range is needed, though this increases pack thickness to ~38mm.

### BMS

**Daly Smart BMS 14S 30A** — unchanged from original selection.
- Over/under voltage per cell, over-current, short circuit, temperature cutoff
- Bluetooth monitoring (cell voltages visible in app)
- Price: ~$20

### Where to Build

Spot welding is required (nickel strip to cell terminals). This cannot be done safely in a flat — use a makerspace.

**Fab Lab Tallinn** (Telliskivi Creative City) — proper fab lab with equipment access. Check [fablabee.ee](https://fablabee.ee/) for membership/access.

### Assembly Notes

1. Tape all cell ends with Kapton before placing in the jig — prevents accidental shorts
2. Use a quality spot welder (Kweld ~€80, or makerspace unit). Cheap welders produce cold welds → high resistance → heat → fire risk
3. Verify each weld with a multimeter (resistance check) before moving on
4. Insulate between cell rows with fish paper
5. Wrap finished pack in heatshrink
6. BMS goes on top of pack within the enclosure

### Safety

A carefully built DIY pack with genuine cells is comparable in safety to a mid-range commercial pack:

| Safety layer | DIY pack | Commercial pack |
|---|---|---|
| Cell quality | Samsung 30Q (genuine, verified) | Varies — good packs use same cells |
| Per-cell protection | PTC in each cell | Same |
| BMS protection | Daly 14S (over-V, under-V, OCP, SCP, temp) | Equivalent |
| Weld quality | Depends on builder + welder | Factory laser welded, QC tested |
| Factory QC | None — self-QC | Yes |

The main risk difference is weld quality and self-QC. Both are mitigated with a good spot welder and methodical resistance checks. The cell chemistry and BMS protection are equivalent to any quality pack.

### Bill of Materials

| Item | Qty | Unit | Total | Source |
|---|---|---|---|---|
| Samsung 18650 30Q cells | 28 | $3.50 | $98 | 18650batterystore.com |
| Daly Smart BMS 14S 30A | 1 | $20 | $20 | AliExpress |
| 0.15mm × 8mm nickel strip, 1m | 1 | $5 | $5 | AliExpress |
| Kapton tape | 1 roll | $5 | $5 | AliExpress |
| Fish paper (insulator sheet) | 1 | $4 | $4 | AliExpress |
| 10AWG silicone wire, 0.5m | 2 | $3 | $6 | AliExpress |
| XT60 connector pair | 1 | $2 | $2 | AliExpress |
| Heatshrink (large diameter) | 1 | $4 | $4 | AliExpress |
| 58.8V 5A charger (14S CC/CV) | 1 | $35 | $35 | AliExpress |
| **Total** | | | **~$179** | excl. makerspace time |

---

## Option B — Pre-made 12S Flat Pack (No Assembly)

If makerspace access is not available, a flat e-skateboard pack avoids all assembly risk.

### Voltage compromise: 12S

12S = 12 × 4.2V = **50.4V full charge** (vs 58.8V at 14S).

Implications:
- Top speed: 12.39 KV × 50.4V = **30 km/h** — still above 25 km/h target ✓
- SBS_hw: no hardware changes; update VESC voltage cutoffs only
- DRV8301 PVDD headroom: 60 − 50.4 = **9.6V** — comfortable (fixes MC-009)
- Charger: 50.4V 12S charger required instead of 58.8V

### Suitable Products

| Pack | Config | Dimensions | Price | Notes |
|---|---|---|---|---|
| ONSRA Challenger | 12S2P 21700 | ~290×150×25mm | ~$390 | Fits in one footpad zone |
| DIYElectricSkateboard 12S3P | 12S3P | ~460×130×25mm | $430 | Too long for single zone — spans motor area |

The ONSRA Challenger at 290mm fits within one footpad zone (340mm available). Confirmed flat format, ~25mm thick.

### Bill of Materials

| Item | Qty | Unit | Total | Source |
|---|---|---|---|---|
| ONSRA Challenger 12S2P | 1 | ~$390 | $390 | onsra.com |
| 50.4V 5A charger (12S CC/CV) | 1 | $30 | $30 | AliExpress |
| XT60 adapter (if needed) | 1 | $5 | $5 | AliExpress |
| **Total landed** | | | **~$425–450** | incl. VAT + shipping to Estonia |

---

## Decision Guide

| | Option A: DIY 14S2P | Option B: Pre-made 12S |
|---|---|---|
| Cost | ~$179 | ~$425–450 |
| Assembly | Spot weld at makerspace | Mount + wire only |
| Voltage | 14S — correct for SBS_hw | 12S — needs VESC cutoff change |
| Top speed | 35 km/h (VESC-limited) | 30 km/h (motor limit at 12S) |
| Thickness | ~20mm | ~25mm |
| Length | 252mm | ~290mm |
| Range | ~3–5 km | ~4–6 km |
| Safety | Depends on build quality | Factory QC'd |
| MC-009 fix | No (still 14S) | Yes (12S gives 9.6V headroom) |

---

## Charging

- Charge through a separate DC barrel or XT60 port on the frame enclosure — do not charge through the VESC
- 14S charger: 58.8V CC/CV, 5A recommended (2A minimum)
- 12S charger: 50.4V CC/CV, 5A recommended
- Charge time at 5A: 14S2P (6Ah) → ~1.2 hours; 12S2P (~10Ah) → ~2 hours
- Never leave charging unattended until you have several cycles of confidence in the pack
