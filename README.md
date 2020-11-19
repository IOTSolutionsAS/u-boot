# Introduction
Please see the original readme file: [README](README)

(C) Copyright 2020, [IOT Solutions AS](https://www.iotsolutions.no/), Johan Åtland Førsvoll

SPDX-License-Identifier:	GPL-2.0+

****************

This is a modified U-Boot port for devices from [FriendylElec/FriendlyArm](https://www.friendlyarm.com/) implementing patches for [Mender](https://mender.io/) so that the Mender client can run on the devices. The sorce code is a GitHub fork of the [FriendlyARM U-Boot source](https://github.com/friendlyarm/u-boot.git). 

# References
* Mender from scratch: https://hub.mender.io/t/mender-from-scratch/391
* Mender without Yocto: https://mender.io/blog/notes-from-a-user-on-mender-without-yocto
* U-Boot for NanoPi R1: https://wiki.friendlyarm.com/wiki/index.php/Building_U-boot_and_Linux_for_H5/H3/H2%2B
* FriendlyARM U-Boot source: https://github.com/friendlyarm/u-boot.git

# Supported devices:
Source code for different supported devices are placed in separate branches. 
| Device | Source branch | Mender branch |
| --- | --- | --- |
| [NanoPi R1](https://www.friendlyarm.com/index.php?route=product/product&product_id=248) | [sunxi-v2017.x](https://github.com/IOTSolutionsAS/u-boot/tree/sunxi-v2017.x) | [sunxi-v2017.x_mender](https://github.com/IOTSolutionsAS/u-boot/tree/sunxi-v2017.x_mender) | 


