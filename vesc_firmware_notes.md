# VESC Firmware Notes
**Project:** SBS_hw v1.0  
**Firmware branch:** VESC 6.x (vedderb/bldc, active development)  
**Last updated:** 2026-06-24

---

## 1. Key Features Relevant to This Project

### Motor control algorithms

| Feature | Description | Relevance |
|---|---|---|
| **FOC** | Sinusoidal field-oriented control — smooth, efficient | Mandatory for both platforms |
| **HFI** | High Frequency Injection — sensorless torque at zero speed | Robot dog stance/hold without encoder |
| **Field weakening** | Extend speed above base speed | Onewheel top-speed headroom |
| **MTPA** | Maximum Torque Per Ampere — efficiency at partial load | Useful for robot dog battery life |
| **Sensorless observer** | Multiple types (Ortega, MXLEMNA) — tuneable | Sensorless startup fallback |
| **Trapezoidal BLDC** | Classic commutation, less smooth | Fallback/testing mode |

### Sensor support

- Hall sensors (×3) — primary mode for startup on PHUB-188 hub motor
- ABI/ABZ encoder, AS5047, SinCos, BiSS-C, PWM, MA782 — optional for robot dog position control
- **BMI160 IMU explicitly supported** (also ICM20948, LSM6DS3, MPU9150)

### Apps

| App | Platform | Notes |
|---|---|---|
| **Balance App** | Onewheel ✓ | Built-in, well-tested by DIY onewheel community. Uses BMI160 pitch for control loop |
| **UART App** | Both | External controller sends commands over UART |
| **PPM/PWM App** | Both | RC receiver or servo signal input |
| **LispBM** | Robot dog ✓ | Sandboxed Lisp scripting on the VESC itself — custom leg control logic |
| **CAN Forwarding** | Robot dog ✓ | Transparent CAN relay between controllers |

### CAN bus (multi-axis, robot dog)

- Standard: ISO 11898, 1 Mbit/s
- Each VESC broadcasts status at configurable rate: RPM, current, voltage, position, temperature
- External controller (Raspberry Pi / Jetson) sends `set_current` or `set_rpm` commands per axis
- VESC IDs assigned in VESC Tool — up to 255 nodes on one bus
- This is the standard architecture for hobby quadrupeds using VESC

### LispBM scripting

Runs a sandboxed Lisp interpreter on the STM32F405 alongside the motor control loop. Useful for:
- Custom impedance/compliance control on a robot leg (runs locally on the VESC, low latency)
- Logging and telemetry
- Custom startup sequences
- Tuning Balance App behaviour without recompiling firmware

---

## 2. Suitability Assessment

### Platform B — Onewheel

**Verdict: fully supported out of the box.**

The Balance App is the core feature and has been used by hundreds of DIY onewheel builders. It reads the BMI160 (already in your schematic) for pitch, and commands motor current to maintain balance. Configuration is done in VESC Tool.

**Known limitation:** The Balance App control loop runs at **~1 kHz** using I2C BMI160. This is adequate for normal riding. For very aggressive dynamic riding, the community has noted this rate is at the lower end. The firmware notes that SPI-connected IMU could reach ~10 kHz, but I2C at 1 kHz is fine for a first build. Your schematic uses I2C — acceptable.

### Platform A — Robot Dog / Quadruped

**Verdict: suitable as a joint-level torque/velocity controller, with an external gait controller.**

VESC does not provide gait planning or leg kinematics — that lives on an external computer (Raspberry Pi, Jetson Nano, etc.) which sends torque or velocity commands to each VESC over CAN. This is the standard architecture.

Per-joint, VESC provides:
- FOC current (torque) control — essential for quasi-direct-drive compliance
- HFI for sensorless zero-speed torque (useful in stance)
- Position control with encoder feedback (optional upgrade path)
- LispBM for local impedance shaping if needed

---

## 3. VESC Tool Configuration Notes

For the onewheel (first bring-up checklist):

- [ ] Run motor detection wizard with PHUB-188 connected — measures R, L, flux linkage, KV
- [ ] Set motor type: BLDC (hub motor)
- [ ] Enable Hall sensor detection sequence
- [ ] Set current limits: motor max 80A, battery max 30A (BMS limit), regen max -30A
- [ ] Set voltage limits: cutoff start 42V (10S equivalent for 14S — adjust to 14S values), absolute min 39.2V
- [ ] Set temperature limits: motor 80°C, VESC 85°C
- [ ] Enable Balance App, configure PID gains (start with community defaults for 10" hub motor)
- [ ] Configure IMU: BMI160, calibrate with board level

For robot dog (per axis):

- [ ] Run FOC motor detection on X4114 at 6S
- [ ] Set unique CAN ID per board (0–3 for four legs)
- [ ] Set current limits appropriate for X4114 bench test results
- [ ] Enable HFI for sensorless zero-speed operation
- [ ] Confirm CAN status packets received on external controller

---

## 4. NRF51822 Module Status

The NRF51822 EYSGJNZWY used in the schematic is the same as the VESC 6 MK5 reference. It provides a proprietary VESC remote protocol (not standard BLE).

**Potential issue:** The NRF51822 was noted in the onewheel blog as potentially EOL. Verify availability. If unavailable:
- **NRF52832** — pin-compatible upgrade path, supported by VESC firmware. Requires new firmware for the module but the VESC host-side already supports it.
- The robot dog doesn't need the wireless module at all (CAN bus is the coordination link).

---

## 5. Firmware Build and Flashing

- **Toolchain:** ARM GCC, build via `make` or VS Code + CMake
- **Flash:** SWD via ST-Link V2 (cheap clones work fine) or OpenOCD
- **VESC Tool:** Pre-built binaries for Linux/Mac/Windows; connects over USB or UART for configuration
- **Custom app code:** Place in `applications/` directory, enabled via `VESC_CONF_MAIN_APP` in config

---

## 6. Links

- Firmware repository: github.com/vedderb/bldc
- VESC Tool: github.com/vedderb/vesc_tool
- VESC Project forums: vesc-project.com/forum
- Balance App community reference: pev.dev (good VESC onewheel guides)
