# SBS_hw PCB Cost Estimate
**Date:** 2026-06-30  
**Exchange rate:** 1 EUR = 1.1416 USD  
**Board spec:** 4-layer, 300×150mm, FR-4, HASL, 1 oz copper  
**Order quantity:** 5 boards (JLCPCB minimum for Economic PCBA)

---

## Summary

| Line item | USD | EUR |
|---|---|---|
| PCB fabrication (5× 4-layer 300×150mm) | $52 | €46 |
| SMT assembly (Economic PCBA, 5 boards) | $67 | €59 |
| Components (all LCSC, for 5 boards) | $175 | €153 |
| DHL Express to Estonia | $35 | €31 |
| EU customs duty (0% on PCBs/assemblies) | $0 | €0 |
| Estonia VAT 22% (on goods + shipping) | $72 | €63 |
| **Total for 5-board run** | **~$401** | **~€351** |
| **Per board** | **~$80** | **~€70** |

Cost range: €61–92/board depending on surface finish, substitute parts, and shipping method (see sensitivity table below).

---

## PCB Fabrication

JLCPCB 4-layer board rate: $70.6/m²

| Item | Calculation | Cost |
|---|---|---|
| Board area | 300mm × 150mm = 0.045 m² per board | — |
| Material (5 boards) | 0.225 m² × $70.6/m² | $15.89 |
| NRE / engineering fee (est.) | fixed | ~$35 |
| **Total PCB fab** | | **~$52** |

Notes:
- Board is 450 cm², below the 650 cm² large-board surcharge threshold ✓
- Board length 300mm, below 600mm long-board surcharge threshold ✓
- HASL assumed; ENIG adds ~$10–15
- Impedance control not specified; no surcharge assumed

---

## SMT Assembly (Economic PCBA)

| Fee item | Rate | Qty | Cost |
|---|---|---|---|
| Setup fee | fixed | 1 | $8.00 |
| Stencil | fixed | 1 | $1.50 |
| SMT solder joints | $0.0016/joint | ~2,000 (400/board × 5) | $3.20 |
| Extended part loading | $3.00/unique type | 17 types | $51.00 |
| THT hand-solder (JST, headers) | $3.50/order | 1 | $3.50 |
| **Total assembly** | | | **$67.20** |

Extended parts counted (17 types):
STM32F405RGT6, DRV8301DCAR, AD8418ARMZ (×2 same part = 1 type), TCAN1051VDQ1, TC2117-3.3VDBTR, IRFS4115TRLPBF, DSS210, SMAJ5.0A, SN74LVC1G66DCKR, BSS138, shunt resistor (C5375456), crystal (C5181477), inductor (C135264), 4.7µF 100V MLCC (C338088), 470µF 100V electrolytic, USB Micro-B (C10418), NTC thermistor (C13564).

BMI160 and NRF51822 excluded (both out of stock on LCSC — see stock flags below).

---

## Component Prices (LCSC, 1-piece / minimum order)

### ICs and Active Components

| LCSC# | Description | Qty | Unit (USD) | Extended | Stock | Notes |
|---|---|---|---|---|---|---|
| C15742 | STM32F405RGT6 | 1 | $5.01 | $5.01 | 800 | OK |
| C98969 | DRV8301DCAR gate driver | 1 | $3.12 | $3.12 | 2,300 | OK |
| C99534 | AD8418ARMZ current sense | 2 | $3.79 | $7.58 | 31 | ⚠️ LOW — only 31 pcs |
| C133788 | TCAN1051VDQ1 CAN xcvr | 1 | $1.24 | $1.24 | 60 | OK |
| C98655 | TC2117-3.3VDBTR LDO | 1 | $1.44 | $1.44 | 414 | OK |
| C94021 | BMI160 IMU | 1 | — | — | 0 | ❌ OUT OF STOCK |
| C2151959 | EYSGJNZWY NRF51822 module | 1 | $4.60 | $4.60 | 0 | ❌ OUT OF STOCK |

### Power Stage

| LCSC# | Description | Qty | Unit (USD) | Extended | Stock | Notes |
|---|---|---|---|---|---|---|
| **C53417** | IRFS4115TRLPBF 150V MOSFET | 6 | $1.87 | $11.22 | 146 | OK — use C53417, NOT C2692945 (wrong part) |

### Discrete Semiconductors

| LCSC# | Description | Qty | Unit (USD) | Extended | Stock |
|---|---|---|---|---|---|
| C511868 | DSS210 bootstrap Schottky | 3 | $0.044 (min 20) | $0.13 | 136,820 |
| C87074 | SMAJ5.0A TVS | 1 | $0.127 (min 5) | $0.13 | 18,345 |
| C113518 | SN74LVC1G66DCKR analog switch | 3 | $0.081 (min 5) | $0.24 | 12,085 |
| **C52895** | BSS138 N-MOSFET | 1 | $0.062 (min 10) | $0.06 | 15,550 — use C52895, NOT C112739 (wrong part) |

### Precision Passives

| LCSC# | Description | Qty | Unit (USD) | Extended | Stock |
|---|---|---|---|---|---|
| C5375456 | 0.5mΩ 2512 shunt | 2 | $0.059 | $0.12 | 55,800 |
| C133937 | FCM1608KF-601T ferrite 0603 | 1 | $0.011 (min 50) | $0.56 | 288,450 |
| C5181477 | 8MHz crystal 3225 | 1 | $0.132 (min 5) | $0.66 | 49,590 |
| C135264 | SMNR4020-22UH inductor | 1 | $0.063 (min 10) | $0.63 | 23,470 |
| C338088 | 4.7µF 100V MLCC 1210 | 3 | $0.724 | $2.17 | 462 |
| C131403 | EEEHB2A471P 470µF 100V | 2 | ~$0.90 est | ~$1.80 | ❌ 404 on LCSC — source from Mouser/DigiKey |
| C137962 | 4.7Ω gate resistor 0402 | 6 | $0.002 (min 100) | $0.24 | ❌ OUT OF STOCK — substitute needed |
| C13564 | 10kΩ NTC 0603 | 1 | $0.066 | $0.07 | 393,290 |

### Connectors

| LCSC# | Description | Qty | Unit (USD) | Extended | Stock |
|---|---|---|---|---|---|
| C10418 | USB Micro-B | 1 | $0.083 (min 5) | $0.42 | 37,815 |
| C157926 | JST-PH 4P TH | 1 | $0.069 (min 10) | $0.69 | 53,760 |
| C157920 | JST-PH 6P TH | 1 | $0.071 (min 10) | $0.71 | 132,960 |
| C157929 | JST-PH 3P TH | 1 | $0.049 (min 10) | $0.49 | 216,780 |
| C2337 | 2.54mm 1×40 header | 1 | $0.156 (min 5) | $0.78 | 79,980 |

### Basic Parts (0603/0402 passives — no Extended fee, near-zero cost)

| LCSC# | Description | Package | Status |
|---|---|---|---|
| C21189 | 0Ω resistor | 0603 | Basic |
| C22775 | 100Ω resistor | 0402 | Basic |
| C25804 | 10kΩ resistor | 0603 | Basic |
| C14663 | 100nF cap X7R | 0603 | Basic |
| C23630 | 2.2µF cap | 0603 | Basic |
| C1644 | 15pF cap | 0603 | Basic |
| (all other standard 0402/0603 passives from CLAUDE.md table) | | | Basic |

**Note:** C21189 (0Ω), C25804 (10kΩ), C14663 (100nF) are confirmed 0603 packages — verify schematic footprints match if 0402 was assumed anywhere.

### Component subtotal (for 5 boards)

| Category | 5-board cost |
|---|---|
| ICs + active | ~$105 |
| MOSFETs | ~$56 |
| Discretes + shunts | ~$7 |
| Precision passives | ~$29 |
| Connectors | ~$16 |
| Basic passives (bulk) | ~$10 |
| **Total components** | **~$223** |

(Includes min-order overages; excludes OOS parts BMI160 + NRF51822 + 470µF cap.)

---

## Stock Flags — Action Required Before Ordering

| Part | LCSC# | Issue | Action |
|---|---|---|---|
| BMI160 IMU | C94021 | OUT OF STOCK | Find alternate: ICM-42688-P (C2825881) or LSM6DSO (C310948) — check VESC firmware compatibility |
| NRF51822 module (EYSGJNZWY) | C2151959 | OUT OF STOCK | Check Mouser/DigiKey; NRF51822-QFAA bare die may need different module; consider NRF52 upgrade |
| AD8418ARMZ | C99534 | Only 31 pcs — enough for 5 boards (need 10 pcs) but no reorder buffer | Buy now; flag for production |
| 4.7Ω gate resistor | C137962 | OUT OF STOCK | Sub: YAGEO RC0402FR-074R7L (search LCSC for 4.7Ω 0402 Basic part) |
| 470µF 100V electrolytic | C131403 | 404 on LCSC | Source EEEHB2A471P from Mouser (€0.74 each); or find LCSC alternate (search "470µF 100V SMD") |
| IRFS4115TRLPBF | C2692945 | Wrong product at this LCSC# | Reassign to **C53417** in all schematic LCSC properties |
| BSS138 | C112739 | Wrong product at this LCSC# | Reassign to **C52895** in all schematic LCSC properties |

---

## Shipping and Taxes

### JLCPCB → Estonia

| Item | Rate | Basis | Cost |
|---|---|---|---|
| DHL Express from JLCPCB | ~$35 | ~0.8 kg gross package | $35 |
| EU customs duty on PCBA | 0% (HS 8543.70) | | $0 |
| Estonia VAT | 22% | On (goods + shipping) | $72 |

### LCSC standalone orders (if sourcing separately)

- LCSC ships to EU via DHL/FedEx; minimum order value ~$20 for free shipping on some routes
- For standalone western distributor parts (EEEHB2A471P etc): Mouser EU ships from Netherlands, no customs, 22% VAT included

---

## Sensitivity Analysis

| Scenario | Per-board (EUR) | Driver |
|---|---|---|
| Optimistic — HASL, all parts on LCSC | €61 | Lowest PCB + assembly cost |
| **Central estimate** | **€70** | As above |
| + ENIG surface finish | €80 | PCB finish upgrade |
| + BMI160/NRF from western distributor | €85 | +€10–15 for OOS parts |
| Pessimistic — ENIG + OOS parts + slower shipping | €92 | All extras combined |

---

## LCSC Number Corrections (applied to schematic 2026-06-30)

Two documented LCSC numbers pointed to wrong products — corrected in schematic:

| Component | Wrong LCSC# | Correct LCSC# | Status |
|---|---|---|---|
| IRFS4115TRLPBF (MOSFETs ×6) | C2692945 (was thin-film resistor) | **C53417** | ✅ Fixed in Power.kicad_sch |
| BSS138 (Q1 level shifter) | C112739 (was KIA431AM voltage ref) | **C52895** | ✅ Fixed in DRV8301.kicad_sch |

---

## Alternative Parts for Out-of-Stock / Obsolete Components

### BMI160 IMU — C94021 (OBSOLETE — discontinued April 2025)

**Drop-in alternative: LSM6DS3TR-C (C967633)**

| Field | BMI160 (original) | LSM6DS3TR-C (alternative) |
|---|---|---|
| LCSC# | C94021 (obsolete) | **C967633** |
| Package | LGA-14, 2.5×3mm | LGA-14, 2.5×3mm — **identical footprint** |
| Interface | I2C/SPI | I2C/SPI |
| Supply | 3.3V | 3.3V |
| Stock | 0 | 13,694 |
| Price | — | $1.58 @1 |
| VESC firmware | `BMI160` driver | `LSM6DS3` driver — **already in VESC bldc repo** |

No schematic changes needed. No footprint changes needed. At bring-up: select "LSM6DS3" in VESC Tool IMU settings instead of "BMI160".

Secondary: BMI270 (C2836813, 11,163 stock, $3.27) — same footprint but no VESC driver; requires firmware port.

**Decision:** swap schematic LCSC# to C967633 before ordering. Update VESC Tool IMU type at bring-up.

---

### NRF51822 Module EYSGJNZWY — C2151959 (OUT OF STOCK on LCSC)

No LCSC alternative exists. Source options:

| Source | Notes | Price |
|---|---|---|
| AliExpress — search "EYSGJNZWY" | Multiple sellers, same module | ~$4–6 |
| Flipsky — NRF51822 BLE module | Pre-flashed with nrf51_vesc firmware | ~$6 |

Any NRF51822 module that accepts the `nrf51_vesc` firmware and has the same UART/VCC/GND connections works. No schematic change required.

Future-proofing: VESC-Express (ESP32-C3) is the supported upgrade path for VESC 6.x wireless. Different footprint — requires schematic change.

---

### 4.7Ω 0402 Gate Resistor — C137962 (OUT OF STOCK on LCSC)

Part (YAGEO RC0402FR-074R7L) is available from western distributors — consign to JLCPCB.

| Distributor | MPN | Stock | Price @100 |
|---|---|---|---|
| DigiKey | RC0402FR-074R7L | >100k | ~$0.01 |
| Mouser | RC0402FR-074R7L | >100k | ~$0.01 |

Do not substitute a different resistance value — 4.7Ω is a deliberate gate drive impedance choice. Ship parts to JLCPCB before assembly order.

---

### 470µF 100V Electrolytic Bus Cap — C131403 (NOT ON LCSC)

**Primary (through-hole, recommended): AISHI ERS1KM471L25OT (C721247)**

| Field | Value |
|---|---|
| LCSC# | **C721247** |
| Value | 470µF, 100V, 105°C |
| Package | Through-hole, D16×L25mm |
| KiCad footprint | `Capacitor_THT:CP_Radial_D16.0mm_P7.50mm` |
| Stock | 10,248 |
| Price | $0.31 @100 |

Use this when placing MC-003 (bus electrolytics) in the schematic. Through-hole preferred over SMD here — the bulk capacitors handle inrush and need good mechanical retention.

SMD alternative (risky stock): C487447 (VKMJ2102A471MV, 45 units — insufficient for production).
