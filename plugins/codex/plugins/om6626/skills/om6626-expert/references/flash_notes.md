# Flash Notes

## Scope
Use this reference for OM6626 flash, MBR, system info, CP/FT calibration, EFUSE,
NVDS, XIP, cache, boot, OTA and erase/write constraints.

## Memory Map Baseline
From SDK docs and linker files:

- ROM code: `[0x00100000, 0x00104000)`.
- ROM alias in current SDK: `[0x00000000, 0x00004000)`.
- RAM: `[0x20000000, 0x20014000)`.
- RAM alias: base `0x00200000` also points to RAM.
- Usable SRAM in current ROM-lib linker scripts: base `0x20000000`, size
  `0x00013ea0`, because some SRAM is reserved/occupied by ROM.
- ARM scatter stack size: `0x0800`.
- ARM scatter general heap size: `0x0000`.
- GCC linker stack size: `0x0800`.
- GCC linker general heap size: `0x0400`.

## Flash Map Baseline
- App XIP flash origin in current linker scripts: `0x00404000`.
- Internal SPI flash cacheable address range:
  `[0x00400000, 0x00800000)`.
- Internal/external SPI flash non-cacheable address range:
  `[0x50000000, 0x51000000)`.
- Flash layout from SDK docs:
  - MBR area: 0-12 KB.
  - System info area: 12-16 KB.
  - Application code area after system info.
  - Configuration/user data area depends on partitioning.
- MBR constants:
  - `OPT_SECTOR_START=0`
  - `MBR_SECTOR_START=1`
  - `CPFT_SECTOR_START=3`
  - `APP_SECTOR_START=4`

## EFUSE
- EFUSE size documented by SDK docs: 256 bits.
- User safety key: 128 bits at address 0.
- System CP/FT calibration data: 128 bits at address 16.
- EFUSE writes are high risk and may be irreversible. Never write EFUSE unless
  explicitly requested, reviewed and verified.

## Boot and Protection
- Bootloader can download app via UART/BLE, program flash, load app to SRAM, or
  execute app directly from flash.
- If secure boot is supported/enabled, normal bootloader assumptions may change.
- Application vector table magic controls run-from-flash behavior in SDK docs.
- Read/write protection can lock SWD. Do not lock SWD or set flash write
  protection unless the production policy is explicit and recovery is verified.

## XIP and Cache
- SDK docs state demos under `projects` use XIP mode.
- XIP can save RAM but introduces flash/cache timing considerations.
- Code that must run during flash erase/write, or code with strict timing, may
  need RAM placement. Confirm section names and linker behavior before moving
  code.
- Do not change linker scripts casually; validate binary/map output.

## Flash Operation Rules
- Flash must be erased before write.
- SDK docs explicitly state flash APIs are not called in IRQ context.
- Program/erase endurance is limited; do not use flash for high-frequency
  counters or logs without wear leveling or batching.
- Call `drv_sf_enable()` before flash operations; SDK docs state flash powers
  off automatically after a delay.
- Read back data after writes.
- For power-loss-sensitive data, design commit markers or double-buffering.

## NVDS and Config
- Treat NVDS/config storage as product data, not temporary scratch space.
- Record keys, value format, ownership and migration policy in
  `PROJECT_NOTES.md`.
- Validate default provisioning, upgrade migration and power-loss behavior.

## OTA
- OTA spans BLE, flash, boot and power loss.
- Validate image size, app partition, config partition, version policy and
  recovery path.
- Test success, interruption, invalid image and repeated update cases when OTA
  is product-critical.

## Protected Regions
Do not erase or overwrite unless explicitly requested and verified:

- MBR.
- System info.
- CP/FT calibration.
- EFUSE.
- Flash security/protection records.
- Existing production NVDS/config data.

## Validation
- Map/listing size check after code or linker changes.
- Flash readback after write/erase.
- Power-cycle after boot/app/OTA changes.
- Verify CPFT/system-info regions are untouched.
- Confirm app starts from expected flash address.
- Compare behavior with and without XIP-sensitive changes if timing changes.
