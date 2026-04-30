# Firmware Architecture

## Overview

The firmware is a single-module Arduino-style program in `src/main.cpp`.
It initializes GPIO outputs for 12 LEDs and continuously polls a 4x4 keypad.

## Static Configuration

- `LEDS = 12`, `ROWS = 4`, `COLS = 4`
- `keys[4][4]`: keypad character map.
- `ledPins[12]`: GPIOs assigned to LED channels.
- `rowPins[4]`, `colPins[4]`: keypad wiring map.

## Runtime Flow

1. **setup()**
   - Configures each pin in `ledPins` as `OUTPUT`.
   - Sets all LEDs to `LOW`.

2. **loop()**
   - Reads current key using `keypad.getKey()`.
   - If a key is pressed, applies switch-case actions.
   - Waits `10 ms` (`delay(10)`) before next scan.

## Behavior Map

- Direct ON actions:
  - `1..8` -> turn ON corresponding blue LED channel.
  - `A..D` -> turn ON corresponding red LED channel.
- Group actions:
  - `9` -> turn ON channels `0..7`.
  - `0` -> turn OFF channels `0..7`.
  - `*` -> turn ON channels `8..11`.
  - `#` -> turn OFF channels `8..11`.

## Notes on Non-Modified Logic

No changes were made to key mapping, GPIO assignment arrays, delays, or key action semantics.
The project improvements are repository organization and documentation only.
