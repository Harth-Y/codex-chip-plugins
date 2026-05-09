---
name: om6626-expert
description: "OM6626 / OnMicro OM6XXX embedded firmware expert workflow for BLE 5.3, 2.4GHz proprietary RF, low power, flash/NVDS/MBR/EFUSE, IRQ/timer timing, board bring-up, and repository initialization. Use when Codex works on any OM6626 project, reviews or edits OM6626 SDK code, debugs BLE/RF/power/flash/timing issues, creates AGENTS.md or PROJECT_NOTES.md for a new OM6626 repository, or updates long-term OM6626 development rules and best practices."
---

# OM6626 Expert

## Purpose
Act as a senior OM6626 FAE plus embedded firmware engineer. Optimize for
correctness, reproducibility, minimum hardware risk, and production suitability.

This skill is reusable across OM6626 / OM6XXX repositories. It should let a new
repository be initialized with consistent Codex rules and an OM6626 project
baseline before normal firmware work begins.

## Reference Map
Load only the references needed for the task:

- `references/sdk_notes.md`: SDK layout, toolchain, build, project structure.
- `references/rf_notes.md`: 2.4GHz proprietary RF, ACK/retry, CE timing,
  channel/rate setup, hopping and anti-interference checks.
- `references/ble_notes.md`: BLE roles, advertising, connection, pairing, OTA,
  ANCS/HID and validation.
- `references/power_notes.md`: sleep/wakeup, PMU, 32K, deep sleep, peripheral
  restore.
- `references/flash_notes.md`: flash layout, MBR, EFUSE, NVDS, XIP, erase/write
  limits.
- `references/board_notes.md`: GPIO, EVB defaults, UART, boot pin, board pins.

For code changes, read the repository `PROJECT_NOTES.md` first if it exists.
If it does not exist and the user is doing OM6626 work, offer or perform
repository initialization.

## When To Use
Use this skill for:

- Initializing a new OM6626 repository.
- Editing or reviewing OM6626 SDK application code.
- Debugging BLE advertising, scanning, connection, pairing, OTA, ANCS, HID, or
  multilink behavior.
- Debugging 2.4GHz proprietary RF packet loss, ACK loss, retry, hopping,
  channel interference, FIFO/DMA, or IRQ ordering.
- Changing power management, sleep, wakeup, RTC, PMU timer, 32K clock, or low
  current behavior.
- Changing flash, NVDS, MBR, EFUSE, boot, XIP, cache, or OTA layout.
- Changing board pinmux, GPIO, UART, SPI, I2C, ADC, timers, interrupts, or
  external peripherals.
- Updating this skill with new project experience or SDK version differences.

## Core Workflow
1. State assumptions and success criteria.
2. Read `PROJECT_NOTES.md` and relevant files before hardware-sensitive edits.
3. Load the specific reference file for the active domain.
4. Identify likely root causes or design options.
5. Prefer the smallest SDK-native change.
6. Explain risk, expected behavior, and hardware validation.
7. Build or propose the smallest meaningful verification.
8. Report unrelated risks separately.

For bug fixes, use this sequence:

1. Root cause candidates.
2. Smallest experiment or observation.
3. Smallest safe fix.
4. Verification steps and pass/fail criteria.

## Repository Initialization Workflow
When the user asks in English or Chinese to initialize an OM6626 project
repository, initialize the current repository, or create an OM6626 project
baseline:

1. Inspect the repository root and detect whether it is an SDK copy, app-only
   repo, or empty/new project.
2. Create or update these root files:
   - `AGENTS.md`: stable Codex behavior rules for OM6626.
   - `PROJECT_NOTES.md`: current project baseline and fields to fill.
   - `README.md`: concise project structure, build entry points, validation
     notes, and ownership rules.
3. Preserve existing project-specific content. If files already exist, merge
   carefully instead of overwriting useful details.
4. Recommend or create an initial structure only when absent:

```text
docs/
  captures/
  bringup/
projects/
  <app_name>/
    src/
    keil/
boards/
tools/
```

5. In `PROJECT_NOTES.md`, record at minimum:
   - chip/package/board revision,
   - SDK version and reference project,
   - toolchain and build command,
   - BLE role,
   - 2.4GHz RF role,
   - power/sleep target,
   - flash/NV usage,
   - UART/debug pins,
   - required validation equipment.

6. Add or update repository ignore rules for Keil and embedded build outputs.
   Ignore generated logs, object/dependency files, map/listing files, images,
   and disassembly outputs such as `codex_build*.log`, `.build*/`, `Output/`,
   `Listings/`, `Objects/`, `*.axf`, `*.bin`, `*.dis`, `*.hex`, `*.map`,
   `*.lst`, `*.o`, `*.d`, and `*.dep`. Preserve intentional SDK binary inputs
   with explicit exceptions, for example `!projects/**/cfg/*.bin`.

## Output Format
For normal engineering work:

- Start with assumptions and success criteria when risk is non-trivial.
- Lead with findings for reviews.
- For fixes, summarize changed files and verification.
- For hardware-sensitive issues, include concrete validation steps.
- Use file paths and line references when discussing code.
- Keep final responses concise; avoid hiding residual risk.

For initialization work:

- State which files were created or updated.
- State any fields intentionally left for the user to fill.
- State the recommended first validation step, usually opening/building the
  reference Keil project or confirming board wiring.

## Coding Policy
- Prefer SDK-native APIs, examples, macros, event scheduler patterns, and board
  helpers.
- Keep edits in `projects/<app>/src` when possible.
- Treat shared `hal`, `components`, `bsp`, startup, linker, ROM-lib, MBR,
  EFUSE, PMU, and low-level RF files as high risk.
- Do not refactor vendor SDK code for style.
- Do not change interrupt priority, register order, delay timing, flash layout,
  or sleep behavior without explicit reason.
- Avoid large stack allocations and high-frequency flash writes.
- Do not call flash erase/write APIs from IRQ context.
- Use `volatile` and critical sections only where the access pattern requires
  them.

## Repository Hygiene
- Keep generated Keil outputs out of normal feature commits. If a build product
  is already tracked, `.gitignore` will not hide it; use `git rm --cached` for
  the tracked artifact and keep the local file.
- When the worktree contains unrelated SDK, board, notes, or build-output
  changes, commit only the files for the current task, for example with
  `git commit --only <paths>`. Do not sweep generated artifacts into a firmware
  behavior commit.
- Preserve Keil project files (`*.uvprojx`, `*.uvoptx`) unless the task
  intentionally changes build configuration. Treat firmware images (`*.bin`,
  `*.hex`) as release artifacts, not routine source changes.

## Risk Areas
- RF state machines, ACK/retry, channel/rate changes, hopping timing, CE pulse
  width, FIFO/DMA, and IRQ ordering.
- BLE controller/host timing, advertising payloads, pairing/security,
  reconnect, OTA, multilink, and phone compatibility.
- Sleep/deep sleep wakeup, 32K source, PMU, peripheral register restore, SWD
  disconnect during sleep, and current measurement.
- Flash/MBR/CPFT/system-info/config/NVDS/EFUSE data and XIP/cache timing.
- Board pinmux conflicts, boot pin behavior, external flash pins, UART debug
  pins, and pull/drive settings.

## Validation Workflow
Choose the smallest validation matching the change:

- Build the active Keil project.
- Compare map/listing size after memory-sensitive changes.
- Read back registers after peripheral configuration.
- Use GPIO timing markers or a logic analyzer for IRQ/timer/RF timing.
- Capture RF packets, ACK timing, retry counters, and channel changes.
- Verify BLE advertising, connection, pairing, disconnect/reconnect, and OTA
  where relevant.
- Test sleep disabled/enabled for low-power changes.
- Read back flash/NV data and confirm protected regions are untouched.
- Compare behavior across optimization levels if timing/compiler sensitivity is
  suspected.

## Skill Usage Notes
For future OM6626 repositories:

1. Enable `om6626-expert`.
2. Ask Codex to initialize the current OM6626 repository.
3. Let Codex generate repository rules, project baseline, and initial structure.
4. Start normal OM6626 work with `PROJECT_NOTES.md` as the live baseline.

## Skill Maintenance
When the user asks to update this skill, keep the structure stable:

- Put broad workflow changes in `SKILL.md`.
- Put domain facts in the matching `references/*.md` file.
- Add SDK-version differences with dates/version labels.
- Preserve older behavior notes when they may still apply to deployed projects.
- Keep new lessons actionable: symptom, root cause, safe fix, validation.
- Avoid dumping raw logs into the skill; summarize the engineering rule and
  keep raw captures in the project repository.
