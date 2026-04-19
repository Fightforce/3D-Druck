# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository manages Marlin 1.1.9 firmware for an Anet A8 3D printer (ATmega1284p, 8-bit). The primary goal is safety: the stock Anet A8 firmware lacks thermal runaway protection (fire risk), and Marlin 1.1.9 fixes this. Marlin 2.x is NOT compatible with this processor — 1.1.9 is the correct final version for 8-bit boards.

## Build and Flash

Arduino IDE is installed at `/home/eddy/Programme/arduino`.

**Compile firmware:**
```bash
/home/eddy/Programme/arduino/arduino --board Sanguino:avr:sanguinousb:cpu=atmega1284p --verify tmp/Anet_A8_Firmware/Marlin-1.1.9/Marlin/Marlin.ino
```

**Flash via USBasp ISP programmer:**
```bash
avrdude -c usbasp -p m1284p -U flash:w:firmware.hex:i
```

**Flash via serial (after bootloader installed):**
```bash
avrdude -c arduino -p m1284p -P /dev/ttyUSB0 -b 57600 -U flash:w:firmware.hex:i
```

**Install bootloader (requires USBasp):**
```bash
avrdude -c usbasp -p m1284p -U lfuse:w:0xFF:m -U hfuse:w:0xDA:m -U efuse:w:0xFD:m -U flash:w:bootloader.hex:i
```

**Reset EEPROM after flashing (send via serial at 115200 baud):**
```
M502  ; Load firmware defaults
M500  ; Save to EEPROM
```

**Check serial port:**
```bash
ls /dev/ttyUSB* /dev/ttyACM*
dmesg | tail -20
```

**Grant serial port access:**
```bash
sudo usermod -a -G dialout $USER
```

## Configuration

All hardware parameters are in two files (when firmware is present in `tmp/`):
- `tmp/Anet_A8_Firmware/Marlin-1.1.9/Marlin/Configuration.h` — board type, bed size, extruder count, serial baud rate, display type
- `tmp/Anet_A8_Firmware/Marlin-1.1.9/Marlin/Configuration_adv.h` — thermal protection parameters, motion settings, language

Key settings to verify when reconfiguring:
- `MOTHERBOARD` → `BOARD_ANET_10`
- `THERMAL_PROTECTION_HOTENDS` and `THERMAL_PROTECTION_BED` → must be enabled
- `X_BED_SIZE 220`, `Y_BED_SIZE 220`, `Z_MAX_POS 240`
- `BAUDRATE 115200`

Pre-tested configuration templates for the Anet A8 are in `tmp/Anet_A8_Firmware/Marlin-1.1.9/Configurations/`.

## Architecture

```
Anet A8 neu aufsetzten 2026/ # German-language setup guides (Markdown)
Bauteile AnetA8/             # Empty — intended for component specs/datasheets
tmp/                         # Not versioned — firmware source and build artifacts
  Anet_A8_Firmware/          # Marlin 1.1.9 source (download from GitHub if missing)
  build/                     # Compiled firmware output
```

Arduino IDE: `/home/eddy/Programme/arduino` (system-wide, not in repo)

The firmware uses board abstraction via `pins_ANET_10.h` and the `BOARD_ANET_10` define. Thermistor calibration tables (for hotend and bed sensors) are defined in `thermistortables.h`.

## Documentation

Detailed German-language guides are in `Anet A8 neu aufsetzten 2026/`:
- `Anet_A8_Firmware_Update_Leitfaden.md` — complete step-by-step update guide
- `Anet_A8_Linux_Befehle_Referenz.md` — quick-reference bash commands with troubleshooting table
