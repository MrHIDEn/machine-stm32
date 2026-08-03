# := so a host CC from the environment does not win
CC := arm-none-eabi-gcc
CFLAGS := -mcpu=cortex-m4 -mthumb -ffreestanding -Os -ffunction-sections -fdata-sections
LDFLAGS := -T linker.ld -Wl,--gc-sections -nostdlib

# Path to Klin entry (bin/klin.dart) or a `klin` binary on PATH.
#   make KLIN=/path/to/klin/bin/klin.dart
KLIN ?= klin

.PHONY: all clean

all: rc.elf

LINK_OBJS := $(shell test -f out/rc.link && cat out/rc.link)

out/rc.c out/rc.link: rc.kl $(wildcard ../../machine_stm32/*.kl)
	@case "$(KLIN)" in \
	  *.dart) dart run "$(KLIN)" --emit-c rc.kl ;; \
	  *) "$(KLIN)" --emit-c rc.kl ;; \
	esac

rc.elf: linker.ld out/rc.c out/rc.link $(LINK_OBJS)
	$(CC) $(CFLAGS) -I. $(LDFLAGS) $$(cat out/rc.link) out/rc.c -o $@

clean:
	rm -rf out rc.elf
