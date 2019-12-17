# bcm2711-kernel
Automated build of the latest 64-bit `bcm2711_defconfig` Linux kernel for the Raspberry Pi 4, updated weekly.

## Description

<img src="https://raw.githubusercontent.com/sakaki-/resources/master/raspberrypi/pi4/Raspberry_Pi_4_B.jpg" alt="Raspberry Pi 4 B" width="250px" align="right"/>

This project contains a weekly autobuild of the default branch (currently, `rpi-4.19.y`) of the [official Raspberry Pi Linux source tree](https://github.com/raspberrypi/linux), for the [64-bit Raspberry Pi 4](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/).

Builds are performed with the standard `bcm2711_defconfig`, with the only change being that the first 12 hex digits of the tip commit SHA1 hash plus `-p4` are appended to `CONFIG_LOCALVERSION` (with a separating hyphen) before building.

> Please note that as the purpose of this project is to provide a 'vanilla' build of the upstream `bcm2711_defconfig`, PRs requesting config changes will be rejected. Instead, please see the sister [`bcm2711-kernel-bis`](https://github.com/sakaki-/bcm2711-kernel-bis) project, which has a weekly autobuild (with versions mirroring this one, using a tweaked `bcm2711_defconfig`), and where such PRs *will* be accepted for review.

A new build tarball is automatically created and uploaded as a release asset each week (unless the tip of the default branch is unchanged from the prior week, or an error occurs during the build process).

> The default branch is used, as that is generally given most attention by RPF upstream.

As an (historical) example, on 24 July 2019, the default branch was `rpi-4.19.y` and the latest commit was `689fc28a5af07cedb88760b2f35b1eb6f6a7e112` (the short form of which is `689fc28a5af0`). The created release was [4.19.59.20190724](https://github.com/sakaki-/bcm2711-kernel/releases/tag/4.19.59.20190724), within which the kernel tarball was `bcm2711-kernel-4.19.59.20190724.tar.xz`, and the corresponding kernel release name was `4.19.59-v8-689fc28a5af0-p4+`.

Each kernel release tarball currently provides the following files:
* `/boot/kernel8-p4.img` (this is the bootable 64-bit kernel);
* `/boot/COPYING.linux` (the kernel's license file);
* `/boot/config-p4` (the configuration used to build the kernel);
* `/boot/Module-p4.symvers.xz` (a table mapping exported symbols to provider, compressed);
* `/boot/System-p4.map.xz` (the kernel's symbol table, compressed);
* `/boot/bcm2711-rpi-4-b.dtb` (the device tree blob; currently only one);
* `/boot/armstub8-gic.bin` (stubs required for the GIC);
* `/boot/overlays/...` (the device tree blob overlays);
* `/lib/modules/<kernel release name>/...` (the module set for the kernel).

> The `/boot/Module-p4.symvers.xz` file is only included in more recent builds. The `/boot/System-p4.map.xz` is supplied in compressed form only in recent builds.

The current kernel tarball may be downloaded from the link below (or via `wget`, or via the corresponding `bcm2711-kernel-bin` ebuild, per the [instructions following](#installation)):

Variant | Version | Most Recent Image
:--- | ---: | ---:
Kernel, dtbs, modules and GIC stub | 4.19.89.20191217 | [bcm2711-kernel-4.19.89.20191217.tar.xz](https://github.com/sakaki-/bcm2711-kernel/releases/download/4.19.89.20191217/bcm2711-kernel-4.19.89.20191217.tar.xz)

The corresponding kernel configuration (derived via `make bcm2711_defconfig`) may be viewed [here](https://github.com/sakaki-/bcm2711-kernel/blob/master/config).

> A list of all releases may be seen [here](https://github.com/sakaki-/bcm2711-kernel/releases).

## <a name="installation"></a>Installation

You can simply untar a kernel release tarball from this project into an existing (32 or 64-bit) OS image to deploy it.

For example, to allow the current 32-bit userland Raspbian (with desktop) image to be booted under a 64-bit kernel on a Pi4, proceed as follows.

> For simplicity, I will assume you are working on a Linux PC, as root, here.

Begin by downloading and writing the Raspbian Buster image onto a _unused_ microSD card, and mounting it. Assuming the card appears as `/dev/mmcblk0` on your PC, and your OS image has two partitions (bootfs on the first, rootfs on the second, as Raspbian does), issue:

```console
linuxpc ~ # wget -cO- https://downloads.raspberrypi.org/raspbian_latest | bsdtar -xOf- > /dev/mmcblk0
linuxpc ~ # sync && partprobe /dev/mmcblk0
linuxpc ~ # mkdir -pv /mnt/piroot
linuxpc ~ # mount -v /dev/mmcblk0p2 /mnt/piroot
linuxpc ~ # mount -v /dev/mmcblk0p1 /mnt/piroot/boot
```

> NB: you **must** take care to substitute the correct path for your microSD card (which may appear as something completely different from `/dev/mmcblk0`, depending on your system) in these instructions, as the contents of the target drive will be irrevocably overwritten by the above operation.

Next, fetch the the current kernel tarball, and untar it into the mounted image. Issue:

```console
linuxpc ~ # wget -cO- https://github.com/sakaki-/bcm2711-kernel/releases/download/4.19.89.20191217/bcm2711-kernel-4.19.89.20191217.tar.xz | tar -xJf- -C /mnt/piroot/
```

Then, edit the image's `/boot/config.txt`:

```console
linuxpc ~ # nano -w /mnt/piroot/boot/config.txt
```

Modify the [pi4] section of this file (it appears near the end of the file), so it reads as follows:
```ini
[pi4]
# Enable DRM VC4 V3D driver on top of the dispmanx display stack
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
# memory must be clamped at 1GiB for current 64-bit Pi4 kernels
# restriction will hopefully be removed shortly
total_mem=1024
arm_64bit=1
enable_gic=1
armstub=armstub8-gic.bin
# differentiate from Pi3 64-bit kernels
kernel=kernel8-p4.img
```

Leave the rest of the file as-is. Save, and exit `nano`. Then unmount the image:

```console
linuxpc ~ # sync
linuxpc ~ # umount -v /mnt/piroot/{boot,}
```

If you now remove the microSD card, insert it into a RPi4, and power on, you should find it starts up under the 64-bit kernel! 

> Users of my [genpi64 overlay](https://github.com/sakaki-/genpi64-overlay) (it is pre-installed on the [gentoo-on-rpi-64bit](https://github.com/sakaki-/gentoo-on-rpi-64bit) image, for example), can simply `emerge` the `bcm2711-kernel-bin` package to deploy (a new ebuild is automatically created to mirror each release here).

> NB: these prebuilt kernels and ebuilds are provided as a convenience only. Use at your own risk! **Given that the releases in this project are created automatically, there is no guarantee that any given kernel will boot correctly.** A 64-bit kernel is necessary, but not sufficient, to boot the RPi4 in 64-bit mode; you also need the supporting firmware, configuration files, and userland software.
