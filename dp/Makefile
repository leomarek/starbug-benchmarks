# -------------------------------------------------------------------
# Toolchain and Paths
# -------------------------------------------------------------------
RISCV_PREFIX   = riscv64-unknown-elf
GCC_TOOLCHAIN  = /opt/homebrew

# Toolchain binaries
CC        = $(GCC_TOOLCHAIN)/bin/$(RISCV_PREFIX)-gcc
WALLY_CC  = $(RISCV_PREFIX)-gcc
OBJDUMP   = $(GCC_TOOLCHAIN)/bin/$(RISCV_PREFIX)-objdump
SIZE      = $(GCC_TOOLCHAIN)/bin/$(RISCV_PREFIX)-size
WALLY_SIZE = $(RISCV_PREFIX)-size
# Common paths (adjust if needed)
COMMON_DIR = ../../common
WALLY_LINKER_SCRIPT = $(COMMON_DIR)/test.ld


# -------------------------------------------------------------------
# Common Flags
# -------------------------------------------------------------------
CFLAGS   = -O0 -march=rv32imac -mabi=ilp32 -Wall
LDFLAGS  = -march=rv32imac -specs=semihost.specs -mabi=ilp32 -T link.ld

# Wally (bare-metal) Flags
ARCH  = rv32gc
ABI   = ilp32
WALLY_CFLAGS  = -O2 -march=$(ARCH) -mabi=$(ABI) -Wall -nostdlib -static -lm -fno-tree-loop-distribute-patterns
WALLY_LDFLAGS = -T$(WALLY_LINKER_SCRIPT) -I$(COMMON_DIR)

# -------------------------------------------------------------------
# Sources
# -------------------------------------------------------------------
DRIVER = dp_main.c
DATA   = data.S

SCALAR_SRC = dp_scalar.c
VLIW_SRC   = dp_vliw_better.S
EMPTY_SRC  = dp_empty.c

# Output binaries
SCALAR_ELF = dp_scalar.elf
VLIW_ELF   = dp_vliw.elf
EMPTY_ELF  = dp_empty.elf

SCALAR_WALLY_ELF = dp_scalar_wally.elf
VLIW_WALLY_ELF   = dp_vliw_wally.elf
EMPTY_WALLY_ELF  = dp_empty_wally.elf

# -------------------------------------------------------------------
# Phony Targets
# -------------------------------------------------------------------
.PHONY: all scalar vliw empty \
        scalar-wally vliw-wally empty-wally \
        dump clean

# -------------------------------------------------------------------
# Default (semihosted) builds
# -------------------------------------------------------------------
all: scalar vliw empty

scalar: $(SCALAR_ELF)
vliw:   $(VLIW_ELF)
empty:  $(EMPTY_ELF)

$(SCALAR_ELF): $(DRIVER) $(SCALAR_SRC) $(DATA)
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^
	$(SIZE) $@

$(VLIW_ELF): $(DRIVER) $(VLIW_SRC) $(DATA)
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^
	$(SIZE) $@

$(EMPTY_ELF): $(DRIVER) $(EMPTY_SRC) $(DATA)
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^
	$(SIZE) $@

# -------------------------------------------------------------------
# Wally (bare-metal) builds
# -------------------------------------------------------------------
scalar-wally: $(SCALAR_WALLY_ELF)
vliw-wally:   $(VLIW_WALLY_ELF)
empty-wally:  $(EMPTY_WALLY_ELF)

# -------------------------------------------------------------------
# Wally (bare-metal) builds — fixed
# -------------------------------------------------------------------
scalar-wally: $(SCALAR_WALLY_ELF)
vliw-wally:   $(VLIW_WALLY_ELF)
empty-wally:  $(EMPTY_WALLY_ELF)

$(SCALAR_WALLY_ELF): $(DRIVER) $(SCALAR_SRC) $(DATA) $(COMMON_DIR)/crt.S $(COMMON_DIR)/syscalls.c
	$(WALLY_CC) -O2 -gdwarf-2 \
	  -march=$(ARCH) -mabi=$(ABI) -mcmodel=medany \
	  -nostdlib -static -fno-tree-loop-distribute-patterns \
	  -T$(WALLY_LINKER_SCRIPT) -I$(COMMON_DIR) \
	  $(COMMON_DIR)/crt.S $(DRIVER) $(SCALAR_SRC) $(DATA) $(COMMON_DIR)/syscalls.c \
	  -lgcc -o $@
	$(WALLY_SIZE) $@

$(VLIW_WALLY_ELF): $(DRIVER) $(VLIW_SRC) $(DATA) $(COMMON_DIR)/crt.S $(COMMON_DIR)/syscalls.c
	$(WALLY_CC) -O2 -gdwarf-2 \
	  -march=$(ARCH) -mabi=$(ABI) -mcmodel=medany \
	  -nostdlib -static -fno-tree-loop-distribute-patterns \
	  -T$(WALLY_LINKER_SCRIPT) -I$(COMMON_DIR) \
	  $(COMMON_DIR)/crt.S $(DRIVER) $(VLIW_SRC) $(DATA) $(COMMON_DIR)/syscalls.c \
	  -lgcc -o $@
	$(WALLY_SIZE) $@

$(EMPTY_WALLY_ELF): $(DRIVER) $(EMPTY_SRC) $(DATA) $(COMMON_DIR)/crt.S $(COMMON_DIR)/syscalls.c
	$(WALLY_CC) -O2 -gdwarf-2 \
	  -march=$(ARCH) -mabi=$(ABI) -mcmodel=medany \
	  -nostdlib -static -fno-tree-loop-distribute-patterns \
	  -T$(WALLY_LINKER_SCRIPT) -I$(COMMON_DIR) \
	  $(COMMON_DIR)/crt.S $(DRIVER) $(EMPTY_SRC) $(DATA) $(COMMON_DIR)/syscalls.c \
	  -lgcc -o $@
	$(WALLY_SIZE) $@


# -------------------------------------------------------------------
# Utility Targets
# -------------------------------------------------------------------
dump:
	$(OBJDUMP) -S -D $(SCALAR_ELF) > $(SCALAR_ELF).dump
	$(OBJDUMP) -S -D $(VLIW_ELF)   > $(VLIW_ELF).dump
	$(OBJDUMP) -S -D $(EMPTY_ELF)  > $(EMPTY_ELF).dump
	$(OBJDUMP) -S -D $(SCALAR_WALLY_ELF) > $(SCALAR_WALLY_ELF).dump
	$(OBJDUMP) -S -D $(VLIW_WALLY_ELF)   > $(VLIW_WALLY_ELF).dump
	$(OBJDUMP) -S -D $(EMPTY_WALLY_ELF)  > $(EMPTY_WALLY_ELF).dump

clean:
	rm -f *.o *.elf *.bin *.hex *.dump *.objdump
