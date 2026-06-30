# Lighting Sub-Assembly

## Requirements
- Front: white headlight (visibility when riding at night / in low light)
- Rear: red tail/brake light
- Controlled by STM32F405 on SBS_hw via GPIO
- Waterproof (outdoor use, rain)
- Low power (<2W total — negligible on a 14S pack)

---

## LED Selection

### WS2812B Addressable LEDs (Recommended)
WS2812B (NeoPixel-compatible) are individually addressable RGB LEDs on a single-wire protocol. Advantages:
- Single data wire from MCU → entire strip
- Can set front white, rear red, turn signals, brake flash, etc., all in firmware
- Available in waterproof (IP65/IP67) strip form
- VESC firmware natively supports WS2812B on a GPIO pin (Status LED config)

**IP65 WS2812B 60 LED/m strip:**
- Source: AliExpress
- Price: ~$8–12 for 1m
- Use 10 LEDs (~17cm) for front, 5 LEDs (~8cm) for rear
- Cut from strip to length

**Power per LED at full white:** ~60mA at 5V = 300mW
At 10 LEDs: 600mA at 5V = 3W — feed from the SBS_hw 3.3V LDO output via a small 5V buck, or run at reduced brightness from 3.3V (WS2812B minimum is ~3.5V — marginal; use a small 5V boost or a separate DC-DC).

**Simpler option:** Use non-addressable LEDs
- 3× white 5mm LED (forward) + 2× red 5mm LED (rear), each with 100Ω resistor
- Drive from MCU GPIO via transistor (BSS138 already in BOM)
- Total ~50mA, no DC-DC needed
- Firmware: GPIO on/off, no special protocol

**Recommendation:** Non-addressable LEDs for simplicity (1 GPIO each direction), upgrade to WS2812B later if animations wanted.

---

## Mounting

- Front LEDs: recess into front face of the 3D printed enclosure nose
- Rear LEDs: recess into rear face of enclosure
- Weatherproof with silicone sealant around the LED opening
- Run wiring inside the aluminum rail channel

---

## Bill of Materials

| Item | Qty | Unit price | Total | Source |
|---|---|---|---|---|
| White LED 5mm (forward lights) | 3 | $0.20 | $0.60 | LCSC / AliExpress |
| Red LED 5mm (rear light) | 2 | $0.20 | $0.40 | LCSC / AliExpress |
| 100Ω resistor (current limiting) | 5 | $0.02 | $0.10 | LCSC (C22775) |
| BSS138 MOSFET (drive from 3.3V GPIO) | 2 | $0.10 | $0.20 | LCSC (C112739) — already in SBS_hw BOM |
| 2-core waterproof cable, 0.5m | 2 | $1 | $2 | AliExpress |
| Silicone sealant | 1 tube | $4 | $4 | Hardware store |
| **Total** | | | **~$7** | |

*Note: BSS138 is already in the SBS_hw motor controller BOM — no additional purchase needed.*
