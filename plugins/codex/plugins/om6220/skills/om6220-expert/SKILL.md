---
name: om6220-expert
description: "OM6220 / HS6220 / RW1602 embedded firmware expert workflow for 2.4GHz GFSK RF, ACK/retry, 2 ms hopping protocols, RW1602N0P8 receiver work, RW1602N2 transmitter work, NY8 SPI drivers, IRQ pin behavior, RF RTC, low-power wakeup, and logic-analyzer-based debugging. Use when Codex works on OM6220, HS6220, RW1602N0P8, RW1602N2, or 6220-family RF firmware, reviews or edits car/remote control code, debugs packet loss, ACK timeout, hopping, SPI timing, IRQ output, RTC interrupt, sleep/wakeup, or updates this skill from 6220 application source material."
---

# OM6220 Expert

## Purpose
Act as a senior 6220-family RF firmware engineer. Optimize for measured RF
behavior, tight timing, minimum ROM/RAM impact, and hardware-safe changes.

This skill covers OM6220 / HS6220-class 2.4GHz RF usage and RW1602 SIP
applications, especially NY8 + RW1602N0P8 receiver and RW1602N2 transmitter
projects.

## Reference Map
Load only the references needed for the task:

- `references/application_notes.md`: distilled 6220/RW1602 application rules,
  pairing packet format, hopping protocol, SPI/IRQ/RTC lessons.
- `references/protocol_timing.md`: 2 ms TX slot, RX follow-hop timing,
  ACK usage, watchdog/recovery behavior, and waveform checks.
- `references/status_debugging.md`: reusable `STATUS` interpretation rules,
  pairing versus normal-mode ACK semantics, low-power triage, and small-ROM
  NY8 change constraints.
- `references/register_quickref.md`: key HS6220/RW1602 registers, commands,
  common values, and status bits.
- `references/rtc_irq_notes.md`: RF RTC, IRQ as event pin versus indicator pin,
  and reset/re-init side effects.
- `references/firmware_examples.md`: what to inspect in the source examples,
  which example teaches which behavior, and maintenance rules.

For code changes, read the repository `PROJECT_NOTES.md` first if it exists.
If the repository contains a local `source-skills/6220 Application Reference`
snapshot and the user is updating this skill, treat that snapshot as source
material and distill it into references instead of publishing raw build output.

## Core Workflow
1. State assumptions, target chip/module, RF role, and success criteria.
2. Read project notes and the smallest relevant code path before changing code.
3. Load the matching reference file from this skill.
4. Classify the RF state first: pairing, diagnostic, normal TX, normal RX,
   recovery, or sleep/wakeup.
5. Identify timing-sensitive paths and any RF reset/init/status-clear side
   effects.
6. Prefer the smallest existing-code-pattern change.
7. Explain expected waveform/register behavior and residual hardware risk.
8. Build if a local build command exists; otherwise give concrete bench checks.
9. Report unrelated generated files, binaries, or Keil outputs separately.

For bugs, use this sequence:

1. Root cause candidates.
2. Smallest observation or logic-analyzer capture to separate them.
3. Smallest safe fix.
4. Pass/fail validation criteria.

## RF Hopping Rules
When working on the current 3-channel car/remote style protocol:

- Preserve the 5-byte packet shape unless the user intentionally changes the
  over-the-air protocol.
- Treat TX hopping as fixed-time: one packet per 2 ms slot, then advance to the
  next channel without waiting for a real ACK decision.
- Keep hardware ACK enabled when the project relies on chip-level retry, but do
  not use ACK success/failure as the upper-layer hop trigger unless the project
  is explicitly the non-hopping ACK-control variant.
- In pairing or diagnostic states, read real `STATUS` and treat `TX_DS` or
  `RX_DR` with no `MAX_RT` as RF-side success when the local project has this
  observed 6220/RW1602 behavior. In normal fixed-slot work, a returned `TX_DS`
  may only mean "TX was triggered"; do not use it as link-quality proof.
- Treat RX hopping as follow-hop: after a legal packet, hop immediately; after a
  miss, timeout after about `(channel_count - 1) * slot_ms`.
- Keep the RX fast loop in receive mode. Do not periodically run
  `CE low -> RX set -> CE high`; only reprepare RX at clear state boundaries
  such as init, legal-packet handling, channel hop, or recovery.
- Validate RF timing with a logic analyzer on CSN/SCK/SDIO/IRQ. If source code
  and waveform disagree, trust the waveform first.

## Coding Policy
- Preserve expanded/high-speed SPI routines in 2 ms slot projects unless the
  task is explicitly ROM optimization with waveform comparison.
- Keep ISR work minimal: set flags in timers or GPIO IRQs; do RF SPI work from
  the main loop unless the existing project proves another pattern.
- Do not add serial logging, long delays, repeated status reads, or repeated
  flush/clear operations inside the 2 ms RF path without a timing budget.
- Prefer the current project pin macros, delay functions, RF wrappers, and NY8
  register naming. Do not rewrite vendor-style drivers for style.
- Treat three-wire SPI direction switching as a first-order failure source.
  Any strange `STATUS` or payload read must be checked on CSN/SCK/SDIO before
  changing RF logic.
- Treat RF reset, `init_rf()`, status clear, FIFO flush, mode switch, sleep
  restore, and RTC clear as operations that may disturb IRQ output semantics.
- Do not mix "IRQ is an RF event input" and "IRQ is a software-maintained LED or
  link indicator" assumptions in one code path.
- For low-power bugs, inspect state machine transitions, idle refresh sources,
  and TX duty cycle before adding RF CE cleanup or extra send-stop code.
- For small-ROM NY8 projects, favor local fixes in existing state machines,
  reuse current buffers and flags, and avoid large tables, deep call chains,
  generic abstractions, and extra RAM.
- Keep firmware images and Keil output as release/build artifacts, not normal
  source edits.

## Validation Workflow
Choose the smallest validation matching the change:

- Build the active NY8/Keil project when the toolchain is available.
- Capture CSN/SCK/SDIO timing around RF reset, channel write, payload write,
  `0xD9` CE pulse, RX payload read, and status clear.
- Confirm TX slot spacing is near 2 ms and each slot sends at most one packet.
- Confirm RX hops immediately after legal packets and times out near the
  intended miss window.
- Check `STATUS`, `FIFO_STATUS`, `DYNPD`, `FEATURE`, and RTC-related extended
  status only where they explain the symptom. Interpret `RX_DR`, `TX_DS`, and
  `MAX_RT` according to pairing/diagnostic versus normal-work state.
- For IRQ indicator work, verify the visible output after every RF reset/init
  and after status clear.
- For RTC work, verify IRQ low events and output toggles over multiple periods;
  schedule the next target from the previous target time to avoid drift.
- For sleep/wakeup work, verify RF power-down, wake source, IO restore,
  `init_rf()`, and link-state recovery on hardware.

## Skill Maintenance
When updating this skill:

- Put broad workflow and trigger behavior in `SKILL.md`.
- Put detailed domain facts in the matching `references/*.md`.
- Summarize source examples as reusable rules; do not paste large raw logs,
  generated `.lst/.map/.o/.elf/.bin` files, or one-off debug history.
- Keep source-material paths only as maintenance hints, not as required runtime
  dependencies for installed users.
