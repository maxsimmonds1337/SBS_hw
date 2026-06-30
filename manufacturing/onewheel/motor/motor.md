# Motor Sub-Assembly

## Motor Selection

### Why hub motor, not belt drive?
Hub motors have no chain/belt to break, are sealed, and simpler mechanically. For a onewheel, the motor IS the wheel — no gearing required. Trade-off: hub motors have lower torque than geared motors, but at 14S with a good VESC tune this is fine for recreational use.

---

## Understanding KV Ratings (and why it matters for our build)

**KV** = motor RPM per volt of supply, at no load. It is the single most important number for choosing a motor at a given voltage.

```
No-load RPM = KV × Battery Voltage
```

For a 10-inch wheel (diameter = 254mm, circumference = 0.798m):
```
Top speed (m/s) = No-load RPM × circumference / 60
               = KV × V × 0.798 / 60
```

At 14S (58.8V nominal):

| KV   | No-load RPM | Top speed | Comment |
|------|-------------|-----------|---------|
| 8    | 470         | 6.3 m/s  = 22.6 km/h | Too slow |
| 12   | 706         | 9.4 m/s  = 33.8 km/h | ✓ Good — Spintend 600W |
| 15   | 882         | 11.7 m/s = 42.2 km/h | Fast but manageable |
| 21   | 1235        | 16.5 m/s = 59.3 km/h | Dangerously fast for a onewheel |

**Verdict:** KV of 10–15 is the target for a 14S onewheel aiming for 30–40 km/h. The VESC can software-limit ERPM below the motor's max, so you can always go slower but not faster. Choose a motor whose natural KV lands in this range at 58.8V.

**Lower KV = more torque** (same winding copper, more turns = more flux linkage = more torque per amp). For hill climbing and responsive balance control, lower KV is better. The balance act of a onewheel benefits from a motor that doesn't try to run away.

---

## Voltage Options — Which to Buy

The PHUB-188 is available in 24V / 36V / 48V / 60V variants. This matters because motor windings are designed for a specific voltage range — running a motor over-voltage overheats the insulation and burns out the windings.

Our SBS_hw battery: **14S LiPo = 58.8V nominal, 60.2V max (fully charged)**

| Motor voltage | Our voltage | Ratio | Safe? |
|---|---|---|---|
| 24V | 58.8V | 245% over | ✗ Will burn out immediately |
| 36V | 58.8V | 163% over | ✗ Will burn out |
| 48V | 58.8V | 122% over | ✗ Risky, insulation stress |
| **60V** | **58.8V** | **2% under** | **✓ Within spec** |

**Buy the 60V version. This is not optional.** Any other voltage would be running the motor outside its design envelope.

---

## Candidates

### Option A — Spintend 10" 600W 60V Hub Motor (Recommended)
**Source:** [spintend.com](https://spintend.com/products/10-inch-10x6-5-5-wide-tyre-hubmotor-for-diy-one-wheel)  
**Price:** $149 USD  
**Specs:**
- Voltage: 60V rated
- Power: 600W continuous
- KV: 12.39 (measured from test data — this is the known, verified number)
- Tire: 10×6-5.5 tubeless, 150mm wide
- Axle: 31mm length, 14mm shaft diameter
- Poles: 30
- Hall sensor: 10kΩ

**Speed at 58.8V:** 12.39 × 58.8 = 729 RPM → 9.7 m/s = 34.9 km/h (~22 mph)

**Why recommended:**
- KV is precisely documented — essential for VESC motor detection and configuration
- Designed and sold specifically for DIY onewheel builds; Spintend is a known supplier in the community
- 600W continuous is sufficient for a ~80kg rider on flat-to-moderate terrain
- VESC can push 2–3× rated for brief bursts (acceleration, hills) — effective peak ~1200–1800W
- Community tested at 12S–15S

**Limitations:**
- 600W continuous is borderline for steep hills with a heavy rider — aggressive hill climbs will trigger VESC current limiting (by design)

---

### Option B — PeiScooter PHUB-188 60V (Budget Option)
**Source:** [peipeiscooter.com/products/618](https://peipeiscooter.com/products/618)  
**Price:** ~$129 USD (select 60V on product page)  
**Specs:**
- Voltage: 60V rated
- Power: ~750–1000W (estimated for 60V variant; 24V version = 600W, scales with voltage)
- KV: Not publicly documented; community estimates 10–14 for 60V winding
- Tire: 10×6.00-5.5 tubeless
- Hall sensor: included

**Why it's viable:** Well-used by the DIY onewheel community. Multiple documented builds at 12S (50.4V) using the 48V variant successfully. The 60V version is the correct choice for 14S.

**Why not recommended as primary:**
- KV is not documented — means you rely on VESC motor detection auto-tune to find it
- Less purpose-built documentation for onewheel use
- The $20 saving largely disappears once you factor shipping from China to Estonia

---

### Option C — Spintend 3000W 60V (Not recommended)
- KV = 21.25 → 58.8 × 21.25 = 1249 RPM → 16.6 m/s = 60 km/h — dangerously fast
- Overkill wattage
- Skip

---

## Decision

**Buy:** Spintend 600W 60V, $149  
**Reasoning:** Known KV (12.39), purpose-built for onewheel, documented community support, top speed of 35 km/h is appropriate, saves wiring/detection time with VESC.

---

## VESC Configuration Notes

After physical installation, motor detection in VESC Tool:
1. Motor Settings → FOC → Motor Parameters → Run Detection
2. Verify detected KV is close to 12.39 — if wildly off, check phase wire connections
3. Set ERPM limits: max ERPM = KV × battery voltage × pole pairs / 2
   - For Spintend (30 poles = 15 pole pairs): max ERPM = 12.39 × 58.8 × 15 = 10,936
   - Set App → Balance → Speed limiting to cap ERPM well below this
4. Current limits: start conservative — 40A motor, 30A battery — increase once comfortable

---

## Procurement

| Item | Supplier | Price | Link |
|---|---|---|---|
| Spintend 600W 60V hub motor | spintend.com | $149 USD | [link](https://spintend.com/products/10-inch-10x6-5-5-wide-tyre-hubmotor-for-diy-one-wheel) |

**Shipping to Estonia:** Spintend ships from China. Expect $20–40 shipping + 22% Estonian VAT on declared value.  
**Landed cost estimate:** $149 × 1.22 (VAT) + $30 shipping ≈ **$212 USD / ~€195**

Note: If ordering from peipeiscooter instead, same landed-cost calculation applies. The Spintend price advantage after taxes is minimal.
