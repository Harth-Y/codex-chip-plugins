# BLE Notes

## Scope
Use this reference for OM6626 BLE work: roles, advertising, scanning,
connections, pairing/security, OTA, ANCS, HID, multilink and BLE validation.

## Baseline
- SDK user guide identifies the platform as Bluetooth Low Energy 5.3.
- Reference project: `projects/ble_app_simple`.
- `ble_app_simple` initializes board, BLE stack, advertising and security from
  `projects/ble_app_simple/src/main`.
- Common reference define:
  `CONFIG_LIB_PRESET_BLE_1PERIPHERAL`.
- Current RTE priority notes:
  - BLE IRQ priority: 2.
  - BLE wakeup IRQ priority: 1.

## Role Checklist
Record the active BLE role in `PROJECT_NOTES.md`:

- Peripheral advertising only.
- Peripheral connectable.
- Central scanning/connecting.
- Multilink.
- Mesh.
- OTA/DFU.
- HID, ANCS, DIS/BAS, custom GATT service or vendor profile.

Do not mix role assumptions. A setting valid for a single peripheral demo may
not be valid for central, scan, multilink or OTA projects.

## Advertising
- Verify advertising type, interval, channel map and connectability.
- Check advertising payload and scan response length before adding data.
- Keep device name, appearance, service UUIDs and manufacturer data consistent
  with product requirements.
- Treat iOS Settings Bluetooth as a filtered pairing UI, not a general BLE
  scanner. If nRF Connect or another scanner sees the advertiser, RF and ADV
  are likely working even when iOS Settings hides the device.
- For better iOS Settings visibility, advertise real standard profiles such as
  HID `0x1812` only when the matching GATT service is actually registered.
  Do not advertise a standard service UUID just to pass phone filtering; iOS
  may show the device but then fail service discovery, pairing, or reconnect.
- Validate with a phone scanner and at least one BLE test tool when available.
- For low power, advertising interval and sleep policy must be considered
  together.

## Connection and GATT
- Validate connection interval, latency, supervision timeout and MTU against
  product needs.
- Do not assume a GATT write/read callback means data is semantically accepted;
  validate application state and response behavior.
- Keep notification/indication flow control explicit.
- For reconnect issues, log disconnect reason, peer address type, bond state and
  security state.

## Pairing and Security
- Treat pairing, bonding, privacy and address type as one state machine.
- Confirm whether the product requires bonding, MITM, passkey, fixed passkey,
  whitelist, privacy or unauthenticated pairing.
- Validate first pairing, reconnect after reset, bond deletion and pairing
  failure recovery.
- Release notes mention fixes related to pairing edge cases; do not assume
  older SDK behavior when debugging security.

## OTA / DFU
- OTA touches BLE, flash, boot and power-loss behavior. Load
  `flash_notes.md` as well.
- Validate image size, partition, version policy, power-loss behavior and
  post-upgrade boot.
- Avoid flash writes from IRQ context.
- Test failed upload, interrupted upload and repeated upgrade/downgrade policy
  if product requirements include them.

## ANCS / HID / Profile Work
- Keep profile-specific state machines separate from BLE connection state.
- For HID, validate report map, report IDs, reconnect behavior and host OS
  compatibility.
- For ANCS, validate bonding/security and phone notification permission flow.
- For battery/DIS/custom GATT, verify UUIDs, properties, permissions and CCCD
  persistence.

## Multilink and Scanning
- Multilink and scan are timing-sensitive. Monitor ACL buffers, role switching,
  scan parameters and connection parameter updates.
- Release notes mention fixes for scan edge cases, multilink stability and
  Coded PHY parameter issues. Record SDK version when debugging these areas.

## Validation
- Advertising visible and payload correct.
- Connection stable across repeated connect/disconnect.
- Pairing and bonding behavior matches product policy.
- GATT read/write/notify/indicate paths verified.
- OTA verified with successful and interrupted update if used.
- Sleep enabled/disabled comparison for wakeup-sensitive BLE issues.
- Phone compatibility checked on the target phone families when relevant.
