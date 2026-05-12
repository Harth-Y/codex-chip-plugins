# 6220 / RW1602 Application Notes

## Scope
Use these notes for OM6220 / HS6220-class 2.4GHz RF applications and RW1602
SIP modules, especially NY8 + RW1602N0P8 receiver and RW1602N2 transmitter
projects.

## Current Hopping Protocol Baseline
Common car/remote projects use:

- Address: `46 0B AF 43 98`
- Channels: `87, 103, 91`
- Channel order: `87 -> 103 -> 91 -> 87`
- Payload length: 5 bytes

Packet format:

```text
byte0 = packet counter
byte1 = command: 0x00 stop, 0x01 forward, 0x02 reverse
byte2 = command inverse: byte1 ^ 0xff
byte3 = remote id
byte4 = hopping sequence index: 0, 1, 2
```

Remote ID is derived from rolling bytes:

```text
0x7D0 = rolling byte0
0x7D1 = rolling byte1
0x7D2 = rolling byte2
byte3 = byte0 ^ byte1 ^ byte2
```

Receiver behavior:

- Power up in pairing/wait state.
- Accept the first legal packet and bind that packet's `remote id`.
- After binding, reject packets from other `remote id` values.

## STATUS Semantics
`STATUS` bits must be interpreted by state:

- Pairing or diagnostic flows may read true `STATUS` and decide RF-side success
  from `TX_DS` or `RX_DR` when `MAX_RT` is clear.
- Normal fixed-slot TX flows may deliberately skip true ACK waiting. A returned
  `TX_DS` can mean only that TX was triggered.
- RX business success should be based on legal packets and freshness, not on a
  TX-side ACK statistic.
- Do not mechanically copy nRF24 assumptions; 6220/RW1602 projects can expose
  valid RF success through `RX_DR` during pairing.

## Timing Rules
Transmitter:

- Run one TX slot about every 2 ms.
- Send at most one packet per slot.
- Keep each packet on one channel only.
- After sending, advance directly to the next channel.
- Keep ACK enabled when the project uses hardware retry, but do not wait for
  true ACK success before hopping in the fixed-slot hopping design.

Receiver:

- Start from a fixed channel after power-up.
- After a legal packet, hop to the next channel immediately.
- On a miss, stay on the current channel until timeout.
- For 3 channels and 2 ms slots, treat miss recovery as about 4 ms from entering
  RX on that channel to hopping to the next channel.

## Main Loop Model
For tight 2 ms TX slots:

- Let Timer0 or an equivalent timer set a slot flag.
- Keep the ISR short; do not send RF packets from the ISR.
- In the main loop, run exactly one TX slot when the flag is set.
- Run slower business logic every N slots, for example every 5 slots for a
  roughly 10 ms control update.

Avoid "delay 10 ms, then send several packets rapidly" patterns. They create
bursty traffic and leave long RF gaps.

## SPI Driver Tradeoff
NY8 ROM/RAM is tight, but SPI time is often tighter than ROM in 2 ms protocols.

- Expanded/unrolled SPI costs more ROM but leaves more slot margin.
- Loop-based SPI saves ROM but stretches waveforms.
- Do not switch back to loop-based SPI to save a few words unless the task is
  explicitly ROM optimization and includes waveform comparison.

Three-wire SPI requires direction discipline:

- After writing MOSI/SDIO, return the line to input/pull-up before reads.
- Before reading payload, `STATUS`, or registers, verify SDIO direction.
- Confirm CSN/SCK/SDIO with a logic analyzer, not source inspection alone.
- Treat wiring or direction mistakes as likely when RF init seems correct but
  `STATUS`, payload, or indicator behavior is impossible.

Preferred tight TX flow:

```text
0x8B reset
clear status
set channel
flush tx/rx
write tx payload
0xD9 CE pulse
```

`0xD9` is the CE pulse command. In the 2 ms path, avoid immediately reading
`STATUS` for statistics or repeating clear/flush work after `0xD9` unless the
project has explicit timing margin for it.

## Debugging Priority
Use waveform evidence first:

- CSN spacing.
- SCK continuity and unexpected gaps.
- SDIO/MOSI direction switching.
- Interval from one `0x8B` reset to the next.
- Whether `0xD9` is used instead of older `0xD5`/`0xD6` CE high/low control.
- RX hop timing after a valid packet.
- RX timeout timing after a miss.

If waveform and code disagree, first inspect software delays, serial logs, main
loop blocking, timer reload/divider, RF reset/init flow, and SPI direction code.

## Small NY8 Change Budget
When ROM is near the 2K-class limit:

- Reuse existing project state, counters, and packet buffers.
- Prefer local condition fixes over new abstractions.
- Avoid debug strings, generic packet layers, large tables, and deep helper
  chains.
- Check the generated image/map size when the toolchain is available.
