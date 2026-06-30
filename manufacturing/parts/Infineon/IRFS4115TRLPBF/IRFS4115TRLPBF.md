---
mpn: IRFS4115TRLPBF
manufacturer: Infineon Technologies (formerly International Rectifier)
description: N-channel MOSFET, 150V 104A D2PAK — main power switch replacement for 14S bus
lcsc: C2692945
package: D2PAK (TO-263-2)
footprint: Package_TO_SOT_SMD:TO-263-2
datasheet_url: https://www.infineon.com/dgdl/irfs4115pbf.pdf?fileId=5546d462533600a401535675aee92289
kicad_datasheet: ${KIPRJMOD}/../../manufacturing/parts/Infineon/IRFS4115TRLPBF/IRFS4115TRLPBF.pdf
used_as: Q1, Q3, Q4, Q5, Q6, Q7 — 3-phase half-bridge switches (6 total)
replaces: IRF7749L1TRPBF (DirectFET L8, 40V — CRITICALLY UNDERSIZED for 14S)
---

# IRFS4115TRLPBF — 150V 104A N-MOSFET (D2PAK)

## Key Specs

| Parameter | Value |
|-----------|-------|
| Vds max | 150 V |
| Id (Tc=25°C) | 104 A |
| Id (Tc=100°C) | 66 A |
| Rds(on) typ | 10 mΩ @ Vgs=10V, Tj=25°C |
| Rds(on) max | 14 mΩ @ Vgs=10V, Tj=25°C |
| Rds(on) @ 125°C | ~20 mΩ (2× typ factor from datasheet) |
| Vgs max | ±20 V |
| Qg total | 93 nC |
| Qgd (Miller) | 19 nC |
| tr / tf | 22 ns / 22 ns (at Rg=1Ω test) |
| θjc | 0.6 °C/W |
| Tj max | 175 °C |
| Package | D2PAK (TO-263-2) |

## Why IRFS4115 replaces IRF7749L1TRPBF

| | IRF7749L1TRPBF (current) | IRFS4115TRLPBF (replacement) |
|--|--|--|
| Vds max | **40 V** | **150 V** |
| Bus voltage (14S) | 58.8 V — **EXCEEDS Vds** | 58.8 V — 2.55× margin |
| Package | DirectFET L8 | D2PAK (TO-263-2) |
| Rds(on) | 2.5 mΩ | 10 mΩ |
| LCSC | N/A for 14S use | C2692945 |

The IRF7749L1TRPBF has Vds(max) = 40 V. The 14S battery charges to 58.8 V, which immediately exceeds the absolute maximum. With inductive switching transients this is a catastrophic failure mode.

## Voltage Derating Analysis

```
Vbus_max        = 58.8 V  (14S × 4.2 V/cell)
L_loop_target   = 30 nH   (achievable with tight half-bridge layout)
I_off_max       = 50 A    (conservative motor peak during turn-off)
tf              = 50 ns   (scaled from datasheet at Rg=4.7Ω, see switching section)

V_spike = L_loop × (I_off / tf)
        = 30 nH × (50 A / 50 ns)
        = 30 nH × 1.0 A/ns
        = 30 V

Vds_peak = Vbus + V_spike = 58.8 + 30 = 88.8 V

Derating: 88.8 V / 150 V = 59.2%  ✓  (target < 80%)
```

If PCB layout achieves L_loop < 50 nH (reasonable for a compact half-bridge):
`V_spike = 50 nH × 1.0 A/ns = 50 V → Vds_peak = 108.8 V → 72.5% derating — acceptable`

> **Layout note:** Each half-bridge capacitor (4.7µF × 2 = 9.4µF per phase) is placed as close as possible to the MOSFET drain/source pads to minimise L_loop. See §PCB Notes.

### Why Not 80V?

80V parts (e.g. DirectFET equivalents) would give only 80 - 58.8 = 21.2 V of headroom for transient spikes. A 50 nH loop with 50A turn-off produces a 50V spike: 58.8 + 50 = 108.8V **exceeds** an 80V part. **Minimum 100V required; 150V recommended.**

## Switching Loss Calculation

```
Application:
  Vbus    = 58.8 V
  I_load  = 40 A  (continuous motor RMS)
  fsw     = 20 kHz  (VESC default)
  Rg      = 4.7 Ω  (gate resistor in schematic, R10/R11/R12/R14/R15/R17)
  Vdrv    = 15 V  (DRV8301 gate drive voltage)
  Vth_miller ≈ 5 V (gate threshold mid-point, approximate)

Gate current during Miller plateau:
  Ig = (Vdrv - Vth_miller) / Rg = (15 - 5) / 4.7 = 2.13 A

Crossover time (Miller plateau):
  t_miller = Qgd / Ig = 19 nC / 2.13 A = 8.9 ns

Total switching time (scaled from datasheet tr=22ns @ Rg=1Ω):
  t_sw ≈ tr × Rg_total / Rg_test
       = 22 ns × (4.7 + 0.5 Ω) / (1 + 0.5 Ω)     [0.5Ω = DRV8301 drive impedance]
       = 22 ns × 3.47 = 76 ns  (both tr and tf)

Energy per turn-on or turn-off event:
  E_sw = 0.5 × Vbus × I_load × t_sw
       = 0.5 × 58.8 × 40 × 76 ns
       = 89.4 µJ

Switching losses per MOSFET (one turn-on + one turn-off per cycle):
  P_sw = 2 × E_sw × fsw
       = 2 × 89.4 µJ × 20 kHz
       = 3.58 W
```

## Conduction Loss Calculation

Using the 3-phase sinusoidal PWM formula for RMS switch current:

```
I_phase_peak = I_motor_rms × √2 = 40 A × 1.414 = 56.6 A  (motor at 40A RMS)

Per-switch RMS (sinusoidal modulation, m=0.85, cosφ=0.9):
  I_sw_rms² = I_phase_peak² × (1/8 + m·cosφ / (3π))
            = 56.6² × (0.125 + 0.085 × 0.9 / 9.42)
            = 3203 × (0.125 + 0.0813)
            = 3203 × 0.2063
            = 660.4
  I_sw_rms  = 25.7 A

Conduction loss at Tj = 125°C (Rds(on) ≈ 20 mΩ typ):
  P_cond = I_sw_rms² × Rds(on)
         = 660.4 × 0.020
         = 13.2 W
```

## Total Dissipation and Thermal Analysis

```
P_total per MOSFET = P_cond + P_sw = 13.2 + 3.58 = 16.8 W  (at 40A continuous)

Thermal path (D2PAK on PCB):
  θjc = 0.6 °C/W   (datasheet)
  θca = 4.0 °C/W   (target: 50 cm² of 2oz copper pour — see PCB Notes)

Tj = Ta + P × (θjc + θca)
   = 40°C + 16.8 W × 4.6 °C/W
   = 40°C + 77.3°C
   = 117.3°C

Tj margin = 175 - 117.3 = 57.7°C  ✓
```

### Peak Current (80A, transient)

```
At I_motor_rms = 80A peak (brief, <5s):
  I_sw_rms = 51.4 A
  P_cond   = 51.4² × 25 mΩ = 66.0 W  (at 125°C, Rds × 2.5)
  P_sw     = ~4 W (switching losses scale weakly with current)
  P_total  ≈ 70 W

Steady-state Tj at 70W (would not be reached in <5s):
  Tj_ss = 40 + 70 × 4.6 = 362°C  — would destroy device if sustained

Thermal capacitance slows temperature rise. For a 2-second pulse:
  Zth(j-c, 2s) ≈ 0.35 °C/W  (from single-pulse curve, D2PAK)
  Zth(c-a, 2s) ≈ 1.5 °C/W   (PCB thermal capacitance)
  ΔTj(2s) = 70 W × 1.85 = 130°C
  Tj_transient = 40 + 130 = 170°C  — at the limit

Conclusion: 80A for >2s will push Tj to the limit. VESC firmware temperature 
derating (MOTOR_TEMP_LIMIT in VESC Tool) must be configured. NTC TH1 on the 
PCB provides closed-loop protection.
```

### Recommendation for High-Duty Use

If targeting consistent 60A+ motor current: **parallel two IRFS4115 per switch position** (2× MOSFET in D2PAK side-by-side on the PCB). This halves conduction losses per device and gives 40°C more thermal margin. Current sharing is adequate due to positive Rds(on) temperature coefficient.

## Snubber Assessment

With good PCB layout (L_loop = 30 nH) and IRFS4115 at 150V:
- Maximum Vds spike = 88.8 V (calculated above)
- 150 V rating provides 61.2 V margin above spike
- **No RC snubber required**

If layout results in L_loop > 50 nH (measured by observing overshoot at bring-up):
- Add TVS: SMDJ58CA (bidirectional, 58V working, 93.6V clamping) across each half-bridge
- Or RC snubber: 10 nF + 10 Ω in series, across each MOSFET drain-source

## Package Change from DirectFET L8

| Aspect | DirectFET L8 | D2PAK (TO-263-2) |
|--------|-------------|------------------|
| PCB footprint | `Package_DirectFET:DirectFET_L8` | `Package_TO_SOT_SMD:TO-263-2` |
| Heatsink method | Top exposed can → copper pour on top layer | Tab on bottom → copper pour on bottom layer |
| θjc | 1.0 °C/W (bottom) | 0.6 °C/W |
| VESC MK5 compatible | Yes (original) | PCB redesign needed |
| Available ≥100V | No confirmed stock | **Yes, LCSC C2692945** |

PCB is not yet laid out, so the footprint change to D2PAK is acceptable. The D2PAK tab mounts flat on the PCB; the drain current passes through the tab. Ensure the footprint includes exposed copper pad + thermal vias to internal copper layers.

## PCB Notes

- **Copper pour:** Minimum 50 cm² of 2oz copper connected to the D2PAK tab (pin 2 = drain)
  for each MOSFET to achieve θca ≤ 4°C/W. With 4-layer PCB: pour on all layers + thermal vias.
- **Half-bridge loop:** High-side drain (SUPPLY), high-side source / low-side drain (phase output),
  low-side source (shunt) — keep this current loop as tight as possible (target < 1 cm²)
- **Decoupling caps:** Place 4.7µF 100V MLCCs (CGA6M3X7S2A475KT0Y3S, already in design)
  as close as possible to the half-bridge node — directly between SUPPLY and the phase output node
- **Gate traces:** Keep gate traces < 20 mm from DRV8301 output pins; 4.7Ω gate resistors
  (R10/R11 etc.) placed right at the gate pin, not at the driver output

## KiCad Symbol Update Required

To replace IRF7749L1TRPBF in the schematic (do this in KiCad with the project closed to this editor):

1. Open `Power.kicad_sch`
2. For Q1, Q3, Q4, Q5, Q6, Q7: right-click → Properties
   - **Value:** `IRFS4115TRLPBF`
   - **Footprint:** `Package_TO_SOT_SMD:TO-263-2`
   - **Datasheet:** `${KIPRJMOD}/../../manufacturing/parts/Infineon/IRFS4115TRLPBF/IRFS4115TRLPBF.pdf`
   - **LCSC:** `C2692945`
3. The symbol lib_id (`Transistor_FET:IRF7748L1`) can remain — the pin ordering (Gate=1, Source=2, Drain=3) is compatible with D2PAK standard pinout.
   - D2PAK TO-263-2: Pin 1 = Gate, Pin 2 = Drain (tab), Pin 3 = Source — verify against footprint.

> Note: KiCad is currently open. Make these changes in the KiCad GUI to avoid conflicts.
