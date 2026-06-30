# Frame Sub-Assembly

## Design Concept

A onewheel frame must:
- Mount the motor axle rigidly (no flex — affects balance control)
- Provide a flat platform for each foot (one forward, one aft of the wheel)
- Be light enough to keep total weight under ~7 kg finished
- Be stiff enough that it doesn't flex under a 100 kg rider load

**Two rails of aluminium rectangular hollow section (RHS)** running front-to-back, one on each side of the motor. The motor axle passes through holes drilled in the rail walls — no machining, no clamp plates.

```
Side view:
   ┌──────────────────────────────────────┐
   │  FOOT PAD  │   MOTOR   │  FOOT PAD  │
   │  <~340mm>  │  <~180mm> │  <~340mm>  │
   └──────────────────────────────────────┘
                    ← 860mm total →

Cross-section (end-on, one rail):
   ┌──────────────────────┐  ← 40mm wide (footpad surface)
   │                      │
   │                      │  20mm deep
   │        ●             │  ← 14mm axle hole drilled through both walls
   │                      │
   └──────────────────────┘
        2mm wall all round
```

The motor axle (14mm diameter) simply passes through 14mm holes drilled through the 2mm side walls of each rail. No milling, no custom clamp plates. This is the same approach used by ByteSizeEngineering's open-source Openwheel build (which uses 1"×2" US equivalent).

---

## Why Rectangular Tube Over L-Profile

| | L-profile 40×40×4mm | RHS 40×20×2mm |
|---|---|---|
| Section modulus | 1.2 cm³ | 1.44 cm³ |
| Torsional stiffness | Low (open section) | High (closed section) |
| Motor mount | Milled slot + clamp plate | Drill 14mm hole — done |
| Weight/m | 0.83 kg/m | 0.46 kg/m |
| Footpad surface | 40mm flat face | 40mm flat face |
| Source (Tallinn) | Metal Express | metall24.ee / Metal Express |

Closed section (tube) is significantly stiffer in torsion — important for balance control since any twist in the rail changes foot angle relative to the IMU.

---

## Dimensions

### Board length
- Spintend hub motor: ~150mm tire width, axle extends ~15mm each side
- Rider foot length: ~280mm (size 42 shoe)
- Footpad zone: 340mm each side
- Total: 340 + 180 (motor) + 340 = **860mm**

### Rail spec — 40×20×2mm EN AW-6060 T6 aluminium RHS

Stress check for 90 kg rider on 340mm cantilever:
- Bending moment per rail: 90 kg × 9.8 × 0.34m / 2 = **150 N·m**
- Section modulus (40mm wide, 20mm deep, 2mm wall): **1,438 mm³**
- Bending stress: 150,000 / 1,438 = **104 MPa**
- EN AW-6060 T6 yield strength: 170 MPa
- Safety factor: **1.63× — adequate**

Weight: ~0.46 kg/m × 0.86m × 2 rails = **0.79 kg for both rails**  
(vs 1.67 kg for L-profile — saves ~880g)

---

## Motor Axle Mounting

1. Mark the axle centreline at the midpoint of each rail (430mm from each end)
2. Drill a **14mm hole** through both walls of the 20mm face (top and bottom wall of the 20mm side)
3. Pass the motor axle through both rails
4. Secure with the axle nuts that come with the motor

No machining needed. The 14mm axle fits cleanly through 14mm holes in the 2mm walls.

**Tip:** Clamp both rails together when drilling so the holes are perfectly aligned.

---

## Where to Source in Tallinn

**metall24.ee (CORM OÜ)** — confirmed product  
Product: [Al. nelikanttoru 40×20×2mm L=6m](https://www.metall24.ee/tooted/al.-nelikanttoru-40x20x20--l-6m/al.-nelikanttoru)  
Address: Suur-Sõjamäe 13, Tallinn (Lasnamäe area)  
Hours: Mon–Fri 09:00–17:00  
Phone: +372 56 211 026  
Ask for: 2× 860mm cut from 6m stock (they cut to length). Price ~€2–3/m.

**Metal Express Estonia** — may also stock, confirm by email  
Email: metalexpress@metalexpress.ee / Tel: +372 6191 070  
Already confirmed stock of 40×40×4mm L-profile; ask specifically for 40×20×2mm RHS.

**K-rauta Tallinn** — check in-store, online range limited to 25×25×2mm  
Tondi store (closest to Pelgulinn): Tammsaare tee 49 — ~1.5 km from Tina 23, walkable or bus 33  
Haabersti store: Paldiski mnt 108a — tram 2/4 along Paldiski maantee, ~15 min  
[K-rauta tubes category](https://www.k-rauta.ee/c/ehitus-ja-remont/metall-plastiktooted-torud/torud/bki)

---

## Cut List

| Part | Profile | Length | Qty | Notes |
|---|---|---|---|---|
| Main rails | 40×20×2mm 6060-T6 RHS | 860mm | 2 | Cut from 6m stock |

Total aluminium: 1.72m of 40×20×2mm RHS

---

## Hardware BOM

| Item | Qty | Unit | Total | Source |
|---|---|---|---|---|
| 40×20×2mm Al RHS, 860mm | 2 | ~€3 | ~€6 | metall24.ee / Metal Express |
| M6×20mm bolt | 8 | €0.25 | €2 | Hardware store |
| M6 nut | 8 | €0.12 | €1 | Hardware store |
| M6 washer | 16 | €0.06 | €1 | Hardware store |
| Rubber feet/bumpers | 4 | €0.75 | €3 | Hardware store |
| **Total** | | | **~€13** | Saves ~€16 vs L-profile (no clamp plates needed) |
