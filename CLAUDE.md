# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a ZMK firmware configuration repository for split keyboards (Corne, Corne Min, and Ferris Sweep/Cradio). The Corne and Ferris use Nice!Nano v2 microcontrollers; the Corne Min uses its own boards from `MechboardsLTD/zmk-module`, and can optionally be driven by a Seeeduino Xiao BLE acting as a Prospector dongle. Firmware is built via GitHub Actions—there is no local build process.

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
- `boards/shields/` - Custom shield definitions
  - `corne_min_dongle/` - Prospector dongle shield for the Corne Min (Kconfig + overlay)
- `build.yaml` - GitHub Actions build matrix
- `zephyr/module.yml` - Zephyr module definition

## Keyboard Configurations

**Corne** (`corne.conf`, `corne.keymap`):
- Has Nice!View display (requires `cs-gpios = <&pro_micro 8 GPIO_ACTIVE_HIGH>` in keymap for mechboards.uk PCB)
- RGB disabled, display widgets enabled (battery %, output status, layer)
- 3 layers (default, lower, raise)
- Driven by a Nice!Nano dongle: `nice_nano` + the `dongle_corne` shield is the split central (mock kscan + a matrix transform copied from ZMK's corne `default_transform`), and both halves are rebuilt as peripherals with `-DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n`. The halves keep their stock transforms (the right half's `col-offset = <6>` is what puts its keys in the right half of the 42-key map), so no position remapping is needed on the dongle. ZMK Studio runs on the dongle, since only the central holds the keymap.
- The dongle shield is named `dongle_corne`, **not** `corne_dongle`: ZMK derives candidate `.conf`/`.keymap` names by stripping trailing `_`-separated pieces off the shield name, so a `corne_*` shield also matches `corne.keymap` — which is checked *before* the shield's own name and overrides `&nice_view_spi`, a label that only exists in nice_view builds. Any new Corne-adjacent shield needs a name that does not reduce to `corne`.

**Corne Min** (`corne_min.conf`, `corne_min.keymap`):
- Uses `corne_min_left` / `corne_min_right` boards from the `MechboardsLTD/zmk-module:corne-min` branch with the `rgbled_adapter` shield
- No display; same 42-key layout and keymap as the standard Corne (copied directly)
- Can be driven by a Prospector dongle: `seeeduino_xiao_ble` board with `corne_min_dongle prospector_adapter` shields (central), and halves rebuilt as peripherals via `-DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n`
- Prospector module pulled from `carrefinho/prospector-zmk-module:main` via `west.yml`. ZMK itself is pinned to the `v0.3` release tag (Zephyr 3.5 / LVGL 8), matching beekeeb's reference config `beekeeb/zmk-config-hshs-with-prospector`. ZMK main (Zephyr 4.1 / HWMv2) is incompatible with MechboardsLTD/zmk-module's `corne-min` board, which uses the HWMv1 layout (`boards/arm/corne_min/`). The pin also keeps `BOARD=nice_nano` resolving for the `nice_view_adapter` shield (4.1 requires the qualified `nice_nano_nrf52840_zmk`). If MechboardsLTD ever ships an HWMv2 Corne Min, ZMK can move back to `main` and the prospector revision should move to `core/zephyr-4-1`.
- Brightness is pinned (`CONFIG_PROSPECTOR_FIXED_BRIGHTNESS=80`, ALS off) in `corne_min.conf`
- Includes a `settings_reset` build for both the prospector and the left half

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
