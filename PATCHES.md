# Patches
For an up to date difference report, please use `git diff` between branches:
```
$ git clone https://github.com/IOTSolutionsAS/u-boot.git -b sunxi-v2017.x_mender
$ git diff origin/sunxi-v2017.x origin/sunxi-v2017.x_mender
```

The following files have been changed based on patches from [Meta Mender - patches](https://github.com/mendersoftware/meta-mender/tree/master/meta-mender-core/recipes-bsp/u-boot/patches)

* 0001-Add-missing-header-which-fails-on-recent-GCC.patch
* 0002-Generic-boot-code-for-Mender.patch
* 0003-Integration-of-Mender-boot-code-into-U-Boot.patch
* 0004-Disable-CONFIG_BOOTCOMMAND-and-enable-CONFIG_MENDER_.patch

These patches may (or may not) be applied autopmatically by using `git apply <patch>`. If automatic `apply` does not work, the patches must be included manually.

## configs/nanopi_h3_defconfig
Ref. [Mender from scratch](https://hub.mender.io/t/mender-from-scratch/391)<br>
Remove:
* `CONFIG_ENV_OFFSET=0x200000`

Add:
* `CONFIG_ENV_IS_IN_MMC=y`
* `CONFIG_USE_BOOTARGS=y`
* `CONFIG_BOOTARGS="console=ttyS0,115200 panic=10 rootfstype=ext4 rw rootwait fsck.epair=yes"`

## env/Kconfig
Ref. [Meta Mender - patches](https://github.com/mendersoftware/meta-mender/tree/master/meta-mender-core/recipes-bsp/u-boot/patches)<br>
Change:
* ENV_OFFSET: `default 0x400000 if ARCH_SUNXI` (was 0x88000)
* ENV_SIZE: `default 0x4000 if ARCH_SUNXI` (was 0x20000)

## include/config_mender.h
New file. Ref. [Meta Mender - patches](https://github.com/mendersoftware/meta-mender/tree/master/meta-mender-core/recipes-bsp/u-boot/patches)

## include/config_mender_defines.h
New file. Ref. [Mender from scratch](https://hub.mender.io/t/mender-from-scratch/391)
```
#ifndef HEADER_CONFIG_MENDER_DEFINES_H
#define HEADER_CONFIG_MENDER_DEFINES_H
#endif

/* Shell variables */
#define MENDER_BOOT_PART_NUMBER 1
#define MENDER_BOOT_PART_NUMBER_HEX 1
#define MENDER_ROOTFS_PART_A_NUMBER 2
#define MENDER_ROOTFS_PART_A_NUMBER_HEX 2
#define MENDER_ROOTFS_PART_B_NUMBER 3
#define MENDER_ROOTFS_PART_B_NUMBER_HEX 3
#define MENDER_UBOOT_STORAGE_INTERFACE "mmc"
#define MENDER_UBOOT_STORAGE_DEVICE 0

/* BB variables. */
#define MENDER_STORAGE_DEVICE_BASE "/dev/mmcblk0p"
#define MENDER_UBOOT_ENV_STORAGE_DEVICE_OFFSET_1 0x400000
#define MENDER_UBOOT_ENV_STORAGE_DEVICE_OFFSET_2 0x800000
#define MENDER_ROOTFS_PART_A_NAME "/dev/mmcblk0p2"
#define MENDER_ROOTFS_PART_B_NAME "/dev/mmcblk0p3"
#define MENDER_MTD_UBI_DEVICE_NAME ""

/* For sanity checks. */
#define MENDER_BOOTENV_SIZE 0x4000

#define MENDER_BOOT_KERNEL_TYPE "bootz"
#define MENDER_KERNEL_NAME "zImage"
#define MENDER_DTB_NAME "sun8i-h3-nanopi-r1.dtb"
#define MENDER_UBOOT_PRE_SETUP_COMMANDS ""
#define MENDER_UBOOT_POST_SETUP_COMMANDS ""
```

## include/configs/sun8i.h
_Note: Additions should be in `include/friendlyelec/boardtype.h`, but this file is not included early enough..._<br>
Add:
* `#define CONFIG_BOOTCOUNT_LIMIT`
* `#define CONFIG_BOOTCOUNT_ENV`
* `#define CONFIG_SYS_REDUNDAND_ENVIRONMENT`

## include/env_default.h
Ref. [Meta Mender - patches](https://github.com/mendersoftware/meta-mender/tree/master/meta-mender-core/recipes-bsp/u-boot/patches)<br>
Add:
* `#include <env_mender.h>`
* `MENDER_ENV_SETTINGS`

Change:
* `CONFIG_BOOTCOMMAND` -> `CONFIG_MENDER_BOOTCOMMAND`

## include/env_mender.h
New file. Ref. [Meta Mender - patches](https://github.com/mendersoftware/meta-mender/tree/master/meta-mender-core/recipes-bsp/u-boot/patches)

Change the `MENDER_LOAD_KERNEL_AND_FDT` to boot on correctly:
```
# define MENDER_LOAD_KERNEL_AND_FDT                                         \
    "if test \"${fdt_addr_r}\" != \"\"; then "                              \
    "load ${mender_uboot_root} ${fdt_addr_r} /boot/${mender_dtb_name}; "    \
    "fdt addr ${fdt_addr_r}; "                                              \
    "fdt set mmc${boot_mmc} boot_device <1>; "                              \
    "fi; "                                                                  \
    "load ${mender_uboot_root} ${kernel_addr_r} /boot/${mender_kernel_name}; "
#endif
```

## scripts/Makefile.autoconf
Ref. [Meta Mender - patches](https://github.com/mendersoftware/meta-mender/tree/master/meta-mender-core/recipes-bsp/u-boot/patches)<br>
Add:
* `echo \#include \<config_mender.h\>`

## tools/env/fw_env.c
Ref. [Meta Mender - patches](https://github.com/mendersoftware/meta-mender/tree/master/meta-mender-core/recipes-bsp/u-boot/patches)<br>
Add:
* `#include <stdint.h>`
