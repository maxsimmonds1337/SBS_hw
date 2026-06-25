# Battery Design & Safety
**Project:** SBS_hw v1.0 — Self-Balancing Unicycle  
**Last updated:** 2026-06-24

---

## 1. Cell Count Estimation

No chassis dimensions exist yet — these are estimates from typical 10-inch DIY onewheel geometry. **Measure your actual foot wells before ordering cells.**

A 10-inch hub motor (254mm diameter, 152mm wide tire) sits centrally. Typical DIY builds give two foot pads roughly **200mm long × 130mm wide**, with **~40–50mm usable depth** under each footpad before hitting frame/axle.

### Cell geometry (21700 recommended)

21700 cells (21mm diameter × 70mm length) lying flat, two layers high (2 × 21mm = 42mm — fits in 45mm depth):

- Per layer per pad: ⌊200/70⌋ × ⌊130/21⌋ = 2 × 6 = **12 cells**
- Two layers per pad: **24 cells**
- Two pads total: **~48 cells** available space

### 14S configurations

The peak current demand sets the minimum parallel count — a single cell cannot handle 80A peak.

| Config | Total cells | Peak A per cell (80A pack) | Suitable cell | Energy (P42A) | Notes |
|---|---|---|---|---|---|
| 14S1P | 14 | **80A — exceeds all cell ratings** | None | 217 Wh | Do not use |
| 14S2P | 28 | 40A | Molicel P42A (45A rated) ✓ | 434 Wh | **Recommended** |
| 14S3P | 42 | 27A | Most quality 21700 cells ✓ | 651 Wh | More range, heavier |

**14S2P with Molicel P42A** is the practical choice. 28 cells fit easily in two foot pads; 434 Wh comfortably exceeds the Onewheel XR (324 Wh).

Maximum pack voltage: 14 × 4.2V = **58.8V** — confirms ≥80V MOSFET requirement.

---

## 2. Cell Selection

Buy from reputable suppliers only. The rewrapped cell market (AliExpress, Amazon) is rampant with counterfeits.

**Recommended UK/EU suppliers:** NKON (Netherlands), 18650uk, Fogstar, EV Works  
**US:** Liion Wholesale, 18650 Battery Store

**Top cells for this application (21700 format):**

| Cell | Capacity | Cont. current | Remark |
|---|---|---|---|
| Molicel P42A | 4200 mAh | 45A | Best choice for 14S2P at 80A peak |
| Samsung INR21700-50S | 5000 mAh | 25A | Higher energy, lower current — 14S3P needed |
| Molicel P45B | 4500 mAh | 45A | Slightly newer than P42A, similar performance |

**Quick counterfeit check:** Weigh cells (P42A ≈ 68g — any cell significantly lighter is suspect). Test internal resistance with a YR1035+ or similar meter.

---

## 3. What Causes Li-ion Fires

The failure mode is always **thermal runaway** — a self-sustaining exothermic reaction once the cell exceeds ~130–150°C. Cell chemistry breaks down, generating heat faster than it dissipates, driving the next stage of breakdown. Once started it cannot be stopped chemically; you can only cool it or contain it.

| Trigger | Mechanism | Design prevention |
|---|---|---|
| **Overcharge** (>4.2V NMC cell) | Lithium plating → dendrite growth → internal short | BMS per-cell overvoltage cutoff at 4.20–4.25V |
| **Overdischarge** (<2.5V) | Copper dissolution from anode → dendrites on recharge | BMS undervoltage cutoff at 2.8–3.0V |
| **Overcurrent** | I²R heating, separator melts at ~130°C | BMS overcurrent, correct cell parallel count |
| **Mechanical damage** | Puncture or crush → immediate internal short | Solid enclosure, no sharp protrusion paths |
| **External heat** | >60°C degrades, >100°C separator melts | NTC monitoring, keep away from heat sources |
| **High-resistance weld** | Local hotspot at terminal → one cell overheats, propagates | Weld quality, pure nickel strip, resistance check |
| **Cell reversal** | Weakest cell in string reaches 0V, driven negative | Match cells, BMS undervoltage per-cell |

---

## 4. Spot Welding Safety

### Equipment

- **Spot welder:** A proper bench welder (KSGER SW6, SUNKKO 709A, Sequre SQ-SW1) delivers consistent, repeatable pulse energy. Avoid pen welders — inconsistent energy = inconsistent weld resistance = uneven cell temperatures in use.
- **Nickel strip:** Pure nickel only, ≥0.15mm thick (0.2mm for high-current busbars). Test with a magnet — nickel is non-magnetic. Nickel-plated steel (common cheap strip) has 5–8× higher resistance.
- **PPE:** Safety glasses (cells can vent), gloves.

### Assembly procedure

1. **Characterise every cell first.** Measure open-circuit voltage (must all match to ±0.05V before connecting in parallel) and internal resistance (match to within 10–15% within each parallel group).
2. **Assemble parallel groups first** (P groups), then connect in series. Never bridge a series connection accidentally.
3. **Tape exposed terminals** on completed groups before moving to the next step.
4. **Test weld strength** on a dummy cell — a good weld requires needle-nose pliers to remove the strip; it tears the nickel, not the weld.
5. **Measure contact resistance** across each weld with a milliohm meter. Consistent, low values confirm good welds.
6. **Never work with metal tools across the full pack voltage.** A 14S2P pack can deliver several thousand amps into a short circuit — tools become plasma.
7. **Charge the completed pack for the first time in a safe location** (outdoors, or in a fireproof container) while monitoring cell voltages individually.

---

## 5. BMS Requirements

For a 14S pack, the BMS is not optional. It must provide:

| Function | Spec for this pack |
|---|---|
| Per-cell overvoltage cutoff | 4.20–4.25V/cell |
| Pack undervoltage cutoff | 2.8–3.0V/cell (39.2–42V pack) |
| Continuous overcurrent | ≥80A (set to 100A) |
| Short circuit protection | <500 µs response |
| Cell balancing | Passive minimum; active preferred |
| Temperature monitoring | NTC on cells, cutoff at 60°C charge / 80°C discharge |

**Recommended BMS options:**
- **Daly Smart BMS 14S 100A** — reliable, has Bluetooth monitoring app, widely used in DIY builds
- **ANT BMS 14S** — better active balancing, higher cost
- **JK BMS** — good balance current, popular in the community

Avoid unbranded BMS boards with suspicious MOSFET ratings printed on them.

---

## 6. Charging Safety

- **Always charge inside a fireproof container** or lipo safe bag, placed on a non-flammable surface (concrete, not carpet or wood).
- **Never charge unattended for extended periods.** At minimum, be present for the first few cycles of a new pack.
- **Do not charge immediately after heavy use** — cells should cool to <40°C before charging.
- **Charge at room temperature** (5–45°C). Cold charging (<0°C) causes lithium plating even with a good BMS.
- **Set charger voltage correctly:** 14S NMC = 58.8V max. Verify with a multimeter before first use.
- **Storage charge:** If not riding for >2 weeks, store at ~50–60% state of charge (3.7–3.8V/cell = 51.8–53.2V pack). Full-charge storage degrades cells faster.

---

## 7. Fire Response

### What you're dealing with

A Li-ion fire is not a normal fire. The electrolyte (LiPF6 in organic solvent) vaporises into **hydrofluoric acid (HF), carbon monoxide, and other toxic gases**. HF is absorbed through skin and lungs and is acutely dangerous — the effects can be delayed. **Get away from the smoke.**

Thermal runaway can **reignite after appearing extinguished** — cells continue reacting internally. A pack that stops smoking is not necessarily safe. Watch it for 30+ minutes.

### If you see smoke or swelling — act immediately

1. **Do not touch the pack with bare hands.**
2. If the pack is inside a lipo bag or metal container, **move the whole container outside** immediately using heat-resistant gloves. Set it on concrete, away from vehicles and buildings.
3. If the pack is not yet on fire (just swelling/venting), **keep your distance** — venting gas is flammable and toxic.
4. **Do not put it in your car** to take it somewhere — packs have started fires in vehicles.

### If it is actively on fire

1. **Evacuate the area. Get everyone out.** Do not attempt to fight a large Li-ion fire.
2. **Call 999 / 112 / your local emergency service.** Tell them it is a lithium battery fire.
3. **Do not use:**
   - **CO2 extinguisher** — ineffective; Li-ion fires are self-oxidising (the electrolyte provides its own oxygen). CO2 smothers flames but cells continue reacting.
   - **Small amounts of water** — risks steam explosion and can spread flaming electrolyte.
   - **Standard dry powder (ABC)** — suppresses surface flames temporarily, does not stop thermal runaway.
4. **What actually works — large volumes of water.** Emergency services flood Li-ion fires with water to cool cells below the thermal runaway threshold. This is counterintuitive but correct for Li-ion (not pure lithium metal). If you have a garden hose and a small pack, fully submerging it in a bucket of water is the recognised hobbyist approach.
5. **After apparent extinguishment:** Leave the pack outside on concrete for several hours. Do not bring it indoors. Li-ion packs are known to reignite hours later.

### Mitigation hardware to own before you build

| Item | Purpose | Notes |
|---|---|---|
| **Lipo/LiHV safe bag** (large, ≥20L) | Contain a venting or igniting pack | Fiberglass construction. Contains a fire for several minutes — buys time to move outdoors. Not indefinite protection. |
| **Steel ammo can** | Better containment than fabric bag | A 50-cal ammo can holds a 14S2P 21700 pack. Can be vented with a hole + steel wool to allow gas to escape without flame. |
| **ABC dry powder extinguisher** (≥2kg) | Suppress flames during evacuation | Not a fix, but buys time. Keep one in your workshop. |
| **Heat-resistant gloves** | Handle a hot pack | Rated to ≥250°C |
| **Smoke detector in workshop** | Early warning | Li-ion venting produces distinctive chemical smell before ignition |
| **YR1035+ internal resistance meter** | Catch degrading cells early | High-resistance cells are thermal runaway risks. Check pack every few months. |

### Where to charge and store

- **Workshop or garage** with a concrete floor, away from flammable materials.
- **Inside a steel ammo can** with the lid not fully sealed (allows gas to vent without pressure build-up).
- **Never in a bedroom, living room, or car** unless using a purpose-built fireproof charging cabinet.
- **Keep a bucket of water nearby** when charging — if a pack starts venting, submerge it immediately.

---

## 8. Storage and Disposal

- **Long-term storage:** 50–60% charge, 15–25°C, dry location.
- **Damaged cells** (swollen, dented, punctured, or with high internal resistance): discharge fully (tape terminals), place in a lipo bag, and take to a battery recycling point. Never throw in general waste.
- **Recycling:** Most councils have battery drop-off points. For larger packs, contact a specialist recycler.

---

## 9. Useful References

- Battery University — batteryuniversity.com (excellent non-commercial resource on Li-ion chemistry and failure modes)
- DIY Onewheel community builds on esk8.news (real-world build experience, what has and hasn't caused fires)
- Molicel P42A datasheet — confirm continuous current and temperature limits before finalising cell choice
