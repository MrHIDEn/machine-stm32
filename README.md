# machine_stm32

STM32 port of a MicroPython-shaped **`machine`** API for [Klin](https://github.com/klin-lang/klin).

Not a MicroPython port. No GC, no hidden heap, no hidden clocks — Pin/Pwm
configure GPIO and timers explicitly via MMIO (STM32F4 AHB1 / APB1).

Decision / catalog: [Klin issue 061](https://github.com/klin-lang/klin/blob/main/issues/061-micropython-machine-api.md).

## Status

| API | Status |
|---|---|
| `Pin` (`pin_out` / `pin_in`, `high` / `low` / `toggle` / `set` / `value`) | MVP |
| `Pwm` (`pwm_out`, `freq`, `duty_u16`, `deinit`) | MVP (`@v0.2.0`) |
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
  version.kl             # version() → 2
  pin.kl
  pwm.kl
  pin_test.kl            # skipped on import
examples/blink_f411/     # Nucleo-F411RE PA5 Pin toggle
examples/pwm_f411/       # PA5 TIM2_CH1 fade
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

```sh
klin get github/klin-lang/machine_stm32@v0.2.0
```

## Pwm shape (convention for other `machine_*` ports)

Same *names* across packages; separate MMIO implementations (not one shared
runtime). Ports should aim for:

| Piece | Role |
|---|---|
| `pwm_out(…)` | factory — chip-specific args OK |
| `freq(hz)` | set frequency in Hz |
| `duty_u16(d)` | duty 0..=65535 (MicroPython-style) |
| `deinit()` | stop this PWM (explicit) |

`machine_rp` / `machine_esp` may use flatter `pwm_out(gpio)` later; method
names stay aligned.

## Examples

```sh
cd examples/blink_f411
make KLIN=/path/to/klin/bin/klin.dart
# → blink.elf

cd examples/pwm_f411
make KLIN=/path/to/klin/bin/klin.dart
# → pwm.elf
```

## Tests

```sh
dart run /path/to/klin/bin/klin.dart test machine_stm32/
```

## License

MIT
