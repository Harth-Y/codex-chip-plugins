# SDK Notes

## Scope
Use this reference for OM6626 SDK layout, build flow, toolchain assumptions, and
repository initialization.

## Known Baseline From SDK_OM6626_svn4181
- Target family: OM6626 / OM6XXX.
- Reference app observed: `projects/ble_app_simple`.
- Common target define: `CONFIG_OM6626A=1`.
- Reference Keil device: `OM662XX`, vendor `Onmicro`.
- CPU setting observed in Keil project and CMSIS header: Cortex-M4.
- Default Keil CPU clock in `ble_app_simple.uvprojx`: 64 MHz.
- FPU: not present.
- MPU: present.
- NVIC priority bits: 4.

## Repository Layout
- `projects/`: application examples and product projects.
- `projects/<app>/src`: preferred location for application changes.
- `projects/<app>/keil`: Keil project, output, binary and debug settings.
- `bsp/`: board selection, pinmux, GPIO and board initialization.
- `components/ble/`: BLE host/controller/API support.
- `components/pm/`: power manager and sleep integration.
- `components/nvds/`: nonvolatile data storage.
- `components/mbr/`: flash partition/security metadata.
- `hal/driver/`: SDK peripheral drivers.
- `hal/device/om6626/`: CMSIS device headers, startup/system code, ROM-lib
  config and linker scripts.
- `docs/html/`: generated SDK user guide and API reference.
- `tools/`: SDK tools and IDE support.

## Build Notes
- Main observed build flow is Keil uVision project files under
  `projects/<app>/keil`.
- Reference project: `projects/ble_app_simple/keil/ble_app_simple.uvprojx`.
- Reference scatter file:
  `hal/device/om6626/rom_lib/current/ARM/linker_flash.sct`.
- ROM library path:
  `hal/device/om6626/rom_lib/current/ARM`.
- Reference defines in `ble_app_simple`:
  - `CONFIG_OM6626A=1`
  - `CONFIG_LIB_PRESET_BLE_1PERIPHERAL`
  - `CONFIG_AUTOCONF_PRESET`
  - `_start=main`
- The ARM scatter file preprocessor line mentions `-mcpu=cortex-m33` while
  the device header and Keil project identify Cortex-M4. Treat this as a vendor
  SDK artifact; do not change architecture flags without confirming toolchain
  requirements.

## Project Initialization Template
When initializing a new repository, generate:

- `AGENTS.md`: stable OM6626 engineering rules.
- `PROJECT_NOTES.md`: active project facts and open fields.
- `README.md`: how this repository is organized, built and validated.

Recommended baseline sections for `PROJECT_NOTES.md`:

- Project summary.
- Target hardware.
- Board defaults and project pinout.
- Toolchain and build command.
- Memory/flash/NV layout.
- BLE role and services.
- 2.4GHz RF role and retry/hopping policy.
- Power/sleep target.
- Validation workflow.
- Known issues and project change policy.

Recommended `README.md` content:

- Project purpose.
- Directory map.
- Build entry point.
- Flash/programming method.
- Hardware setup.
- Validation checklist.
- Rules for generated outputs and protected regions.

## Change Policy
- Prefer application-level changes first.
- Avoid shared SDK edits unless the defect is proven there.
- Do not modify startup, linker, ROM-lib, MBR, EFUSE, PMU, BLE stack internals,
  or low-level RF code for convenience.
- If a shared SDK change is required, document affected projects and validation
  requirements.

## Validation
- Build the active Keil project after source changes.
- Inspect map/listing output after memory-sensitive changes.
- Use binary diff for low-level build/linker/startup changes.
- Record SDK version and project defines when reporting build results.
