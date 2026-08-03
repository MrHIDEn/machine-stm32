# machine_stm32

STM32 port of a MicroPython-shaped **`machine`** API for [Klin](https://github.com/klin-lang/klin).

Not a MicroPython port. No GC, no hidden heap, no hidden clocks — peripherals
are configured explicitly via MMIO (STM32F4 AHB1 / APB1 / APB2).

Decision / catalog: [Klin issue 061](https://github.com/klin-lang/klin/blob/main/issues/061-micropython-machine-api.md).

## Status

| API | Status |
|---|---|
| `Pin` (`pin_out` / `pin_in`, `high` / `low` / `toggle` / `set` / `value`) | MVP |
| `Pwm` (`pwm_out`, `freq`, `duty_u16`, `deinit`) | MVP (`@v0.2.0`) |
| `Rc` (`rc_out`, `out` / `out_f32` / `pulse_us`, `deinit`) | MVP (`@v0.3.0`) |
| `Uart` (`uart_out`, `write_u8` / `write` / `read_u8` / `try_read_u8` / `any` / `deinit`) | MVP (`@v0.4.0`) |
| `I2c` (`i2c_out`, `writeto` / `readfrom_into` / `write_readfrom_into` / `deinit`) | MVP (`@v0.5.0`) |
| `Spi` (`spi_out`, `write_read_u8` / `write` / `readinto` / `write_readinto` / `deinit`) | MVP (`@v0.5.0`) |
| `Adc` (`adc_out`, `read_u12` / `read_u16` / `deinit`) | MVP (`@v0.5.0`) |
| Other MCU families | separate repos / ports (e.g. `machine_rp`) |

Target family for addresses: **STM32F411 / F401**-class (hardcoded MMIO — no
runtime chip detect). Nucleo-F411RE: LED **PA5**, VCP **USART2 PA2/PA3**,
Arduino I2C1 **PB8/PB9**, SPI2 **PB13/14/15**, A0 **PA0**.

## Requirements

- [Klin](https://github.com/klin-lang/klin) compiler (`klin` or `dart run path/to/bin/klin.dart`)
- For board ELFs: `arm-none-eabi-gcc`

## Layout

```text
machine_stm32/           # module machine_stm32 (directory package)
  version.kl             # version() → 5
  pin.kl / pwm.kl / rc.kl / uart.kl / i2c.kl / spi.kl / adc.kl
  *_test.kl              # skipped on import
examples/blink_f411/     # PA5 Pin toggle
examples/pwm_f411/       # PA5 TIM2_CH1 fade
examples/rc_f411/        # PA5 TIM2_CH1 servo sweep
examples/uart_f411/      # USART2 VCP hello + echo
examples/i2c_f411/       # I2C1 PB8/PB9 init + LED blink
examples/spi_f411/       # SPI2 PB13/14/15 clock out
examples/adc_f411/       # ADC1 PA0 → PWM PA5
```

## Usage — Pin / Pwm / Rc / Uart

See earlier tags / issue 061. Short forms:

```klin
import "github/klin-lang/machine_stm32" machine

let led = machine.pin_out(machine.Port.A, 5)
let pwm = machine.pwm_out(machine.Port.A, 5, 2, 1, 1, 16000000)
let servo = machine.rc_out(machine.Port.A, 5, 2, 1, 1, 16000000, 50, 1000, 2000)
let u = machine.uart_out(2, machine.Port.A, 2, 7, machine.Port.A, 3, 7, 16000000, 115200)
```

## Usage — I2c

Caller passes instance (1..=3), SCL/SDA port+pin+AF, APB clock, and bus `freq_hz`.
Pins are AF open-drain + pull-up. 7-bit addresses. Blocking transfers.

```klin
import "github/klin-lang/machine_stm32" machine

@[link("startup.s")]
fn main() {
    let bus = machine.i2c_out(
        1,
        machine.Port.B, 8, 4,
        machine.Port.B, 9, 4,
        16000000,
        100000
    )
    let mut w: [1]u8
    w[0] = 0x00
    bus.writeto(0x50, w)
    let mut r: [2]u8
    bus.readfrom_into(0x50, r)
    bus.write_readfrom_into(0x50, 0x00, r)
}
```

## Usage — Spi

Master, soft NSS (`SSM`+`SSI`) — drive CS with a separate `Pin`. `mode` is 0..=3.

```klin
import "github/klin-lang/machine_stm32" machine

@[link("startup.s")]
fn main() {
    let s = machine.spi_out(
        2,
        machine.Port.B, 13, 5,
        machine.Port.B, 14, 5,
        machine.Port.B, 15, 5,
        16000000,
        1000000,
        0
    )
    let cs = machine.pin_out(machine.Port.B, 12)
    cs.low()
    let v = s.write_read_u8(0x9F)
    cs.high()
}
```

## Usage — Adc

ADC1 only on F411. Pass GPIO + channel explicitly (no hidden pin map).

```klin
import "github/klin-lang/machine_stm32" machine

@[link("startup.s")]
fn main() {
    // PA0 = ADC1_IN0
    let adc = machine.adc_out(machine.Port.A, 0, 0)
    let raw = adc.read_u12()   // 0..=4095
    let u16 = adc.read_u16()   // 0..=65535
}
```

```sh
klin get github/klin-lang/machine_stm32@v0.5.0
```

## Shape (convention for other `machine_*` ports)

| Piece | Role |
|---|---|
| `*_out(…)` | factory — chip-specific args OK; clocks explicit |
| `deinit()` | stop peripheral (explicit) |
| `writeto` / `readfrom_into` | I2C into caller buffers (no heap) |
| `write_read_u8` / `readinto` | SPI full-duplex / fill buffer |
| `read_u12` / `read_u16` | ADC raw / MicroPython-scaled |

## Examples

```sh
cd examples/adc_f411   # or i2c_f411 / spi_f411 / …
make KLIN=/path/to/klin/bin/klin.dart
```

## Tests

```sh
dart run /path/to/klin/bin/klin.dart test machine_stm32/
```

## License

MIT
