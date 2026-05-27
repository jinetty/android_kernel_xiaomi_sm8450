kernelsu-objs := core/init.o
kernelsu-objs += feature/kernel_umount.o
kernelsu-objs += feature/sucompat.o

# Architecture
ifneq ($(filter y arm64 aarch64,$(CONFIG_ARM64) $(ARCH)),)
KSU_ARCH := arm64
else ifneq ($(filter y x86_64 i386,$(CONFIG_X86_64) $(ARCH)),)
KSU_ARCH := x86_64
endif

# Core utilities ('selinux hide'/hooks)
ifeq ($(strip $(CONFIG_KPROBES)),y)
kernelsu-objs += hook/lsm_hook.o
ifdef KSU_ARCH
kernelsu-objs += hook/$(KSU_ARCH)/patch_memory.o
endif
endif

# Hooks (excluded for SuSFS)
ifneq ($(strip $(CONFIG_KSU_SUSFS)),y)
ifeq ($(strip $(CONFIG_KPROBES)),y)
kernelsu-objs += hook/syscall_event_bridge.o
kernelsu-objs += hook/syscall_hook_manager.o
kernelsu-objs += hook/tp_marker.o
ifdef KSU_ARCH
kernelsu-objs += hook/$(KSU_ARCH)/syscall_hook.o
endif
endif
endif

kernelsu-objs += hook/setuid_hook.o
kernelsu-objs += extras.o
kernelsu-objs += tiny_sulog.o

kernelsu-objs += infra/file_wrapper.o
kernelsu-objs += infra/seccomp_cache.o
kernelsu-objs += infra/su_mount_ns.o
kernelsu-objs += infra/symbol_resolver.o

ifneq ($(strip $(CONFIG_KSU_DISABLE_MANAGER)),y)
kernelsu-objs += manager/apk_sign.o
kernelsu-objs += manager/pkg_observer.o
kernelsu-objs += manager/throne_tracker.o
endif

kernelsu-objs += policy/allowlist.o
kernelsu-objs += policy/app_profile.o
kernelsu-objs += policy/feature.o

kernelsu-objs += runtime/boot_event.o
kernelsu-objs += runtime/ksud_integration.o

kernelsu-objs += selinux/selinux.o
kernelsu-objs += selinux/rules.o
kernelsu-objs += selinux/sepolicy.o

kernelsu-objs += supercall/dispatch.o
kernelsu-objs += supercall/perm.o
kernelsu-objs += supercall/supercall.o

kernelsu-objs += feature/adb_root.o

kernelsu-objs += feature/selinux_hide.o

ifdef KBUILD_EXTMOD
ifeq ($(strip $(CONFIG_KSU_DISABLE_MANAGER)),y)
ccflags-y += -DCONFIG_KSU_DISABLE_MANAGER=1
endif
ifeq ($(strip $(CONFIG_KSU_DISABLE_POLICY)),y)
ccflags-y += -DCONFIG_KSU_DISABLE_POLICY=1
endif
ifeq ($(strip $(CONFIG_KSU_DEBUG)),y)
ccflags-y += -DCONFIG_KSU_DEBUG=1
endif
endif

ccflags-y += -I$(srctree)/security/selinux -I$(srctree)/security/selinux/include
ccflags-y += -I$(objtree)/security/selinux -include $(srctree)/include/uapi/asm-generic/errno.h

ifeq ($(filter /%,$(src)),)
KSU_KERNEL_DIR := $(srctree)/$(src)
else
KSU_KERNEL_DIR := $(src)
endif

ccflags-y += -I$(KSU_KERNEL_DIR) -I$(KSU_KERNEL_DIR)/include

obj-$(CONFIG_KSU) += kernelsu.o

KSU_VERSION := 33168
$(info -- KernelSU-Next version: $(KSU_VERSION))
ccflags-y += -DKSU_VERSION=$(KSU_VERSION)

KSU_VERSION_TAG := v3.2.0
$(info -- KernelSU-Next tag: $(KSU_VERSION_TAG))
ccflags-y += -DKSU_VERSION_TAG=\"$(KSU_VERSION_TAG)\"

KSU_NEW_DCACHE_FLUSH := $(shell grep -q __flush_dcache_area $(srctree)/arch/arm64/include/asm/cacheflush.h ; echo $$?)
$(info -- KernelSU-Next new DCACHE flush: $(KSU_NEW_DCACHE_FLUSH))

ifndef KSU_NEXT_MANAGER_LIST
# KSUN, pershoot
KSU_NEXT_MANAGER_LIST := 0x3e6:79e590113c4c4c0c222978e413a5faa801666957b1212a328e46c00c69821bf7,0x338:f26471a28031130362bce7eebffb9a0b8afc3095f163ce0c75a309f03b644a1f
endif

ifdef KSU_MANAGER_PACKAGE
ccflags-y += -DKSU_MANAGER_PACKAGE=\"$(KSU_MANAGER_PACKAGE)\"
$(info -- KernelSU-Next Manager package name: $(KSU_MANAGER_PACKAGE))
endif

$(info -- KernelSU-Next Manager signatures: $(KSU_NEXT_MANAGER_LIST))
ccflags-y += -DKSU_NEXT_MANAGER_LIST=\"$(KSU_NEXT_MANAGER_LIST)\"

ccflags-y += -DKSU_NEW_DCACHE_FLUSH=$(KSU_NEW_DCACHE_FLUSH)

ccflags-y += -Wno-strict-prototypes -Wno-int-conversion -Wno-gcc-compat -Wno-missing-prototypes
ccflags-y += -Wno-declaration-after-statement -Wno-unused-function -Wno-unused-variable

## For susfs stuff ##
ifeq ($(strip $(CONFIG_KSU_SUSFS)),y)
ifeq ($(shell test -e $(srctree)/fs/susfs.c; echo $$?),0)
$(eval SUSFS_VERSION=$(shell cat $(srctree)/include/linux/susfs.h | grep -E '^#define SUSFS_VERSION' | cut -d' ' -f3 | sed 's/"//g'))
$(info )
$(info -- SUSFS_VERSION: $(SUSFS_VERSION))
else
$(info -- You have not integrated susfs in your kernel yet.)
$(info -- Read: https://gitlab.com/simonpunk/susfs4ksu)
endif
endif

# Keep a new line here!! Because someone may append config
