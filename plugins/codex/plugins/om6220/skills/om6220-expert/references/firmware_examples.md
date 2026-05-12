# Firmware Example Guide

## Purpose
Use this guide when a repository contains the original 6220 application source
snapshot or a project derived from it. The installed plugin should not depend on
the snapshot being present; use these notes to know what each example teaches.

## Source Material Layout
The marketplace source snapshot was organized as:

```text
source-skills/6220 Application Reference/
  6220 application document
  docs/HS6220_RF/
  docs/RW1602N0P8_receiver/
  docs/RW1602N2_transmitter/
  firmware/car_ack_rx/
  firmware/remote_ack_hop_tx/
  rtc_examples/HS6220_RTC_PB1_input_PB2_toggle_stable/
  rtc_examples/HS6220_RTC_periodic_interrupt_test/
```

Names may be Chinese in the actual filesystem. Search by chip/module names such
as `HS6220`, `RW1602N0P8`, `RW1602N2`, `car_ack_rx`, `remote_ack_hop_tx`, or
`RTC`.

## Receiver Example: `firmware/car_ack_rx`
Use this example for:

- RX hopping.
- Binding a `remote id` after the first legal packet.
- Maintaining motor action state and automatic stop behavior.
- RF silence recovery and full `init_rf()` recovery.
- Reapplying IRQ/link indicator state after RF operations.
- Receiver-side SPI direction handling.
- Polling `STATUS.RX_DR` while IRQ is reserved for link or low-voltage
  indication.

High-risk areas:

- Main-loop delays that stretch the RX miss timeout.
- RF reset/init code that clears indicator output.
- Motion watchdog values that hide or expose packet-loss windows.
- Motor-noise mitigation that changes RF polling cadence.
- Repeated `CE low -> RX set -> CE high` inside the fast receive loop.

## Transmitter Example: `firmware/remote_ack_hop_tx`
Use this example for:

- 2 ms TX slot design.
- 5-byte payload generation.
- Rolling-byte-derived `remote id`.
- Channel sequence index handling.
- Expanded/high-speed SPI routines.
- `0xD9` CE pulse transmit flow.
- Key/business logic running slower than the RF slot.
- Pairing-state true `STATUS` checks versus normal-state fixed-schedule TX.

High-risk areas:

- Moving SPI work into an ISR.
- Adding `STATUS` polling after every packet.
- Replacing expanded SPI with loop-based SPI without waveform measurement.
- Reintroducing burst-send patterns that create RF gaps.
- Treating a normal-mode synthetic `TX_DS` return as real ACK quality.
- Adding large abstractions when ROM is close to the NY8 limit.

## RTC Examples
Use RTC examples for:

- RF RTC periodic interrupt setup.
- IRQ low detection through MCU input.
- PB2 toggle output for period validation.
- `STATUS_EXT` read/clear style.
- Next-target scheduling from the previous target tick.

Do not use RTC examples as proof that RTC can schedule the 2 ms hopping slot.
They prove low-frequency periodic wake/status behavior.

## Documentation Examples
Use `RW1602N0P8` documentation for:

- SIP module pin/function constraints.
- NY8 memory and IO limitations.
- HS6220 register descriptions.
- Common initialization sequences.
- Basic TX/RX examples.

Use `HS6220_RF` PDFs and regmaps only when local PDF access is available and the
task needs a datasheet-level detail not captured in this skill.

## Maintenance Rules
When updating this skill from source examples:

- Distill behavior into references; do not copy whole generated projects.
- Exclude `.bin`, `.elf`, `.o`, `.lst`, `.map`, `.s`, `.d`, `.htm`, and Keil
  intermediate files from the plugin unless the user explicitly asks for a
  release artifact.
- Preserve exact protocol constants and command sequences when they are part of
  a validated over-the-air behavior.
- Label notes that are project-specific versus generally applicable.
- Keep dates only when they explain version lineage or compatibility.
