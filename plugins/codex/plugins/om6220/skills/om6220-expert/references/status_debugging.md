# STATUS And Debugging Rules

## STATUS Interpretation
Do not interpret 6220/RW1602 `STATUS` bits as a universal nRF24-style truth
table. Always bind the meaning to the current state and firmware flow.

Use these reusable rules:

- `MAX_RT` dominates TX-side success checks. If `MAX_RT` is set, treat the
  operation as a retry/sync failure until the local code proves a different
  mode-specific meaning.
- In pairing and diagnostic states, RF-side success may appear as `TX_DS` or
  `RX_DR` as long as `MAX_RT` is clear. Do not require a pure `TX_DS` result
  for 6220/RW1602 pairing unless the project has measured that exact behavior.
- In normal fixed-slot TX work, a helper may return `TX_DS` simply to report
  that the transmit command was triggered. Treat that value as an application
  result, not a measured ACK or link-quality statistic.
- On the RX side, make packet validity the business truth: payload length,
  command inverse/checksum, remote id, sequence/channel index, and action
  freshness matter more than a single TX-side ACK bit.
- Clear only the status bits that the local driver expects, and re-check any
  IRQ-as-indicator state after status clear.

## Pairing Versus Normal Work
Separate these flows in code review and bug fixes:

- Pairing/bring-up/diagnostic: it is acceptable to read true `STATUS`, print or
  expose `TX_DS`/`RX_DR`/`MAX_RT`, and block briefly for an RF decision when the
  timing budget is explicit.
- Normal fixed-slot control: keep the 2 ms slot cadence. Send at most one
  packet, advance channel by schedule, and do not wait for ACK to decide the
  hop. Use receiver legal-packet count, action continuity, and visible link
  state to judge product behavior.
- ACK-control variants: if the whole protocol is intentionally non-hopping and
  ACK-driven, `TX_DS` without `MAX_RT` can be the connection condition. Do not
  copy that rule back into fixed-slot hopping.

## Three-Wire SPI Checks
Three-wire SPI direction mistakes often look like RF state-machine faults.
Before changing packet logic, capture CSN/SCK/SDIO and confirm:

- SDIO is released to input before register, `STATUS`, or payload reads.
- The `STATUS` byte appears at the expected clock edge and command boundary.
- Payload reads do not show the MCU still driving the shared data line.
- Repeated reads are stable across reset/init and channel changes.
- Wiring errors are ruled out before tuning waits or retry parameters.

## RX Fast Loop Rule
Keep a fast receiver loop in RX and poll `STATUS.RX_DR` or the project's
equivalent ready check. Avoid periodic receive-window rebuilding:

```text
CE low -> RX set -> CE high
```

Use RX reprepare only at explicit state boundaries:

- RF init or full recovery.
- Legal-packet handling after payload read and status/FIFO cleanup.
- Channel hop after packet or miss timeout.
- Sleep/wakeup restore.

Repeated reprepare inside the hot poll loop can break packet reception and ACK
timing, especially in 2 ms hopping protocols.

## Low-Power Triage
For unexpected current draw or failure to sleep, inspect product state before
adding RF cleanup code:

- Pairing versus normal state: an unpaired remote may intentionally keep sending
  probes.
- Idle timer sources: raw floating or not-yet-released keys should not refresh
  active state.
- TX duty cycle: idle heartbeat windows should be sparse; motion/demo paths may
  legitimately send every slot.
- Wake sources and LED modes: visible indicators can dominate observed current.
- RF state: confirm power-down and wake/re-init with waveform or current trace.

Do not use extra CE-low tails, forced re-init, or repeated status clears to hide
an incorrect application state machine.

## Small-ROM NY8 Constraints
Many NY8 6220/RW1602 projects are close to ROM and RAM limits. Prefer changes
that preserve compiled size and timing:

- Reuse existing flags, counters, buffers, and state machines.
- Add narrow condition fixes before adding new framework-style helpers.
- Avoid large lookup tables, debug strings, serial logging, deep call graphs,
  generic packet abstractions, and RAM-heavy queues.
- Preserve proven unrolled SPI in 2 ms paths unless the task is specifically ROM
  recovery and includes waveform comparison.
- Treat `.bin` size and generated map/list output as validation data, not skill
  content.
