# Enclosure Sub-Assembly

## What Needs Covering

1. **Electronics bay** — SBS_hw PCB + battery (or battery separate bay)
2. **Side covers** — decorative/protective covers over the motor hub gap
3. **Footpad surfaces** — grip surface over the rail / FSR sensors
4. **Nose/tail** — front and rear end caps (also house LED lights)

---

## Material Recommendations

| Part | Material | Why |
|---|---|---|
| Electronics bay cover | PETG | UV + impact resistant, tight tolerances |
| Footpad top surface | TPU (95A shore) | Flexible, grippy, absorbs impact |
| Side covers | PETG or ABS | UV resistant, paintable |
| Nose / tail caps | PETG | Houses LEDs, needs some flexibility for LED diffusion |

---

## 3D Print Parameters

**PETG parts:**
- Layer height: 0.2mm
- Infill: 40% gyroid
- Walls: 4 perimeters
- Print temperature: 240°C / 85°C bed
- No supports where avoidable (design for print orientation)

**TPU footpad:**
- Layer height: 0.25mm
- Infill: 30% gyroid
- Print speed: 25–30mm/s (slower for TPU)
- No cooling fan (first few layers especially)

---

## Filament Estimate

| Part | Material | Est. weight |
|---|---|---|
| Electronics bay cover | PETG | ~80g |
| Side covers × 2 | PETG | ~120g |
| Footpad surfaces × 2 | TPU 95A | ~100g |
| Nose + tail caps | PETG | ~60g |
| **Total** | | **~360g** |

At ~€25/kg for PETG and ~€30/kg for TPU:
- PETG: 260g × €0.025 = **€6.50**
- TPU: 100g × €0.030 = **€3.00**
- **Total filament: ~€10**

---

## Design Notes (to be done in CAD — OnShape preferred)

- Electronics bay dimensions: target ~SBS_hw PCB size + 10mm clearance each side
  - SBS_hw PCB is 4-layer, approx 80×120mm (estimate; confirm from PCB layout)
  - Bay: ~100×140×40mm
- Footpad: 300×140mm × 8mm thick, textured top surface, 15mm × 5mm recess for FSR sensor
- Side covers: fit around 155mm motor hub, snap onto rail or M3 screw mount
- Nose/tail: rounded profile, 3× LED holes (Ø5mm) in nose, 2× in tail

---

## Bill of Materials

| Item | Qty | Unit price | Total | Source |
|---|---|---|---|---|
| PETG filament (250g) | 1 | €7 | €7 | Local 3D print store / Amazon |
| TPU 95A filament (100g) | 1 | €4 | €4 | Local 3D print store |
| M3×8mm screws (for cover attachment) | 12 | €0.10 | €1.20 | Hardware store |
| M3 heat-set inserts | 12 | €0.15 | €1.80 | AliExpress |
| **Total** | | | **~€14** | |
