# Foot Pads Sub-Assembly

## Function

Foot pads serve two roles:
1. **Physical surface** — grip surface for the rider's feet
2. **Presence detection** — FSR (Force Sensitive Resistor) sensor detects rider is on the board; when both pads are unoccupied, the motor cuts off (safety)

The SBS_hw CONN_SW circuit already implements the latch logic (EN_BUCK/SHUTDOWN via 1kΩ PTC resistors + Zener). The FSR sensors connect directly to this circuit.

---

## FSR Sensor Options

### Option A — Spintend FSR + Adapter (Recommended, easiest)
**Source:** [spintend.com](https://spintend.com/products/high-performance-force-pressure-sensor-for-diy-onewheel)  
**Price:** $29 (sensor + adapter) or $25 (sensor only)  
**Trigger weight:** 500g  
**Notes:**
- Designed specifically for DIY onewheel + VESC
- Adapter board provides correct signal conditioning for VESC balance app
- 2-pin connector to VESC ADC input
- Connects to SBS_hw's CONN_SW / ADC header pins

### Option B — Spintend Large FSR (larger surface coverage)
**Source:** [spintend.com](https://spintend.com/products/big-size-high-performance-force-pressure-sensor-for-build-onewheel)  
**Price:** $9–14 (sensor + adapter)  
**Notes:**
- Larger sensing area — better for varied foot placement
- Similar signal interface

### Option C — DIY FSR (cheapest)
Buy raw FSR film sensors from AliExpress (~$2 each, e.g. Interlink 402 FSR or generic).  
Wire directly to SBS_hw CONN_SW pins via a simple voltage divider (10kΩ pull-up to 3.3V, FSR to GND).  
**Cost: ~$5 for 2 sensors + $1 in resistors**  
**Trade-off:** Need to calibrate trigger threshold in VESC; less plug-and-play.

---

## How the CONN_SW Circuit Works (SBS_hw)

From the SBS_hw schematic (SBS_hw.kicad_sch, Sheet 1):
- Two 1kΩ PTC resistors (R14, R15) on SHUTDOWN/CONN_SW signal path
- A Zener (D4, BZX84C3V6 3.6V) provides clamping
- The latch: when both footpads are pressed, EN_BUCK goes high → buck converter enables → board powers up motor outputs
- When either foot lifts (FSR resistance >> threshold): EN_BUCK falls → motor output disables
- This is the same "footpad engagement" safety logic as the commercial Onewheel XR

**Wiring the FSR to SBS_hw:**
- FSR + → CONN_SW_A pin (front footpad)
- FSR − → GND
- Signal to VESC ADC for analog measurement (threshold set in VESC balance app)

---

## Physical Footpad Platform — Wood (Recommended)

Birch plywood is the standard DIY onewheel footpad material — the same wood used in skateboard and longboard decks. It's stiff, light, easy to cut, accepts grip tape well, and trivial to route a pocket for the FSR sensor.

**Spec: 12mm Baltic birch plywood**
- 12mm is stiff enough for a 90 kg rider on a ~300mm unsupported span without noticeable flex
- 9mm works if you want to save ~100g per pad, but 12mm is safer for curb drops
- Size per pad: 300mm × 150mm (trimmed to fit rail width)
- The FSR pocket: 5mm deep × sensor footprint, routed or chiselled into the underside

**Construction:**
1. Cut two pieces 300×150mm from sheet
2. Rout a shallow pocket on the underside for the FSR sensor (5mm deep, sensor footprint)
3. Run sensor cable through a 4mm hole to the top face, then back down to the rail
4. Mount pad to L-rail with 4× M4 countersunk screws (flush top)
5. Apply grip tape to top surface

### Where to buy in Tallinn

**MASS.ee** — confirmed stock in Tallinn, 12mm birch ply from **€27.51 incl. VAT**  
Website: [mass.ee](https://www.mass.ee)  
Note: full sheets are 1250×2500mm (~€55); you only need 0.09m² total — ask for offcuts or get one cut at the store

**Bauhof (Laki 3a, Tallinn)** — stocks birch ply in various sizes, offers in-store cutting service. Most convenient option; typically €3–5/m² for offcuts.

**K-Rauta (Tallinn)** — similar to Bauhof, birch ply in stock.

**Cost for footpad wood:** ~€3–8 for offcuts — negligible.

---

## Alternatives to Wood

1. **Grip tape directly on the L-rail flat face** — zero extra material, but no FSR pocket and less comfortable underfoot
2. **3D print in TPU** — see `../enclosure/enclosure.md` — heavier, slower to make, but nice if you want a custom shape
3. **Aftermarket Onewheel-style pads** — check Craft&Ride or FloatLife for compatible pads, but width may not match custom frame

---

## Bill of Materials

| Item | Qty | Unit price | Total | Source |
|---|---|---|---|---|
| 12mm birch ply offcut (~400×200mm) | 1 | ~€5 | ~€5 | Bauhof / MASS.ee, Tallinn |
| Spintend large FSR + adapter | 2 | $9 | $18 | spintend.com |
| OR: DIY FSR (Interlink 402) | 2 | $2 | $4 | AliExpress |
| Grip tape (skateboard, 300×150mm) | 2 sheets | €3 | €6 | Local skate shop / Bauhof |
| M4×20mm countersunk screw | 8 | €0.15 | €1.20 | Hardware store |
| M4 nut | 8 | €0.10 | €0.80 | Hardware store |
| **Total (Spintend FSR + wood)** | | | **~€30** | |
| **Total (DIY FSR + wood)** | | | **~€15** | |
