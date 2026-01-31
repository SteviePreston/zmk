# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a ZMK firmware configuration repository for split keyboards (Corne and Ferris Sweep/Cradio) using Nice!Nano v2 microcontrollers. Firmware is built via GitHub Actions—there is no local build process.

## Build Process

Firmware builds are triggered automatically on push/PR via GitHub Actions. The workflow (`.github/workflows/build.yml`) uses ZMK's official `build-user-config.yml` reusable workflow (pinned to v0.3). Build artifacts (`.uf2` files) are available in the GitHub Actions run artifacts.

The build matrix is defined in `build.yaml` and specifies:
- Board/shield combinations for each keyboard half
- ZMK Studio support for left halves (enables real-time keymap editing)
- Nice!View display support for Corne

## Repository Structure

- `config/` - Keyboard configurations
  - `*.conf` - ZMK settings (display, sleep, Bluetooth power)
  - `*.keymap` - Key bindings and layers
  - `west.yml` - ZMK dependency manifest
- `build.yaml` - GitHub Actions build matrix
- `zephyr/module.yml` - Zephyr module definition

## Keyboard Configurations

**Corne** (`corne.conf`, `corne.keymap`):
- Has Nice!View display (requires `cs-gpios = <&pro_micro 8 GPIO_ACTIVE_HIGH>` in keymap for mechboards.uk PCB)
- RGB disabled, display widgets enabled (battery %, output status, layer)
- 3 layers with conditional layer support

**Ferris Sweep** (`cradio.conf`, `cradio.keymap`):
- No display
- Uses home row mods with custom `hm` hold-tap behavior:
  - `tapping-term-ms: 280` - time before hold triggers
  - `quick-tap-ms: 175` - enables fast repeated taps
  - `require-prior-idle-ms: 150` - prevents accidental holds during fast typing
  - `flavor: balanced` - hold on key-up if another key pressed

## Key Technical Details

- ZMK Studio is enabled on left halves only (via `snippet: studio-rpc-usb-uart`)
- Sleep timeouts: 15min idle (900000ms), 30min deep sleep (1800000ms)
- Bluetooth TX power set to +8dBm for better range
- Nice!View SPI configuration must be in the keymap file, not the conf file
- Keyboard names are limited to 16 characters

## Useful Resources

- [ZMK Documentation](https://zmk.dev/docs)
- [ZMK Studio](https://zmk.studio) - Real-time keymap editing
- [Keymap Editor GUI](https://nickcoutsos.github.io/keymap-editor)
