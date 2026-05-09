# Power Notes

## Scope
Use this reference for OM6626 sleep, wakeup, PMU, RTC, PMU timer, 32K clock,
deep sleep, ultra deep sleep and low-current debugging.

## Power Modes
SDK documentation describes five power modes:

- Active: CPU and modules active.
- Idle: CPU clock gated; all interrupts can wake the system.
- Sleep: GPIO, PMU_TIMER, RTC and BLE can wake the system.
- Deep sleep: GPIO and RTC can wake if 32K is running; BLE stack does not work.
- Ultra deep sleep: similar wake sources, but SRAM is powered off and wake
  triggers reboot.

## Default Board Behavior
- EVB board init enables DCDC mode.
- EVB selects RC32K by default: `PMU_32K_SEL_RC`.
- Default pin drive current is normal.
- Board init pulls up all IOs, except IO24/IO25 when using a 32.768 kHz crystal.

## Sleep Entry Rules
- Sleep is managed by the event scheduler and power manager.
- BLE stack status affects whether sleep is allowed.
- Drivers may prevent sleep while peripherals are busy.
- Some receive flows, especially UART long packets, require application-level
  sleep prevention because the driver may not know packet boundaries.
- Use SDK power-management APIs rather than ad hoc global flags when possible.

## Wakeup and Restore Risks
SDK docs state that after sleep wakeup, several peripheral registers may need
reconfiguration:

- GPIO.
- Timer.
- UART.
- SPI.
- DMA.
- ADC.
- I2C.

Pinmux/GPIO may be restored by hardware in some modes, but project code should
not assume every peripheral returns configured.

## GPIO Wake Notes
- If sleep and GPIO input interrupts are both enabled, SDK docs require
  `GPIO_TRIG_RISING_FAILING_EDGE` so the power manager can detect sleep state.
- Confirm pull mode, active level and debounce strategy before using a wake pin.
- GPIO wake behavior can differ between normal GPIO and GPIO in the PMU domain.

## 32K Clock Notes
- Record RC32K vs 32.768 kHz crystal in `PROJECT_NOTES.md`.
- RC32K drift can affect BLE timing and long sleep behavior.
- For RTC/deep sleep tests, verify the 32K source actually runs in the target
  power mode.
- Release notes mention an internal function for large RC32K drift scenarios;
  do not enable undocumented workarounds without vendor/project approval.

## Debugging Sleep
- SWD may disconnect when the system enters sleep. Disable sleep for debug when
  needed using the SDK-supported method.
- Compare behavior with sleep disabled and enabled before blaming BLE/RF logic.
- Use GPIO markers around sleep entry, wake IRQ, peripheral restore and first
  application event.
- Measure current only after logging and debug outputs are disabled or accounted
  for.

## Low-Power Validation
- Active current baseline.
- Idle/sleep/deep-sleep current baseline.
- Wake source verified.
- Wake latency measured if timing matters.
- Peripheral restore verified after wake.
- BLE advertising/connection behavior verified across sleep.
- UART/SPI/I2C/ADC/timer behavior verified after wake when used.

## Safe Change Policy
- Do not change sleep mode globally without checking BLE role and wake sources.
- Keep sleep prevention scoped and paired with sleep allow.
- Avoid long critical sections around power manager entry.
- Record all wake pins, wake levels and retained state in `PROJECT_NOTES.md`.
