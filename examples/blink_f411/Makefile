# := so a host CC from the environment does not win
CC := arm-none-eabi-gcc
CFLAGS := -mcpu=cortex-m4 -mthumb -ffreestanding -Os -ffunction-sections -fdata-sections
LDFLAGS := -T linker.ld -Wl,--gc-sections -nostdlib

# Path to Klin entry (bin/klin.dart) or a `klin` binary on PATH.
#   make KLIN=/path/to/klin/bin/klin.dart
KLIN ?= klin

.PHONY: all clean

all: blink.elf

LINK_OBJS := $(shell test -f out/blink.link && cat out/blink.link)

out/blink.c out/blink.link: blink.kl $(wildcard ../../machine_stm32/*.kl)
	@case "$(KLIN)" in \
	  *.dart) dart run "$(KLIN)" --emit-c blink.kl ;; \
	  *) "$(KLIN)" --emit-c blink.kl ;; \
	esac

blink.elf: linker.ld out/blink.c out/blink.link $(LINK_OBJS)
	$(CC) $(CFLAGS) -I. $(LDFLAGS) $$(cat out/blink.link) out/blink.c -o $@

clean:
	rm -rf out blink.elf
