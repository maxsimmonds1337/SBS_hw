# Bill of Materials — SBS_hw v1.0
**Date:** 2026-06-25  
**Status:** DRAFT — requires user review and approval before ordering  
**All prices:** approximate LCSC single-unit price (bulk pricing is lower)

---

## ⚠️ Critical Flags — Resolve Before Ordering

| # | Flag | Impact |
|---|---|---|
| **F1** | **MOSFET Vds violation:** IRF7749L1TRPBF (40V) cannot survive 58.8V bus. Must replace with IRFS4115TRLPBF (150V). Package changes from DirectFET L8 → D2PAK — footprint update required in PCB layout. | Hard block on layout |
| **F2** | **Bootstrap diode voltage:** PMEG6020ER (60V) insufficient for 58.8V bus. Replace with DSS210 (100V 2A). Package changes from SOD-123W → SOD-123FL — footprint update required. | Hard block on layout |
| **F3** | **Bulk cap rating:** C6 is 100µF / 63V — insufficient for 58.8V (derating rule requires ≥80V, prefer 100V). Replace before layout. | Hard block on layout |
| **F4** | **DRV8301 package mismatch:** Schematic footprint is PVQFN-40 (DRV8301DCPR), but only the HTSSOP-56 variant (DRV8301DCAR, C98969) is currently stocked on LCSC. Decide: update footprint to HTSSOP-56 and use C98969, OR source DRV8301DCPR from Mouser/Digikey. | Must resolve before BOM lock |
| **F5** | **BMI160 missing from schematic:** All sheets searched — BMI160 (balance IMU) is not placed. VESC Balance App requires it. Must be added to MCU sheet (I2C: SDA→PB7, SCL→PB6; INT→MCU GPIO). LCSC: C94021. | Hard block on function |
| **F6** | **MC74VHC1GT66 not stocked on LCSC:** Could not confirm a direct listing. Search LCSC for "74VHC1G66" — if not found, substitute SN74LVC1G66DBVR (TI, SOT-23-5; same function, pin-compatible). | Confirm before ordering |

---

## BOM Table

### Integrated Circuits

| Ref | Value / Description | LCSC# | Pkg | Qty | Unit price (£) | Notes |
|---|---|---|---|---|---|---|
| U1 (MCU) | STM32F405RGT6 | **C15742** | LQFP-64 | 1 | ~£2.05 | Commercial temp (0–70°C). Exact match to firmware target `60_mk5` |
| U (DRV8301) | DRV8301DCAR — 3-phase gate driver | **C98969** | HTSSOP-56 | 1 | ~£1.10 | ⚠️ F4: schematic footprint is PVQFN-40. Must update footprint to HTSSOP-56 to use this part, OR source DRV8301DCPR from Mouser. |
| U4, U5 | AD8418ABRMZ-RL — bidirectional current sense amp | **C99534** | MSOP-8 | 2 | ~£1.55 ea | 20V/V gain, 70V CMR, pairs with 0.5mΩ shunts |
| U (CAN) | TCAN1051VDQ1 — CAN transceiver | **C133788** | SOIC-8 | 1 | ~£0.27 | TI, automotive-grade, 5V supply |
| U (LDO) | TC2117-3.3VDBTR — 3.3V 800mA LDO | **C98655** | SOT-223-3 | 1 | ~£0.31 | Powers 3.3V logic rail from DRV8301 5V buck |
| U (BMI160) | BMI160 — 6-axis IMU | **C94021** | LGA-14 | 1 | ~£2.67 | ⚠️ F5: NOT IN SCHEMATIC — must be added. Required for Balance App. |
| U (NRF) | EYSGJNZWY — Taiyo Yuden NRF51822 module | **C2151959** | SMD-28 | 1 | ~£3.62 | Same as VESC 6 MK5 reference. EOL risk noted (see vesc_firmware_notes.md §4) |
| U3 | MC74VHC1GT66 — single analog switch | **⚠️ Search LCSC** | SC-88A/SOT-353 | 1 | ~£0.05 | ⚠️ F6: search "74VHC1G66". If unavailable, substitute SN74LVC1G66DBVR (search LCSC for "SN74LVC1G66DBVR") |

### Power Stage — MOSFETs (Critical Replacement)

| Ref | Value / Description | LCSC# | Pkg | Qty | Unit price (£) | Notes |
|---|---|---|---|---|---|---|
| Q1–Q6 | **IRFS4115TRLPBF** — 150V 195A N-ch MOSFET | **C53417** | D2PAK (TO-263-2) | 6 | ~£0.93 ea | ⚠️ F1: REPLACES IRF7749L1TRPBF (40V). Rds(on) = 12.1mΩ @ Vgs=10V. 150V > 58.8V bus ✓. Footprint change required. |

### Discrete Semiconductors

| Ref | Value / Description | LCSC# | Pkg | Qty | Unit price (£) | Notes |
|---|---|---|---|---|---|---|
| D1 (TVS) | SMAJ5.0A-13-F — 5V TVS | **C87074** | SMA (DO-214AC) | 1 | ~£0.04 | On DRV8301 sheet, 3.3V rail protection |
| D2–D4 (bootstrap) | **DSS210** — 100V 2A Schottky | **C511868** | SOD-123FL | 3 | ~£0.02 ea | ⚠️ F2: REPLACES PMEG6020ER (60V). 100V > 58.8V bus ✓. Footprint changes from SOD-123W → SOD-123FL. |
| D (USB VBUS Schottky) | Generic Schottky, 40V+ 1A | Search LCSC "SS14" | SMA or SOD-123 | 2 | ~£0.02 ea | VBUS protection diode on USB connector circuit |
| D (Zener) | Generic Zener, ~5.1V | Search LCSC "BZX84C5V1" | SOD-323 | 1 | ~£0.01 | USB D+/D− ESD protection |
| D (LED) | Red LED 0402 | Search LCSC "red LED 0402" | 0402 | 1 | ~£0.01 | Status indicator |

### Shunt Resistors

| Ref | Value / Description | LCSC# | Pkg | Qty | Unit price (£) | Notes |
|---|---|---|---|---|---|---|
| R9, R? | 0.5mΩ 3W 1% shunt | **C5375456** (HoYLR2512E-3W-0.5mR-1%) | 2512 | 2 | ~£0.02 ea | 3W rating: at 80A peak, P = 80²×0.5mΩ = 3.2W — 3W part handles bursts; 2W (C500611) is marginal |

### Passives — Precision / Specific

| Ref | Value / Description | LCSC# | Pkg | Qty | Unit price (£) | Notes |
|---|---|---|---|---|---|---|
| FB (ferrite bead) | FCM1608KF-601T05 — 600Ω @100MHz | **C133937** | 0603 | 1 | ~£0.004 | MCU VDD filter |
| Y (crystal) | 8MHz 20pF ±10ppm crystal | **C5181477** (SOSET 3225 8M 20PF 10PPM) | 3225-4P | 1 | ~£0.04 | STM32 HSE clock. Load capacitor: 20pF; pair with 15pF load caps |
| L? (inductor) | 22µH SMD inductor, ≥1A | Search LCSC "22uH 1A 0806 inductor" | 1008/0806 | 1 | ~£0.05 | DRV8301 internal buck converter output filter |
| C6 (bulk cap) | **100µF 100V** SMD electrolytic | Search LCSC "100µF 100V aluminum SMD" | D10×10mm | 1 | ~£0.15 | ⚠️ F3: replaces current 63V part. Must be ≥80V, prefer 100V. Size ~D10×10.5mm |

### Passives — Resistors (bulk, standard 0402/0603)

These are standard values — source from any LCSC generic resistor listing (0.5% or 1% tolerance, 100mW).

| Value | Pkg | Qty (approx) | Use |
|---|---|---|---|
| 4.7Ω | 0402 | 6 | MOSFET gate resistors (one per gate) |
| 39kΩ (60V-rated) | 0603 | 3 | Phase voltage sense dividers — use 0603 for 1/8W |
| 2.2kΩ | 0603 | 4 | DRV8301 signal pull-up/down |
| 10kΩ | 0402 | 3 | CAN pull-up, general purpose |
| 220Ω | 0603 | 1 | CAN bus termination |
| 22Ω | 0402 | 2 | USB D+/D− series resistors |
| 56kΩ | 0402 | 1 | USB ID resistor |
| 1kΩ | 0402 | 2 | LED current limit |
| 100Ω | 0402 | 1 | General |
| 1kΩ PTC | 0402 | 2 | Overcurrent sense |
| 0Ω | 0402 | 1 | Config jumper |
| 2kΩ | 0402 | 1 | DRV8301 CSB pull-up |

### Passives — Capacitors (bulk, standard 0402/0603, X5R/X7R)

| Value | Voltage | Pkg | Qty (approx) | Use |
|---|---|---|---|---|
| 100nF | 16V+ | 0402 | 10 | Bulk MCU decoupling |
| 2.2µF | 16V+ | 0603 | 6 | MCU VDD filtering |
| 15pF | 50V | 0603 | 2 | Crystal load caps |
| 15nF | 25V | 0603 | 2 | DRV8301 bootstrap caps |
| 4.7µF | 100V | 0603/0805 | 3 | DRV8301 bootstrap caps (100V rating required) |
| 220nF | 25V | 0603 | 2 | DRV8301 bypass |
| 2.2µF | 16V | 0603 | 4 | CAN / general bypass |
| 100nF | 25V | 0603 | 4 | CAN / general bypass |

### Connectors

| Ref | Value / Description | LCSC# | Qty | Unit price (£) | Notes |
|---|---|---|---|---|---|
| J (USB) | USB Micro-B SMD 5-pin | **C10418** (Jing MICRO-USB-5S-B) | 1 | ~£0.08 | VESC Tool connection |
| J (HALL_ENC) | JST-PH 1.0mm 5-pin R/A | Search LCSC "JST PH 5pin 1.0mm SMD" | 1 | ~£0.08 | Hall sensor input from PHUB-188 |
| J (SERVO/COMM) | 2.54mm header 3–6 pin | Generic on LCSC | 3 | ~£0.05 ea | SERVO, COMM headers |
| J (SWD) | 1.27mm or 2.54mm 4-pin header | Generic on LCSC | 1 | ~£0.05 | SWD debug / flash |
| J (CAN) | 2.54mm 2-pin | Generic on LCSC | 1 | ~£0.04 | CAN H/L output |

---

## Estimated BOM Cost (single board, approx)

| Category | Approx. cost (£) |
|---|---|
| ICs (STM32, DRV8301, AD8418×2, TCAN, TC2117, BMI160, NRF module) | ~£13.60 |
| MOSFETs (6× IRFS4115) | ~£5.60 |
| Discrete semis (TVS, bootstrap diodes, LED, protection diodes) | ~£0.50 |
| Shunt resistors | ~£0.05 |
| Precision passives (crystal, ferrite, inductor, bulk cap) | ~£0.30 |
| Bulk passives (resistors, caps) | ~£1.00 |
| Connectors | ~£0.50 |
| **Total (approx)** | **~£21.50** |

> Prices are single-unit LCSC. At 5–10 boards (JLCPCB order quantity), expect ~20–30% reduction on most lines. Exchange rate used: 1 USD ≈ 0.79 GBP.

---

## Parts to Source Elsewhere

| Part | Reason | Suggested source |
|---|---|---|
| DRV8301DCPR (PVQFN-40) | Not found on LCSC — only HTSSOP-56 (DCAR) available. Resolve F4 first before deciding which to source. | Mouser / Digikey |
| PHUB-188 hub motor | Mechanical component | peipeiscooter.com (per blog0) |
| Hall sensor cable (motor) | Comes with motor | Included with PHUB-188 |
| XT90-S antispark connector pair | Not carried by LCSC | AliExpress / Hobbyking |
| Molicel P42A cells × 28 | Li-ion cells | NKON, Fogstar, 18650uk |
| Daly Smart BMS 14S 100A | Not a PCB component | AliExpress / Battery Hookup |
| 58.8V 5A CC/CV charger | Mechanical/PSU | AliExpress (14S lithium charger) |

---

## Approval Checklist

Before placing an LCSC order, confirm the following:

- [ ] **F1 resolved** — MOSFET replacement confirmed (IRFS4115TRLPBF C53417 approved)
- [ ] **F2 resolved** — Bootstrap diode replacement confirmed (DSS210 C511868 approved, or alternative)
- [ ] **F3 resolved** — Bulk cap 100µF/100V part selected and added to BOM
- [ ] **F4 resolved** — DRV8301 package decision: update footprint to HTSSOP-56 (use C98969) OR source DCPR from Mouser
- [ ] **F5 resolved** — BMI160 added to schematic MCU sheet, LCSC C94021 added to BOM
- [ ] **F6 resolved** — MC74VHC1GT66 / SN74LVC1G66 confirmed in stock
- [ ] Exact passive quantities verified by running a BOM export from KiCad
- [ ] Footprints for all replaced parts updated in schematic before PCB layout
- [ ] 4.7µF bootstrap caps confirmed as 100V-rated in schematic

---

*Sources used for LCSC part lookups:*  
*DRV8301DCAR: [C98969](https://www.lcsc.com/product-detail/Motor-Driver-ICs_Texas-Instruments_C98969.html) · STM32F405RGT6: [C15742](https://www.lcsc.com/product-detail/C15742.html) · AD8418ABRMZ: [C99534](https://lcsc.com/product-detail/Current-Sensing-Amplifiers_Analog-Devices-AD8418ABRMZ_C99534.html) · TCAN1051VDQ1: [C133788](https://www.lcsc.com/product-detail/CAN_TI_TCAN1051VDQ1_TCAN1051VDQ1_C133788.html) · PMEG6020ER (existing): [C98663](https://www.lcsc.com/product-detail/Schottky-Barrier-Diodes-SBD_Nexperia-PMEG6020ER-115_C98663.html) · TC2117-3.3VDBTR: [C98655](https://lcsc.com/product-detail/Linear-Voltage-Regulators-LDO_Microchip-Tech-TC2117-3-3VDBTR_C98655.html) · SMAJ5.0A-13-F: [C87074](https://www.lcsc.com/product-detail/TVS_Diodes-Incorporated-SMAJ5-0A-13-F_C87074.html) · IRFS4115TRLPBF: [C53417](https://www.lcsc.com/product-detail/C53417.html) · DSS210: [C511868](https://www.lcsc.com/product-detail/Schottky-Diodes_BORN-DSS210_C511868.html) · BMI160: [C94021](https://www.lcsc.com/product-detail/Sensors_Bosch_BMI160_BMI160_C94021.html) · EYSGJNZWY: [C2151959](https://www.lcsc.com/datasheet/lcsc_datasheet_2411271937_Taiyo-Yuden-EYSGJNZWY_C2151959.pdf) · 0.5mΩ shunt: [C5375456](https://www.lcsc.com/product-detail/current-sense-resistors-shunt-resistors_milliohm-hoylr2512e-3w-0-5mr-1_C5375456.html) · FCM1608KF-601T05: [C133937](https://www.lcsc.com/product-detail/ferrite-beads_tai-tech-fcm1608kf-601t05_C133937.html) · USB micro-B: [C10418](https://www.lcsc.com/product-detail/Micro-USB-Connectors_MICRO-USB-5S-B-Type-horns-High-temperature_C10418.html)*
