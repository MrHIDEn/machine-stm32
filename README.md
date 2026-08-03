# machine_stm32

STM32 port of a MicroPython-shaped **`machine`** API for [Klin](https://github.com/klin-lang/klin).

Not a MicroPython port. No GC, no hidden heap, no hidden clocks — Pin/Pwm/Rc
configure GPIO and timers explicitly via MMIO (STM32F4 AHB1 / APB1).

Decision / catalog: [Klin issue 061](https://github.com/klin-lang/klin/blob/main/issues/061-micropython-machine-api.md).

## Status

| API | Status |
|---|---|
| `Pin` (`pin_out` / `pin_in`, `high` / `low` / `toggle` / `set` / `value`) | MVP |
| `Pwm` (`pwm_out`, `freq`, `duty_u16`, `deinit`) | MVP (`@v0.2.0`) |
| `Rc` (`rc_out`, `out` / `out_f32` / `pulse_us` / `deinit`) | MVP (`@v0.3.0`) |
| `Uart`, … | later |
| Other MCU families | separate repos / ports (e.g. `machine_rp`) |

Target family for addresses: **STM32F411 / F401**-class (hardcoded MMIO — no
runtime chip detect). Nucleo-F411RE LED = **PA5** (GPIO or TIM2_CH1 AF1).

## Requirements

- [Klin](https://github.com/klin-lang/klin) compiler (`klin` or `dart run path/to/bin/klin.dart`)
- For board ELFs: `arm-none-eabi-gcc`

## Layout

```text
machine_stm32/           # module machine_stm32 (directory package)
  version.kl             # version() → 3
  pin.kl
  pwm.kl
  rc.kl
  pin_test.kl / rc_test.kl   # skipped on import
examples/blink_f411/     # Nucleo-F411RE PA5 Pin toggle
examples/pwm_f411/       # PA5 TIM2_CH1 fade
examples/rc_f411/        # PA5 TIM2_CH1 servo sweep
```

## Usage — Pin

```klin
import "github/klin-lang/machine_stm32" machine

@[link("startup.s")]
fn main() {
    let led = machine.pin_out(machine.Port.A, 5)
    while true {
        led.toggle()
    }
}
```

## Usage — Pwm

Caller passes timer, channel, AF, and timer input clock explicitly (after any
APB multiplier). No automatic timer pick or clock-tree read.

```klin
import "github/klin-lang/machine_stm32" machine

@[link("startup.s")]
fn main() {
    // PA5 = TIM2_CH1 AF1; HSI 16 MHz at reset → tim_clk_hz = 16_000_000
    let led = machine.pwm_out(machine.Port.A, 5, 2, 1, 1, 16000000)
    led.freq(1000)
    led.duty_u16(32768)   // ~50%
    while true {}
}
```

## Usage — Rc (servo / RC pulse)

Same HW args as `pwm_out`, plus frame rate and pulse range in µs.
`pos` is `0..=100_000` (or `out_f32` `0.0..=1.0`); `k` is trim µs (−1000..=1000).

```klin
import "github/klin-lang/machine_stm32" machine

@[link("startup.s")]
fn main() {
    let servo = machine.rc_out(machine.Port.A, 5, 2, 1, 1, 16000000, 50, 1000, 2000)
    servo.out(50000, 0)       // center
    servo.out_f32(0.25, 10)   // 25% + 10 µs trim
    servo.pulse_us(1500)
}
```

```sh
klin get github/klin-lang/machine_stm32@v0.3.0
```

## Pwm / Rc shape (convention for other `machine_*` ports)

| Piece | Role |
|---|---|
| `pwm_out(…)` | factory — chip-specific args OK |
| `freq` / `duty_u16` / `deinit` | MicroPython-style PWM |
| `rc_out(…, freq_hz, us_min, us_max)` | servo/RC on same HW args |
| `out` / `out_f32` / `pulse_us` / `deinit` | position + trim / raw µs |

## Examples

```sh
cd examples/blink_f411
make KLIN=/path/to/klin/bin/klin.dart
# → blink.elf

cd examples/pwm_f411
make KLIN=/path/to/klin/bin/klin.dart
# → pwm.elf

cd examples/rc_f411
make KLIN=/path/to/klin/bin/klin.dart
# → rc.elf
```

## Tests

```sh
dart run /path/to/klin/bin/klin.dart test machine_stm32/
```

## License

MIT
