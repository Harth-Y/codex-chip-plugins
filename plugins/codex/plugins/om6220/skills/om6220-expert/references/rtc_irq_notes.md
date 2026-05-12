# RTC And IRQ Notes

## IRQ Has Two Distinct Roles
The 6220-family IRQ pin can be used as:

- A real RF/RTC event output, usually active low.
- A software-maintained indicator output, for example link, low-voltage, or
  pairing state display in a constrained receiver design.

Do not write logic that assumes both meanings at the same time. Name helpers and
comments around the chosen role for the current project.

## IRQ Indicator Side Effects
When IRQ is repurposed as an indicator:

- RF reset can restore default IRQ behavior.
- `init_rf()` can overwrite DYNPD/FEATURE/CONFIG-related state.
- Clear status / clear IRQ operations can change the pin state.
- FIFO flush and mode switch flows can be adjacent to register writes that undo
  indicator configuration.
- Sleep/wakeup restore can leave the pin in a default event-output state.

After any of these flows, call the project's indicator-restore helper or
re-apply the DYNPD/output bits directly.

Keep RF event handling and indicator output separate. Receiver hot loops should
prefer polling `STATUS.RX_DR` for packets when IRQ is owned by link or low-voltage
display logic.

## DYNPD Direct Output Pattern
The source lineage uses DYNPD direct output for receiver indicator semantics:

```text
bit0 = pipe0 DPL when the RF protocol uses dynamic payload on pipe0
bit5 = 0
bit6 = 1
bit7 = output level
```

Use the local project's macro names when they exist. If not, isolate direct
literal writes behind a small helper such as `rf_apply_link_indicator()`.

## RTC Use Cases
RF RTC is appropriate for low-frequency, jitter-tolerant tasks:

- Low-power wakeup.
- Slow LED blinking.
- Low-frequency keepalive or status refresh.
- Periodic validation of IRQ event handling.

RF RTC is not a replacement for the 2 ms hopping slot. Keep 2 ms RF scheduling
on the MCU timer/main-loop path and verify it with a logic analyzer.

## Verified RTC Test Pattern
Known-good RTC validation pattern from source material:

- Connect MCU `PB1` to HS6220 `IRQ`.
- Use MCU `PB2` as a test output.
- Configure RTC for about 500 ms or 1 s periods.
- Wait for IRQ low in the main loop.
- Read and clear the RTC-related extended status, for example `STATUS_EXT`.
- Toggle PB2 after each handled event.
- Load the next target tick.

For stable long-term periods, compute:

```text
next_target = previous_target + period_ticks
```

Do not repeatedly use:

```text
next_target = current_time + period_ticks
```

The previous-target method avoids accumulating main-loop handling delay into the
period. Small jitter from tick quantization, polling latency, and measurement
resolution is normal.

## RTC Register Window Discipline
Some source examples temporarily open a BLE/RTC-related register window through
`FEATURE`, perform RTC/status operations, then restore the prior `FEATURE`
value.

When editing that flow:

- Save the original `FEATURE` value before enabling the window.
- Restore it after the RTC operation.
- Keep the critical window short.
- Avoid mixing normal RF TX/RX operations into the window.
- Re-check IRQ indicator state after the operation if IRQ is also an indicator.

## Validation
For RTC or IRQ changes, verify on hardware:

- IRQ low event appears at the expected period.
- The MCU detects the event without busy loops that block RF-critical work.
- Status clear releases/re-arms the next interrupt.
- Indicator output still has the expected level after RF reset/init/status clear.
- Sleep entry disables or preserves RF/RTC state according to the intended
  design.
- Wakeup re-initializes RF and reapplies indicator state.
