# Electronics Sub-Assembly

## Overview

The electronics for the onewheel are almost entirely the **SBS_hw motor controller board** — the design this entire repo documents.

This file exists as a pointer and integration checklist, not new design work.

---

## SBS_hw Integration Checklist

Before installing the board into the onewheel frame:

- [ ] All 4 KiCad schematic blockers resolved (MC-002, MC-003, MC-013, MC-015)
- [ ] Schematic annotated (all R?/C?/U? → R1/C1/U1 etc.)
- [ ] Gerbers generated and ordered from JLCPCB
- [ ] PCB assembled and tested on bench with 14S pack
- [ ] VESC firmware flashed (VESC 6.x, target `60_mk5`)
- [ ] Motor detection run with Spintend 600W motor
- [ ] Balance app configured: VESC Balance (FloatPackage or native)
- [ ] Footpad FSR thresholds configured
- [ ] Current limits set (start: 40A motor / 30A battery)
- [ ] LED GPIO assigned and tested

---

## Connectors Needed on SBS_hw for Onewheel

| Signal | SBS_hw connector | Connects to |
|---|---|---|
| SUPPLY / GND | Battery input | 14S4P pack XT60 |
| PHASE_A/B/C | Motor output (3-phase) | Hub motor phase wires |
| HALL_A/B/C + VCC + GND | J3 JST-PH 6P | Hub motor hall sensor |
| CONN_SW_A | J5 / CONN_SW pins | Front FSR sensor |
| CONN_SW_B | COMM header | Rear FSR sensor |
| SH_A | EN_BUCK signal | (internal latch) |
| LED_OUT | GPIO via BSS138 | Front/rear LEDs |
| CAN_H / CAN_L | J2 JST-PH 4P | (optional: remote display) |
| USB | J1 USB Micro-B | VESC Tool configuration |

---

## PCB Cost Estimate

| Item | Est. cost | Notes |
|---|---|---|
| JLCPCB PCB fabrication (5 off) | ~$30 | 4-layer, standard lead time |
| JLCPCB PCBA (assembly, 1 board) | ~$40–60 | Includes all Basic Parts placement |
| LCSC components (non-Basic) | ~$40 | MOSFETs, ICs, connectors |
| **Total per assembled board** | **~$110–130** | |

---

## What SBS_hw Does Not Cover

- **Battery management (BMS)** → see `../battery/battery.md`
- **Charger** → see `../battery/battery.md`
- **Rider presence detection logic** → FSR connects to CONN_SW; SBS_hw has the hardware, VESC firmware does the logic
