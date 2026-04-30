# Wiring and GPIO Mapping (Raspberry Pi Pico W)

## Components List (from Wokwi diagram)

- 1x Raspberry Pi Pico / Pico W
- 1x 4x4 membrane keypad
- 12x LEDs
  - 8 blue LEDs (labels `1..8` in diagram)
  - 4 red LEDs (labels `A..D` in diagram)
- 12x 220 Ω resistors (LED current limiting)
- 4x 1 kΩ resistors (keypad row pull-ups to 3V3)
- Jumper wires

## Power and Ground

- `pico:3V3` feeds keypad row pull-up resistor network (4x 1kΩ chain to R1-R4 lines).
- `pico:GND.4` is common ground for all LED cathodes.

## Keypad Connections

### Columns

| Keypad Pin | Pico W GPIO |
|---|---|
| C1 | GP19 |
| C2 | GP18 |
| C3 | GP17 |
| C4 | GP16 |

### Rows

| Keypad Pin | Pico W GPIO | Pull-up |
|---|---|---|
| R1 | GP26 | 1kΩ to 3V3 |
| R2 | GP22 | 1kΩ to 3V3 |
| R3 | GP21 | 1kΩ to 3V3 |
| R4 | GP20 | 1kΩ to 3V3 |

## LED Connections

Each LED anode is driven by a GPIO pin through a 220 Ω resistor; each cathode goes to GND.

| LED Label | Pico W GPIO | Source Array Index |
|---|---|---|
| 1 | GP11 | ledPins[0] |
| 2 | GP10 | ledPins[1] |
| 3 | GP9  | ledPins[2] |
| 4 | GP8  | ledPins[3] |
| 5 | GP7  | ledPins[4] |
| 6 | GP6  | ledPins[5] |
| 7 | GP5  | ledPins[6] |
| 8 | GP4  | ledPins[7] |
| A | GP3  | ledPins[8] |
| B | GP2  | ledPins[9] |
| C | GP28 | ledPins[10] |
| D | GP27 | ledPins[11] |

## Assumptions

- Diagram part is `wokwi-pi-pico`; project target is documented as Pico W and is pin-compatible for used GPIOs.
- LED labels in Wokwi are presentation labels; firmware behavior is driven by `ledPins[]` ordering.
