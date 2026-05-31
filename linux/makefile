SHELL=/bin/bash
ULIMIT=ulimit -t 300

ifeq ($(TARGET), 8960)
ARCHV := 4
H2K_KERNEL_PGSIZE := 3
H2K_ALLOC_HEAP_SIZE := 0xb000
NO_LOAD=1
NO_PRINT=1
H2K_LOAD_ADDR=0xbf800000
LINUX_LINK_ADDR=0x80400000
endif

ifeq ($(TARGET), 8974)
ARCHV := 5
H2K_KERNEL_PGSIZE := 3
H2K_ALLOC_HEAP_SIZE := 0xb000
NO_LOAD=1
NO_PRINT=1
H2K_LOAD_ADDR=0x0
LINUX_LINK_ADDR=0x02000000
endif

ifeq ($(TARGET), zebu_v60)
ARCHV := 60
H2K_KERNEL_PGSIZE := 3
H2K_ALLOC_HEAP_SIZE := 0xb000
#NO_LOAD=1
#NO_PRINT=1
H2K_LOAD_ADDR=0x0
LINUX_LINK_ADDR=0x04000000
endif

ifeq ($(TARGET), zebu_v65)
ARCHV := 65
H2K_KERNEL_PGSIZE := 3
H2K_ALLOC_HEAP_SIZE := 0xb000
#NO_LOAD=1
#NO_PRINT=1
H2K_LOAD_ADDR=0x0
LINUX_LINK_ADDR=0x04000000
endif

include ../scripts/Makefile.inc.tools

H2DIR := ..

ifeq ($(INSTALLPATH),)
export INSTALLPATH := $(H2DIR)/install
endif

ifeq ($(KERNELPATH),)
export KERNELPATH := $(H2DIR)/kernel
endif

KERNEL_BUILD_DIR := $(patsubst %/install,%/build/kernel,$(INSTALLPATH))

OPTIMIZE=-O2

BOOTVM_PROG := loadlinux

CFLAGS = -Wall -G0 -g $(OPTIMIZE) -mv${TOOLARCH} -DARCHV=${ARCHV} -I. -Iinc -Isrc -I$(INSTALLPATH)/include -I$(KERNEL_BUILD_DIR)/include -L. -L$(INSTALLPATH)/lib $(EXTRA_CFLAGS) -DQEMU_DEVICE_TREE=$(QEMU_DEVICE_TREE)
LDFLAGS = -L$(INSTALLPATH)/lib -moslib=h2

include ../scripts/Makefile.inc.config
include ../scripts/Makefile.inc.opensource
include ../scripts/Makefile.inc.bootvm


ifeq ($(LINUX_LINK_ADDR),)
#LINUX_LINK_ADDR = $(H2K_GUEST_START)
LINUX_LINK_ADDR = 0x80400000
endif

ifeq ($(LINUX_OFFSET_ADDR),)
LINUX_OFFSET_ADDR = 0x00000000
endif

LINUX_STEXT = $(LINUX_LINK_ADDR)

ifeq ($(LINUX_LOAD_ADDR),)
LINUX_LOAD_ADDR = $(LINUX_LINK_ADDR)
endif

ifeq ($(FRAME_BUFFER),)
FRAME_BUFFER = 0xbf400000
endif

ifeq ($(MILLISECONDS),)
MILLISECONDS = 5
endif

ifneq ($(NO_PRINT),)
CFLAGS += -DNO_PRINT
endif

ifneq ($(NO_LOAD),)
CFLAGS += -DNO_LOAD
endif

ifneq ($(NO_STLB),)
CFLAGS += -DNO_STLB
endif

ifneq ($(INT_SHIFT),)
CFLAGS += -DINT_SHIFT
endif

LINUX = 1

ifneq ($(LINUX),)
CFLAGS += -DLINUX
endif

CFLAGS += -DLINUX_LINK_ADDR=$(LINUX_LINK_ADDR) -DLINUX_LOAD_ADDR=$(LINUX_LOAD_ADDR) -DLINUX_OFFSET_ADDR=$(LINUX_OFFSET_ADDR) -DFRAME_BUFFER=$(FRAME_BUFFER)

BOOT_OBJ = boot.o

.PHONY: all clean sim

all: loadlinux

$(BOOTVM_IMAGE): loadlinux.o loadlinux_event.o $(BOOTVM_IMAGE_DEPS)
	${CC} $(LDFLAGS) $(CFLAGS) $(BOOTVM_IMAGE_CFLAGS) -Wl,--defsym=HEAP_SIZE=0x10000 -Wl,--defsym=STACK_SIZE=0x1000 -Wl,--defsym=linux_stext=$(LINUX_STEXT) -o $@ loadlinux.o loadlinux_event.o $(BOOTVM_ENTRY_O)

clean:
	rm -f *.o loadlinux $(BOOTVM_CLEAN)


sim test.log : loadlinux
	$(ULIMIT) && $(RUN) $(SIMF) -- loadlinux 2>&1 | tee test.log
