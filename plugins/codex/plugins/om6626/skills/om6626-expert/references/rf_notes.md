# 2.4GHz RF Notes

## Scope
Use this reference for OM6626 proprietary 2.4GHz RF work: packets, ACK/retry,
hopping, channel interference, CE timing, FIFO/DMA, IRQ ordering and RF bring-up.

## Capabilities
SDK documentation describes the proprietary RF protocol as supporting:

- 2.4GHz RF transceiver with GFSK / 2-FSK style operation.
- Air data rates from 25 kbps to 2 Mbps.
- Extended frequency range from 2358 MHz to 2512 MHz.
- CRC, whitening, timestamp, dual sync word detection.
- Auto ACK, auto retransmission, dynamic/static payload length.
- FIFO/DMA movement between system RAM and RF FIFOs.
- Packet-engine operation that reduces MCU timing burden but makes state
  correctness important.

## Frame Modes
- Frame A supports auto ACK and auto retransmission.
- Frame B supports auto ACK but not automatic retransmission; software must
  decide retry behavior.
- Frame A uses PID and CRC to identify retransmitted packets. Packet loss and
  ACK loss can look similar unless IRQ order and retry counters are inspected.

## Timing Rules
- `MAC_CE` / CE pulse timing matters. SDK docs note a minimum 10 us high pulse
  for TX and Standby-II transitions.
- For TX, the radio must have valid TX FIFO content before CE-triggered
  transmit.
- For RX, do not change sync word while receiving; turn RF off first.
- ACK payload timing constrains retransmit delay. A too-short ARD can break
  ACK payload operation at lower data rates.
- Treat delay-sensitive fixes as clues. If adding delay "fixes" behavior,
  investigate missed windows, stale state, IRQ clear timing, RF startup time,
  FIFO readiness, or role transition order.

## Channel, Rate and Hopping
- Prefer `om24g_set_rf_parameters()` when changing rate/frequency because SDK
  docs note it also configures frequency offset.
- When implementing hopping, record:
  - channel table,
  - hop trigger condition,
  - hop timing relative to TX/RX/ACK,
  - whether both peers switch from the same event,
  - resync policy after packet loss,
  - maximum dwell time and regulatory constraints if relevant.
- Do not change RF channel while a packet transaction is active unless the
  driver/API explicitly supports that state.
- Reset or interpret packet-loss counters carefully around channel changes.
- Use stable seed/state for reproducible hopping tests.

## Anti-Interference Checklist
- Verify channel occupancy with repeated packets, not one success.
- Compare static channel vs hopping behavior.
- Track TX complete, RX complete, timeout, CRC error and max retry events
  separately.
- Log current channel, next channel, retry count, packet sequence, and role.
- Capture at least one good transaction and one failed transaction with a logic
  analyzer or RF sniffer when possible.
- Confirm both peers agree on packet structure, address, sync word, CRC,
  whitening, payload length mode and data rate.

## IRQ and FIFO/DMA Risks
- Separate "packet received" from "payload consumed".
- Drain RX FIFO according to SDK API expectations.
- Do not assume a single IRQ means a single packet if the FIFO can hold more.
- Clear IRQ flags in the order expected by the driver; avoid reordering register
  writes without proof.
- Avoid large local RX/TX buffers on the stack.
- Treat DMA completion, RF completion and application event scheduling as
  separate state transitions.

## Minimal Debug Pattern
1. Disable hopping; prove fixed-channel TX/RX or ACK works.
2. Enable sequence numbers and log event order.
3. Add GPIO markers around CE high, IRQ entry, IRQ clear and channel switch.
4. Introduce hopping with one controlled trigger.
5. Test packet loss and ACK loss cases separately.
6. Validate across multiple channels and data rates.

## Safe Fix Pattern
- Prefer state-machine guards over arbitrary delays.
- If a delay is required by RF startup or CE timing, use SDK delay helpers and
  document the required timing.
- Keep retry/hopping policy in application code unless a driver bug is proven.
- Add validation notes for packet count, loss rate, retry count and recovery.
