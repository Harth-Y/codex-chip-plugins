# Protocol Timing Notes

## Fixed-Slot Hopping
Use this model for the 5-byte, 3-channel hopping car/remote protocol:

```text
slot period ~= 2 ms
channels = [87, 103, 91]
payload length = 5
one packet per slot
```

TX behavior:

- Send one packet in the current slot.
- Advance channel after the packet command sequence.
- Do not use ACK result as the upper-layer hop decision.
- Treat ACK as chip-level retry assistance only.
- In normal product mode, do not treat a helper's `TX_DS` return as measured
  link quality if that helper no longer waits for real `STATUS`.

RX behavior:

- Configure RX on the expected channel.
- On legal packet: read payload, validate counter/command/inverse/remote id,
  then hop immediately.
- On miss: wait about `(channel_count - 1) * slot_ms`; with 3 channels and 2 ms
  slots this is about 4 ms.
- Measure the timeout from the CSN low that starts configuring RX on the current
  channel to the next channel-write CSN low after timeout.
- Keep RX prepared during the fast poll loop. Do not repeatedly execute
  `CE low -> RX set -> CE high`; reserve RX reprepare for init, legal-packet
  handling, channel hop, or recovery.

## Pairing/Diagnostic Versus Normal Work
Use real `STATUS` reads only where the timing and state justify them:

- Pairing: probe at a low rate, read true `STATUS`, and accept `TX_DS` or
  `RX_DR` with no `MAX_RT` as RF-side success when validated for the project.
- Diagnostics: expose `TX_DS`/`RX_DR`/`MAX_RT` directly, but do not make the
  diagnostic statistics the product scheduler.
- Normal control: keep fixed-slot hopping and low-latency RX follow-hop. Judge
  product quality from receiver legal packets, action continuity, and link
  indicator behavior.

## Non-Hopping ACK-Control Variant
Some RW1602N2/RW1602N0P8 control tests use a simpler ACK protocol:

```text
address = 46 0B AF 43 98
channel = 45
payload length = 3
byte0 = 0x5A
byte1 = cmd: 0x00 stop, 0x01 forward, 0x02 reverse
byte2 = seq
```

Typical RF settings seen in the source material:

- `FEATURE=0x14`
- `EN_AA=0x01`
- `EN_RXADDR=0x01`
- `SETUP_RETR=0x25` or another deliberately chosen retry value
- `DYNPD=0x07` in transmitter tests
- `RF_SETUP=0x47` in the current test lineage

In this variant, `STATUS.TX_DS` with no `STATUS.MAX_RT` can be a connection
success condition. Do not copy that rule into fixed-slot hopping unless the user
is intentionally changing the protocol.

## Main Loop Timing Pattern
Recommended TX scheduler:

1. Timer ISR sets a `tx_slot_due` flag.
2. Main loop consumes the flag.
3. `run_tx_slot()` sends at most one packet and advances channel.
4. Every 5 slots, run slower key/business logic.

Avoid:

- SPI from timer ISR.
- Serial logging in the RF slot path.
- Long button scan/debounce work before sending a due slot.
- Waiting for ACK and polling `STATUS` inside every 2 ms hopping slot unless the
  slot budget has been measured.

## Receiver Watchdogs And Recovery
For simple command receivers:

- Use a motion watchdog so forward/reverse commands must be refreshed.
- Stop immediately on a valid stop command.
- Only reset RF silence timers from legal project packets, not from arbitrary RF
  events.
- Use light recovery before full `init_rf()` when the receiver is still mostly
  alive.
- Full RF re-init can interrupt reception; avoid running it too frequently in a
  lossy environment.

For motor-noise testing:

- Time-sliced motor OFF windows can help prove whether RF loss is coupled to
  motor noise.
- If using OFF windows, turn off motor outputs first, re-enter RX, then poll RF
  during the quiet window.

## Logic Analyzer Checklist
Capture at least:

- CSN
- SCK
- SDIO/MOSI
- IRQ when relevant
- Optional MCU debug GPIO marking timer slot or main-loop state

Check:

- One RF TX command sequence per 2 ms slot.
- No hidden 10 ms gap from business logic.
- `0xD9` appears where the tight path expects CE pulse.
- Channel-write cadence matches expected hop order.
- RX payload read and next-channel write happen immediately after legal packets.
- Timeout-to-hop interval is near the target miss window.
