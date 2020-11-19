# Introduction
Please see the original readme file: [README](README)

(C) Copyright 2020, [IOT Solutions AS](https://www.iotsolutions.no/), Johan Åtland Førsvoll

SPDX-License-Identifier:	GPL-2.0+

****************

This is a modified U-Boot port for [NanoPi R1](https://www.friendlyarm.com/index.php?route=product/product&product_id=248) from [FriendylElec/FriendlyArm](https://www.friendlyarm.com/) implementing patches for [Mender](https://mender.io/) so that the Mender client can run on the devices. The sorce code is a GitHub fork of the [FriendlyARM U-Boot source](https://github.com/friendlyarm/u-boot.git). The modifications are based on branch `sunxi-v2017.x`. This branch is further branched into `sunxi-v2017.x_mender`

This GitHub fork of Nano Pi R1 U-Boot including Mender patches can be located on [github.com/IOTSolutionsAS/u-boot/tree/sunxi-v2017.x_mender](https://github.com/IOTSolutionsAS/u-boot/tree/sunxi-v2017.x_mender)

# References
* Mender from scratch: https://hub.mender.io/t/mender-from-scratch/391
* Mender without Yocto: https://mender.io/blog/notes-from-a-user-on-mender-without-yocto
* U-Boot for NanoPi R1: https://wiki.friendlyarm.com/wiki/index.php/Building_U-boot_and_Linux_for_H5/H3/H2%2B
* FriendlyARM U-Boot source: https://github.com/friendlyarm/u-boot.git

# Building instructions
This is based on information from [U-boot for NanoPi R1](https://wiki.friendlyarm.com/wiki/index.php/Building_U-boot_and_Linux_for_H5/H3/H2%2B)
## Download and install tools and toolchains
* Download the cross compiler `arm-cortexa9-linux-gnueabihf-4.9.3-20160512.tar.xz` from [FriendlyArm](http://download.friendlyarm.com/nanopineo) -> [Google drive](https://drive.google.com/drive/folders/1naHlZq10whp-3SWFLa6zbp9J8MpSJ3gA)
* Install cross compiler
```
$ sudo mkdir -p /opt/FriendlyARM/toolchain
$ sudo tar xf arm-cortexa9-linux-gnueabihf-4.9.3-20160512.tar.xz -C /opt/FriendlyARM/toolchain/
$ export PATH=/opt/FriendlyARM/toolchain/4.9.3/bin:$PATH
$ export GCC_COLORS=auto
$ . ~/.bashrc
$ apt-get install swig python-dev python3-dev
$ arm-linux-gcc -v
```
Last command returns `gcc version 4.9.3 (ctng-1.21.0-229g-FA)` 

## Apply Mender patches
_**Note:** All patches have been applied to the git repo. [github.com/IOTSolutionsAS/u-boot/](https://github.com/IOTSolutionsAS/u-boot/tree/sunxi-v2017.x_mender) on branch **sunxi-v2017.x_mender**_. Please see [Patches](PATCHES.md) for details.

## Compile Mender U-Boot
```
$ git clone https://github.com/IOTSolutionsAS/u-boot.git -b sunxi-v2017.x_mender --depth 1
$ cd u-boot
$ export PATH=/opt/FriendlyARM/toolchain/4.9.3/bin:$PATH
$ make nanopi_h3_defconfig ARCH=arm CROSS_COMPILE=arm-linux-
$ make ARCH=arm CROSS_COMPILE=arm-linux-
$ make envtools ARCH=arm CROSS_COMPILE=arm-linux-
```
This will produce the files `tools/env/fw_printenv` and `u-boot-sunxi-with-spl.bin`

## Create Mender env config

Create a file `fw_env.config` containing:
```
/dev/mmcblk0    0x400000    0x4000
/dev/mmcblk0    0x800000    0x4000
```

## Result
Copy files to `mender-convert` project folder so that the custom U-Boot can be installed using `mender-convert` tool. The converter needs the following files:
```
fw_env.config
fw_printenv
u-boot-sunxi-with-spl.bin
```

## Mender integration
Note: This is only a brief description on how to create the final Mender disk image for the NanoPi R1 based on the Mender U-Boot built in [previous step][#result]. Further integration with mender follow the instructions on [Convert a Mender Debian image](https://docs.mender.io/system-updates-debian-family/convert-a-mender-debian-image)

### Files from U-Boot
Place the files from U-Boot build in [previous step][#result] in a folder `mender-convert/u-boot-files/`
```
fw_env.config
fw_printenv
u-boot-sunxi-with-spl.bin
```
### Configuration
Create a configuration file for NanoPi R1 and store it in `mender-convert/configs/nanopi_r1_config`:

```
MENDER_DEVICE_TYPE="NanoPi_R1"
MENDER_GRUB_EFI_INTEGRATION=n
MENDER_KERNEL_IMAGETYPE="zImage"
MENDER_IGNORE_MISSING_EFI_STUB=1

# 8MB alignment
MENDER_PARTITION_ALIGNMENT="8388608"
MENDER_STORAGE_TOTAL_SIZE_MB="7456"
MENDER_COMPRESS_DISK_IMAGE="gzip"

function platform_modify() {
  log_info "Copying boot-files to rootfs/boot/"
  run_and_log_cmd "sudo cp -R work/boot/* work/rootfs/boot/"

  log_info "Installing fw_printenv and fw_setenv"
  run_and_log_cmd "sudo install -m 755 u-boot-files/fw_printenv work/rootfs/sbin/fw_printenv"
  run_and_log_cmd "sudo ln -fs /sbin/fw_printenv work/rootfs/sbin/fw_setenv"

  run_and_log_cmd "sudo ln -fs /sbin/fw_printenv work/rootfs/usr/bin/fw_printenv"
  run_and_log_cmd "sudo ln -fs /sbin/fw_printenv work/rootfs/usr/bin/fw_setenv"

  run_and_log_cmd "sudo ln -fs /sbin/fw_printenv work/rootfs/usr/local/bin/fw_printenv"
  run_and_log_cmd "sudo ln -fs /sbin/fw_printenv work/rootfs/usr/local/bin/fw_setenv"

  run_and_log_cmd "sudo cp u-boot-files/fw_env.config work/rootfs/etc"
}

function platform_package() {
  log_info "Writing pre-compiled U-Boot image to ${img_path}"
  run_and_log_cmd "sudo dd if=u-boot-files/u-boot-sunxi-with-spl.bin of=${img_path} bs=1024 seek=8 conv=notrunc"
}
```

### Create image
Follow the instructions on [Convert a Mender Debian image](https://docs.mender.io/system-updates-debian-family/convert-a-mender-debian-image). This includes setting up input disk image, release name, token et. Run the `mender-convert` command with `--config configs/nanopi_r1_config` as parameter:
<br>
Example:
```
$ mender-convert --disk-image $INPUT_DISK_IMAGE --config configs/nanopi_r1_config --overlay rootfs_overlay/
```

The produced image will contain a complete image with Mender U-Boot included on the boot partiton.