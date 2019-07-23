# bcm2711-kernel
Automated build of the latest 64-bit `bcm2711_defconfig` Linux kernel for the Raspberry Pi 4, updated weekly.

**Caution - this repo is work in progress, and not yet ready for production use!**

## Description

<img src="https://raw.githubusercontent.com/sakaki-/resources/master/raspberrypi/pi4/Raspberry_Pi_4_B.jpg" alt="Raspberry Pi 4 B" width="250px" align="right"/>

This project contains a weekly autobuild of the default branch (currently, `rpi-4.19.y`) of the [official Raspberry Pi Linux source tree](https://github.com/raspberrypi/linux), for the [64-bit Raspberry Pi 4](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/).

Builds are performed with the standard `bcm2711_defconfig`, with the only change being that the first 12 hex digits of the tip commit SHA1 hash plus `-p4` are appended to `CONFIG_LOCALVERSION` (with a separating hyphen) before building.

> Please note that as the purpose of this project is to provide a 'vanilla' build of the upstream `bcm2711_defconfig`, PRs requesting config changes will be rejected. Instead, please see the sister [`bcm2711-kernel-bis`](https://github.com/sakaki-/bcm2711-kernel-bis) project, which has a weekly autobuild (with versions mirroring this one, using a tweaked `bcm2711_defconfig`), and where such PRs *will* be accepted for review.

A new build tarball is automatically created and uploaded as a release asset each week (unless the tip of the default branch is unchanged from the prior week, or an error occurs during the build process).

> The default branch is used, as that is generally given most attention by RPF upstream.

As an (historical) example, on 23 July 2019, the default branch was `rpi-4.19.y` and the latest commit was `a21b98653ecf7b2f71906228d9965d8174a1c275` (the short form of which is `a21b98653ecf`). The created release was [4.19.59.20190723](https://github.com/sakaki-/bcm2711-kernel/releases/tag/4.19.59.20190723), within which the kernel tarball was `bcm2711-kernel-4.19.59.20190723.tar.xz`, and the corresponding kernel release name was `4.19.59-v8-a21b98653ecf-p4+`.

Each kernel release tarball currently provides the following files:
* `/boot/kernel8p4.img` (this is the bootable 64-bit kernel);
* `/boot/COPYING.linux` (the kernel's license file);
* `/boot/config-p4` (the configuration used to build the kernel);
* `/boot/System-p4.map` (the kernel's symbol table);
* `/boot/bcm2711-rpi-4-b.dtb` (the device tree blob; currently only one);
* `/boot/armstub8-gic.bin` (stubs required for the GIC);
* `/boot/overlays/...` (the device tree blob overlays);
* `/lib/modules/<kernel release name>/...` (the module set for the kernel).

The current kernel tarball may be downloaded from the link below (or via `wget`, or via the corresponding `bcm2711-kernel-bin` ebuild, per the [instructions following](#installation)):

Variant | Version | Most Recent Image
:--- | ---: | ---:
Kernel, dtbs, modules and GIC stub | 4.19.59.20190723 | [bcm2711-kernel-4.19.59.20190723.tar.xz](https://github.com/sakaki-/bcm2711-kernel/releases/download/4.19.59.20190723/bcm2711-kernel-4.19.59.20190723.tar.xz)

The corresponding kernel configuration (derived via `make bcm2711_defconfig`) may be viewed [here](https://github.com/sakaki-/bcm2711-kernel/blob/master/config).

> A list of all releases may be seen [here](https://github.com/sakaki-/bcm2711-kernel/releases).

## <a name="installation"></a>Installation

To deploy (assuming that your RPi4's micro SD-card's first partition is mounted as `/boot`, and you are already running a 32-bit or 64-bit RPi4-capable image, simply download, untar into the root directory, and reboot:
```console
pi64 ~ # wget -c https://github.com/sakaki-/bcm2711-kernel/releases/download/4.19.59.20190723/bcm2711-kernel-4.19.59.20190723.tar.xz
pi64 ~ # tar -xJf bcm27111-kernel-4.19.59.20190723.tar.xz -C /
pi64 ~ # sync && reboot
```

Alternatively, if you have my [rpi3 overlay](https://github.com/sakaki-/rpi3-overlay) installed (it is pre-installed on the [gentoo-on-rpi3-64bit](https://github.com/sakaki-/gentoo-on-rpi3-64bit) image), you can simply emerge the `bcm2711-kernel-bin` package (a new ebuild is automatically created to mirror each release here). For example, to install the latest available version (and start using it):
```console
pi64 ~ # emaint sync --repo rpi3
pi64 ~ # emerge -av bcm2711-kernel-bin
pi64 ~ # reboot
```

Edit `/boot/config.txt` and ensure you have a Pi-4 specific section at the bottom, as follows:
```ini
[pi4]
# memory must be clamped at 1GiB for current 64-bit Pi4 kernels
# restriction will hopefully be removed shortly
total_mem=1024
arm_64bit=1
enable_gic=1
armstub=armstub8-gic.bin
# differentiate from Pi3 64-bit kernels
kernel="kernel8p4.img"
```

> NB: these prebuilt kernels and ebuilds are provided as a convenience only. Use at your own risk! **Given that the releases in this project are created automatically, there is no guarantee that any given kernel will boot correctly.** A 64-bit kernel is necessary, but not sufficient, to boot the RPi4 in 64-bit mode; you also need the supporting firmware, configuration files, and userland software.
