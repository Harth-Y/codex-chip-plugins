# Board Notes

## Scope
Use this reference for OM6626 board bring-up, GPIO, pinmux, UART, boot pin,
external flash pins, EVB defaults and external peripheral wiring.

## Default EVB Files
- Board header: `bsp/OM662X_EVB/board_om6626a_evb.h`.
- Board source: `bsp/OM662X_EVB/board_om6626a_evb.c`.
- Board init configures DCDC, 32K source, pull modes, drive current, pinmux and
  GPIO.

## Default Pins Observed
- UART0 TXD: IO5.
- UART0 RXD: IO6.
- UART1 TXD: IO4.
- UART1 RXD: IO3.
- LED pads: IO8, IO9, IO10, IO11.
- Button pads: IO2, IO17.
- LED off level: `GPIO_LEVEL_HIGH`.
- Button GPIO trigger in EVB config: `GPIO_TRIG_RISING_FAILING_EDGE`.

## Boot Pin
- SDK bootloader docs identify GPIO4 as BOOT PIN.
- Low at reset enters ISP mode.
- High with valid app enters boot mode.
- GPIO4 may also be used as UART1 TXD in the EVB default header, so board usage
  must be checked before relying on both behaviors.

## External Flash Pins
Bootloader docs list external flash pins:

- MISO: IO7.
- CS: IO8.
- CLK: IO9.
- MOSI: IO10.

Treat these pins as flash-sensitive. Avoid assigning them to LEDs or peripherals
in products that use external flash.

## Pinmux Policy
- Verify pad number, mux function, pull mode, drive current and active level.
- Keep board-specific changes in board files or a project-local board layer.
- Do not assume EVB pin choices are valid for product hardware.
- Record every product pin assignment in `PROJECT_NOTES.md`.
- For sleep wake pins, also record wake level and pull mode.

## UART and Debug
- Record which UART is used for logs, HCI, production test or boot/ISP.
- Avoid mixing debug UART with pins required for boot mode or external flash
  unless the board explicitly supports it.
- Disable or account for logging during low-current measurement.

## GPIO and IRQ
- GPIO input used with sleep may need both-edge trigger as documented by SDK
  power notes.
- ISR-shared flags require correct `volatile` and critical-section handling.
- Use GPIO timing markers for RF, IRQ and sleep debugging.
- Confirm IRQ polarity, clear method and debounce requirements from the active
  peripheral or board circuit.

## External Peripherals
For each peripheral, document:

- Bus: GPIO, UART, SPI, I2C, ADC, timer/PWM or custom.
- Pin numbers and mux.
- Pull mode and reset state.
- Active level and interrupt polarity.
- Required power sequencing or delays.
- Whether it must work after sleep wakeup.
- Validation method and expected signal levels.

## Bring-Up Checklist
- Confirm power rails and DCDC/LDO mode.
- Confirm 32K source.
- Confirm boot pin level.
- Confirm UART logs or SWD access.
- Confirm pinmux for LEDs/buttons/peripherals.
- Confirm no conflict with external flash pins.
- Confirm sleep wake pins before enabling deep sleep.
- Capture key signals if RF, timer, IRQ or low-power behavior is involved.
