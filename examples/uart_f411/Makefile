# := so a host CC from the environment does not win
CC := arm-none-eabi-gcc
CFLAGS := -mcpu=cortex-m4 -mthumb -ffreestanding -Os -ffunction-sections -fdata-sections
LDFLAGS := -T linker.ld -Wl,--gc-sections -nostdlib

# Path to Klin entry (bin/klin.dart) or a `klin` binary on PATH.
#   make KLIN=/path/to/klin/bin/klin.dart
KLIN ?= klin

.PHONY: all clean

all: uart.elf

LINK_OBJS := $(shell test -f out/uart.link && cat out/uart.link)

out/uart.c out/uart.link: uart.kl $(wildcard ../../machine_stm32/*.kl)
	@case "$(KLIN)" in \
	  *.dart) dart run "$(KLIN)" --emit-c uart.kl ;; \
	  *) "$(KLIN)" --emit-c uart.kl ;; \
	esac

uart.elf: linker.ld out/uart.c out/uart.link $(LINK_OBJS)
	$(CC) $(CFLAGS) -I. $(LDFLAGS) $$(cat out/uart.link) out/uart.c -o $@

clean:
	rm -rf out uart.elf
