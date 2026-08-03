# := so a host CC from the environment does not win
CC := arm-none-eabi-gcc
CFLAGS := -mcpu=cortex-m4 -mthumb -ffreestanding -Os -ffunction-sections -fdata-sections
LDFLAGS := -T linker.ld -Wl,--gc-sections -nostdlib

# Path to Klin entry (bin/klin.dart) or a `klin` binary on PATH.
#   make KLIN=/path/to/klin/bin/klin.dart
KLIN ?= klin

.PHONY: all clean

all: pwm.elf

LINK_OBJS := $(shell test -f out/pwm.link && cat out/pwm.link)

out/pwm.c out/pwm.link: pwm.kl $(wildcard ../../machine_stm32/*.kl)
	@case "$(KLIN)" in \
	  *.dart) dart run "$(KLIN)" --emit-c pwm.kl ;; \
	  *) "$(KLIN)" --emit-c pwm.kl ;; \
	esac

pwm.elf: linker.ld out/pwm.c out/pwm.link $(LINK_OBJS)
	$(CC) $(CFLAGS) -I. $(LDFLAGS) $$(cat out/pwm.link) out/pwm.c -o $@

clean:
	rm -rf out pwm.elf
