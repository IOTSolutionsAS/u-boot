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
