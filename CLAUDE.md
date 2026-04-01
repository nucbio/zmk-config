# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZMK firmware configuration for a **Corne Choc Pro** wireless split keyboard (NRF52840-based). This is a user config repo that layers on top of the main ZMK firmware via a west manifest (`config/west.yml`).

## Build System

Firmware is built via GitHub Actions. Push to any branch triggers `.github/workflows/build.yml`, which calls the official ZMK reusable workflow (`zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`).

**`build.yaml`** defines the build matrix — currently two targets:
- `corne_choc_pro_left` with `settings_reset` shield and `studio-rpc-usb-uart` snippet
- `corne_choc_pro_right` with the same

There is no local build; firmware is compiled in CI and downloaded as UF2 artifacts.

## Architecture

### Board definition (`boards/shields/corne_choc_pro/`)

Custom board (not a shield despite the directory path). Key files:

- **`corne_choc_pro.dtsi`** — base device tree: key matrix transforms (14×4 default, 12×4 five-col), SPI display, 4 rotary encoders, flash partitions
- **`corne_choc_pro_left.dts`** / **`_right.dts`** — per-half GPIO assignments, WS2812B RGB (23 LEDs via SPI3), MAX17048 fuel gauge, encoder enable
- **`corne_choc_pro-layouts.dtsi`** — physical layout position metadata for two layout variants
- **`Kconfig.defconfig`** — default firmware options (BLE, USB, split, RGB underglow, battery)
- **`*_defconfig`** — per-half Kconfig (enables encoders, RGB driver, ZMK Studio alpha, experimental BLE fixes)

### User config (`config/`)

- **`corne_choc_pro.keymap`** — 3-layer keymap (default QWERTY, symbols/numbers, controls) with encoder sensor bindings
- **`corne_choc_pro.conf`** — high-level feature toggles (keyboard name, split, USB, BLE, sleep)
- **`corne_choc_pro.json`** — layout metadata for ZMK Studio / keymap editor integration

### Key conventions

- Device tree uses ZMK idioms: `&kp`, `&mo`, `&bt`, `&rgb_ug`, `&sys_reset`, `&bootloader`
- Encoder bindings use `sensor-bindings` with `&inc_dec_kp` pairs
- RGB and battery config duplicated in both left and right `.dts` files (ZMK split requirement)
