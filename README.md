# machine-stm32

STM32 port of a MicroPython-shaped **`machine`** API for [Klin](https://github.com/MrHIDEn/klin).

Not a MicroPython port. No GC, no hidden heap, no hidden clocks — Pin configures
the GPIO clock and mode explicitly via MMIO (STM32F4 AHB1).

Decision / catalog: [Klin issue 061](https://github.com/MrHIDEn/klin/blob/main/issues/061-micropython-machine-api.md).

## Status

| API | Status |
|---|---|
| `Pin` (`pin_out` / `pin_in`, `high` / `low` / `toggle` / `set` / `value`) | MVP |
| `Pwm`, `Uart`, … | later |
| Other MCU families | separate repos / ports |

Target family for addresses: **STM32F411 / F401**-class (Nucleo-F411RE LED = **PA5**).

## Requirements

- [Klin](https://github.com/MrHIDEn/klin) compiler (`klin` or `dart run path/to/bin/klin.dart`)
- For the blink ELF: `arm-none-eabi-gcc`

## Layout

```text
machine/                 # module machine (directory package)
  version.kl
  pin.kl
  pin_test.kl            # skipped on import
examples/blink_f411/     # Nucleo-F411RE PA5
```

## Usage

Remote `import "github/mrhiden/machine-stm32"` does **not** work yet: the repo
name has a hyphen, and Klin requires the last import segment to match a valid
`module` identifier. Use a path or `KLIN_PATH`:

```klin
import "../../machine"        // from examples in this repo
// or with KLIN_PATH=<clone-root>:
import machine
```

```klin
import machine

@[link("startup.s")]
fn main() {
    let led = machine.pin_out(machine.Port.A, 5)
    while true {
        led.toggle()
    }
}
```

Optional later: rename this GitHub repo to `machine_stm32` for `klin get` parity
with [`osa`](https://github.com/MrHIDEn/osa).

## Blink example

```sh
# from a Klin checkout, or with `klin` on PATH:
cd examples/blink_f411
make KLIN=/path/to/klin/bin/klin.dart
# → blink.elf  (flash with your usual OpenOCD / probe tool)
```

## Tests

```sh
# needs Klin on PATH or:
dart run /path/to/klin/bin/klin.dart test machine/
```

## License

MIT
