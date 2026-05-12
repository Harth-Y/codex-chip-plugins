# HS6220 / RW1602 Register Quick Reference

## RF Basics
- ISM band: 2.400 to 2.483 GHz for normal 2.4GHz use.
- Frequency rule: `F0 = 2400 + RF_CH MHz`.
- Modulation: GFSK.
- Common rates: 1 Mbps and 2 Mbps.
- Payload: normally 1 to 32 bytes unless long-packet mode is enabled.

## Key Bank0 Registers
| Addr | Name | Use |
| --- | --- | --- |
| `0x00` | `CONFIG` | RX/TX mode, power-up, CE by register, CRC, IRQ masks |
| `0x01` | `EN_AA` | Auto ACK enable per pipe |
| `0x02` | `EN_RXADDR` | RX pipe enable and scramble-related control |
| `0x03` | `PMU_CTL` | RF power state and 32K RTC enable |
| `0x04` | `SETUP_RETR` | Auto retry delay/count |
| `0x05` | `RF_CH` | RF channel |
| `0x06` | `RF_SETUP` | Data rate, PA power, calibration flags |
| `0x07` | `STATUS` | IRQ flags, bank bit, CE state, pipe number |
| `0x0A` | `RX_ADDR_P0` | Pipe0 RX address |
| `0x10` | `TX_ADDR` | TX address |
| `0x11` | `RX_PW_P0` | Pipe0 payload width |
| `0x16` | `STATUS_EXT` | Extended status, used by RTC flows in source examples |
| `0x17` | `FIFO_STATUS` | TX/RX FIFO state |
| `0x18` | `CONFIG_EXT` | Extended config, including FIFO interrupt masks |
| `0x1C` | `DYNPD` | Dynamic payload and IRQ output-control related bits |
| `0x1D` | `FEATURE` | Dynamic payload, ACK payload, BLE/RTC window, soft reset |

## Common CONFIG Values
- `0x08`: power-down-style baseline in many examples.
- `0x0A`: TX mode, `PWR_UP=1`, `PRIM_RX=0`.
- `0x0B`: RX mode, `PWR_UP=1`, `PRIM_RX=1`.

Always confirm local bit definitions because vendor examples may combine project
specific masks, IRQ masks, or CE register control.

## STATUS Bits
| Bit | Mask | Meaning |
| --- | --- | --- |
| 7 | `0x80` | Bank indicator / bank switch control in common examples |
| 6 | `0x40` | `RX_DR`, RX data ready, write 1 to clear |
| 5 | `0x20` | `TX_DS`, TX complete, write 1 to clear |
| 4 | `0x10` | `MAX_RT` or sync/max-retry style event depending on mode |
| 3 | `0x08` | CE state in source definitions |
| 0 | `0x01` | TX full |

Clearing status can disturb IRQ pin behavior if the project repurposes IRQ as an
indicator. Re-apply indicator output state after status/IRQ clear when needed.

Interpretation rules:

- Pairing/diagnostic code may treat `TX_DS` or `RX_DR` with no `MAX_RT` as a
  measured RF-side success when validated on the target 6220/RW1602 firmware.
- Normal fixed-slot TX code may return `TX_DS` without reading real `STATUS`.
  Treat that as "TX triggered" unless the function explicitly waits for status.
- `MAX_RT` should block success decisions in ACK/retry flows.
- RX-side application success should require a legal payload, not just `RX_DR`.

## DYNPD And IRQ Output Use
Source material uses `DYNPD` bits to direct-control the IRQ pin as a visible
connection or low-voltage indicator in receiver firmware:

- `bit0=1`: pipe0 dynamic payload length in common ACK/DPL setups.
- `bit5=0`: do not bypass the relevant IO behavior.
- `bit6=1`: enable register-controlled output.
- `bit7=0/1`: drive output low/high in that mode.

Do not assume this is active after RF reset, soft reset, `init_rf()`, sleep
restore, clear status, or FIFO flush. Re-apply the indicator state after any RF
flow that can restore defaults.

Because `DYNPD` can carry both DPL and direct IRQ-output semantics, never rewrite
it as a plain dynamic-payload register in projects that use IRQ as LED/link
output. Preserve both payload and output bits.

## FEATURE Notes
Common bits seen in source definitions:

- `EN_DYN_ACK`: dynamic/no-ack command support.
- `EN_ACK_PAY`: ACK payload support.
- `EN_DPL`: dynamic payload length.
- `BLE_EN`: opens BLE/RTC related register behavior in some example code.
- `SOFT_RST`: soft reset.
- `LONG_PACKET_EN`: long packet mode.

RTC helper code may temporarily enable the BLE/RTC window through `FEATURE`,
perform extended-register operations, then restore the previous `FEATURE` value.

## SPI Commands
| Command | Value | Use |
| --- | --- | --- |
| `R_REGISTER` | `0x00-0x1F` | Read register |
| `W_REGISTER` | `0x20-0x3F` | Write register |
| `R_RX_PAYLOAD` | `0x61` | Read RX payload |
| `W_TX_PAYLOAD` | `0xA0` | Write TX payload |
| `FLUSH_TX` | `0xE1` | Flush TX FIFO |
| `FLUSH_RX` | `0xE2` | Flush RX FIFO |
| `REUSE_TX_PL` | `0xE3` | Reuse TX payload |
| `R_RX_PL_WID` | `0x60` | Read dynamic payload width |
| `W_ACK_PAYLOAD` | `0xA8-0xAF` | Write ACK payload for pipe |
| `W_TX_PAYLOAD_NOACK` | `0xB0` | Write payload without ACK |
| `NOP` | `0xFF` | No operation / status read |
| `CE pulse` | `0xD9` | Tight-path pulse used by current 2 ms TX flow |

## Power And Timing Values
- Power down to standby is commonly treated as about 2 ms.
- Standby to TX/RX active is commonly treated as about 130 us.
- CE high minimum for older explicit CE control is about 20 us, but current tight
  TX flow prefers `0xD9` CE pulse where supported by the driver.

## Bank1 Caution
Bank1 has PLL, calibration, RX control, frequency deviation, AGC, and test
registers. Treat Bank1 writes as high risk:

- Preserve existing initialization sequences unless the source material or chip
  document explains the change.
- If changing calibration or AGC values, record the old value, the intended RF
  effect, and the bench validation.
- Always return to the expected bank before normal Bank0 operations.
