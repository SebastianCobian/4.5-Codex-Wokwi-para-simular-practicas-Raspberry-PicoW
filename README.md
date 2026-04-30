# Raspberry Pi Pico W Keypad-to-LED Controller

This project runs on a **Raspberry Pi Pico W (RP2040)** and maps a **4x4 matrix keypad** to **12 LEDs**.
Each key press turns on/off specific LEDs according to fixed logic implemented in `src/main.cpp`.

> The firmware source is preserved from the provided code and intentionally keeps the original behavior.

## Repository Structure

- `src/main.cpp` - Main firmware logic (Arduino-style C++).
- `include/` - Reserved for future headers.
- `docs/wiring.md` - Hardware components, wiring, and GPIO mapping.
- `docs/architecture.md` - Firmware architecture and behavior mapping.
- `CMakeLists.txt` - Minimal scaffold for repository consistency.

## Features

- 4x4 keypad input scanning via `Keypad` library.
- 12 independently controlled LEDs.
- Group control shortcuts:
  - `9` -> turn on LEDs mapped to keys `1..8`
  - `0` -> turn off LEDs mapped to keys `1..8`
  - `*` -> turn on LEDs mapped to `A..D`
  - `#` -> turn off LEDs mapped to `A..D`

## Hardware/Software Targets

- **Board**: Raspberry Pi Pico W
- **Framework style used by source**: Arduino-compatible C++ (`setup()` / `loop()`)
- **Simulation**: Wokwi
- **Real hardware**: Pico W + keypad + LEDs + resistors

## Build/Flash Options

Because the supplied firmware uses Arduino APIs (`pinMode`, `digitalWrite`, `delay`, `Keypad`), the easiest path is Arduino-compatible tooling.

### Option A - Wokwi (Recommended)

1. Create/open a Wokwi project with **Raspberry Pi Pico** or **Pico W**.
2. Paste `src/main.cpp` into the sketch source.
3. Ensure `diagram.json` reflects the wiring in `docs/wiring.md`.
4. Start simulation.

### Option B - Real Pico W using Arduino IDE

1. Install Arduino IDE 2.x.
2. Install a Pico core compatible with RP2040 (e.g., Arduino-Pico).
3. Install library: `Keypad`.
4. Open/import `src/main.cpp` as sketch source.
5. Select board: **Raspberry Pi Pico W**.
6. Put Pico W in BOOTSEL mode and upload.

### About `CMakeLists.txt`

A minimal `CMakeLists.txt` is included only to keep a C/C++-oriented repository layout. The provided program itself is Arduino-style, not native Pico SDK style.

## Wi-Fi Notes

Current firmware does **not** use Wi-Fi APIs. No SSID/password setup is required.
If Wi-Fi is added later, store credentials in ignored local config files and never commit secrets.

## Safety Notes

- Use one resistor per LED (220 Ω in this design).
- Share common GND between all modules.
- Validate external wiring before powering physical hardware.
