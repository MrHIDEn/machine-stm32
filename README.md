# machine_stm32

STM32 port of a MicroPython-shaped **`machine`** API for [Klin](https://github.com/MrHIDEn/klin).

Not a MicroPython port. No GC, no hidden heap, no hidden clocks — Pin configures
the GPIO clock and mode explicitly via MMIO (STM32F4 AHB1).

Decision / catalog: [Klin issue 061](https://github.com/MrHIDEn/klin/blob/main/issues/061-micropython-machine-api.md).

## Status

| API | Status |
|---|---|
| `Pin` (`pin_out` / `pin_in`, `high` / `low` / `toggle` / `set` / `value`) | MVP |
| `Pwm`, `Uart`, … | later |
| Other MCU families | separate repos / ports (e.g. `machine_rp`) |

Target family for addresses: **STM32F411 / F401**-class (Nucleo-F411RE LED = **PA5**).

## Requirements

- [Klin](https://github.com/MrHIDEn/klin) compiler (`klin` or `dart run path/to/bin/klin.dart`)
- For the blink ELF: `arm-none-eabi-gcc`

## Layout

```text
machine_stm32/           # module machine_stm32 (directory package)
  version.kl
  pin.kl
  pin_test.kl            # skipped on import
examples/blink_f411/     # Nucleo-F411RE PA5
```

## Usage

```klin
import "github/mrhiden/machine_stm32" machine

@[link("startup.s")]
fn main() {
    let led = machine.pin_out(machine.Port.A, 5)
    while true {
        led.toggle()
    }
}
```

```sh
klin get github/mrhiden/machine_stm32@v0.1.0
```

Local / in-repo example:

```klin
import "../../machine_stm32" machine
```

## Blink example

```sh
cd examples/blink_f411
make KLIN=/path/to/klin/bin/klin.dart
# → blink.elf
```

## Tests

```sh
dart run /path/to/klin/bin/klin.dart test machine_stm32/
```

## License

MIT
