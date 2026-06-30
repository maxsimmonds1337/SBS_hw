# DIY Onewheel — Manufacturing Documentation

## What this is

A self-balancing electric unicycle board ("onewheel") built around the SBS_hw motor controller and VESC firmware. A hub motor sits at the center; two foot platforms extend forward and rear. Foot pressure sensors (FSR) on each pad detect the rider; the VESC balance algorithm keeps the board level.

---

## Assemblies

| Assembly | Folder | Status | Est. cost |
|---|---|---|---|
| Motor | `motor/` | Chosen — Spintend 600W 60V | $149 |
| Frame | `frame/` | Specified — Al 40×40×4mm L-rails | ~€29 |
| Battery | `battery/` | Specified — 14S4P Samsung 21700 50E | ~$396 |
| Foot pads | `foot_pads/` | Specified — Spintend FSR + grip tape | ~$24 |
| Lighting | `lighting/` | Specified — simple LEDs on SBS_hw GPIO | ~$7 |
| Enclosure | `enclosure/` | Concept — PETG/TPU 3D printed | ~€14 |
| Electronics | `electronics/` | In design — SBS_hw PCB (this repo) | ~$120 |

**Total estimated landed cost to Estonia: ~$850–950 USD (including VAT and shipping)**

See [BOM.md](BOM.md) for the full itemised list.

---

## Why these choices

### Motor: Spintend 600W 60V hub motor ($149)
The motor must match our battery voltage. At 14S Li-ion = 58.8V nominal, only a 60V-rated motor is safe. Running a 48V motor at 58.8V is 22% overvoltage — it will damage the winding insulation.

The Spintend 600W has a documented KV of 12.39. At 58.8V this gives 729 RPM → ~35 km/h on a 10" wheel. That is appropriate for a recreational onewheel. The KV number matters because it tells you exactly what top speed you'll get and lets you configure the VESC correctly without guessing.

See `motor/motor.md` for the full KV tutorial and candidate comparison.

### Frame: Aluminum L-profile rails
Two 40×40×4mm EN AW-6060 T6 L-profile rails, each 860mm, clamped around the motor axle. This is the simplest stiff frame for a single-wheel board. Structural analysis confirms 40×40×4mm handles a 90 kg rider with ~1.35× margin.

Source: Metal Express, Tallinn (metalexpress.ee). Cut-to-length from 6m stock.

See `frame/frame.md` for cut list, axle mounting options, and structural analysis.

### Battery: 14S4P Samsung 21700 50E
- 14S for correct SBS_hw voltage (58.8V)
- 4P for current headroom (9.8A × 4 = 39A; VESC will limit to 30A)
- Samsung 50E: 5Ah cells, widely available, well-characterised; ~$5.50 each

56 cells total. Assembled with a Daly 14S 30A Smart BMS for protection and balancing. Pack dimensions ~520×100×80mm, 2.8 kg. Provides ~8–12 km range.

See `battery/battery.md` for cell comparison, pack assembly notes, and charger spec.

### Foot pads: Spintend FSR
Force Sensitive Resistors under grip tape. When both pads are loaded, EN_BUCK goes high and the board arms. Lift either foot → motor disables. This is the same principle as the commercial Onewheel XR.

The SBS_hw board already has the CONN_SW latch circuit (R14/R15 PTC + Zener clamp). The Spintend FSR sensor + adapter outputs the right signal level for the VESC ADC input.

See `foot_pads/foot_pads.md` for wiring to SBS_hw and DIY FSR alternative.

### Electronics: SBS_hw PCB
The SBS_hw motor controller is the "brains" — it runs VESC firmware, controls the motor via 3-phase FOC, reads the IMU (BMI160) for balance, reads hall sensors from the hub motor, and drives the LEDs. The PCB design and all component analysis live in the parent directory of this repo.

See `electronics/electronics.md` for the integration checklist and connector mapping.

### Lighting: Discrete LEDs on GPIO
Simple white (front) and red (rear) LEDs driven from MCU GPIO via BSS138. No DC-DC converter needed, no special firmware protocol. Can be upgraded to WS2812B addressable strip later.

### Enclosure: PETG/TPU 3D printed
Electronics bay cover and side cosmetic panels in PETG (rigid, UV resistant). Footpad top surface in TPU 95A (grip, impact absorption). CAD to be done in OnShape. Total filament cost ~€10, print time estimate ~8 hours.

---

## Assembly sequence

1. **PCB** — Order and assemble SBS_hw. Test on bench at 14S (58.8V bench supply, no motor). Flash VESC firmware.
2. **Motor** — Order Spintend 600W 60V. Run motor detection in VESC Tool.
3. **Battery** — Assemble 14S4P pack. Charge and verify cell voltages balance. Install BMS.
4. **Frame** — Cut L-rails to 860mm. Machine axle slots. Assemble clamp hardware.
5. **Motor + frame** — Mount motor axle into rails. Verify alignment and clamping.
6. **Battery + frame** — Design battery bay dimensions from assembled pack. 3D print enclosure.
7. **Electronics + frame** — Mount SBS_hw PCB. Run wiring to motor, battery, FSRs, LEDs.
8. **Foot pads** — Mount FSR sensors. Apply grip tape.
9. **First power-on** — With board elevated. Verify balance PID responds. Set ERPM limits.
10. **Ride test** — Start on grass. Increase VESC speed limit incrementally.

---

## Open items / CAD work needed

- [ ] OnShape model of frame (rail dimensions, axle slots, foot platform cutout)
- [ ] OnShape model of electronics bay and enclosure
- [ ] FSR pocket design in footpad
- [ ] Battery bay dimensions once pack is assembled
- [ ] Verify axle clamping torque spec (M6 bolt tightening)

---

## Safety notes

- **Never bypass the footpad safety circuit.** The EN_BUCK latch must cut motor power when feet leave the board.
- **Charge the battery in a fire-safe location** (LiPo-rated bag or fireproof container). See `battery/battery.md` for fire response notes.
- **First rides:** Always start with weight on the board before arming. VESC balance app has a "startup speed" requirement — the board will not try to balance until moving above ~0.5 km/h.
- **ERPM limits:** Set these in VESC before first ride. Recommended starting maximum: 6000 ERPM (~20 km/h) until comfortable.
