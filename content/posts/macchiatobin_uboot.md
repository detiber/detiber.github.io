---
title: "Current u-boot on a Macchiatobin"
date: 2021-02-10T15:40:00-05:00
draft: false
categories:
- Tinkerbell
- Hardware
tags:
- ARM
- Macchiatobin
- u-boot
---

I recently broke out as Macchiatobin that I had laying around from a few years ago for testing arm64 support with [Tinkerbell](https://tinkerbell.org). I started with getting an [updated version of EDK2 running on it.]({{< ref "/posts/macchiatobin_uefi" >}}) While the UEFI firmware was working well, it didn't give me the ability to override the MAC addresses for the interfaces, which caused issues after the board booted into Linux, where a random MAC address was generated for each interface. As a result, I thought I would give a newer build of u-boot a shot to see if the UEFI/ACPI support provided by u-boot would work better for my use case.

## Prerequisites

You will need all the software dependencies listed in http://wiki.macchiatobin.net/tiki-index.php?page=Build+system+requirements and http://wiki.macchiatobin.net/tiki-index.php?page=Build+from+source+-+Bootloader

```sh
mkdir -p ~/mcbin
export BASEDIR=~/mcbin
```

## Linaro Toolchain Setup

```sh
mkdir -p ~/mcbin/gcc && cd ~/mcbin/gcc
wget http://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz
tar -xvf gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz
cd ../
export CROSS_COMPILE=${BASEDIR}/gcc/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-
```

## Setup the u-boot repository and build

```sh
mkdir -p ~/mcbin/Conf
git clone https://gitlab.denx.de/u-boot/u-boot
cd u-boot
git checkout v2021.01
make mvebu_mcbin-88f8040_defconfig
make -j4
export BL33=$(pwd)/u-boot.bin
```

## Setup the ATF repositories and build

```sh
cd ${BASEDIR}
git clone https://github.com/ARM-software/arm-trusted-firmware
cd arm-trusted-firmware
git checkout v2.4
cd ../

git clone https://github.com/MarvellEmbeddedProcessors/mv-ddr-marvell

git clone https://github.com/MarvellEmbeddedProcessors/binaries-marvell
cd binaries-marvell
git checkout binaries-marvell-armada-18.12

export SCP_BL2=${BASEDIR}/binaries-marvell/mrvl_scp_bl2.img

export ARCH=arm64
export CROSS_COMPILE=${BASEDIR}/gcc/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-

cd ../arm-trusted-firmware
make USE_COHERENT_MEM=0 LOG_LEVEL=20 MV_DDR_PATH=${BASEDIR}/mv-ddr-marvell PLAT=a80x0_mcbin all fip mrvl_flash
```

The resulting image will be created in `~/mcbin/arm-trusted-firmware/build/a80x0_mcbin/release/flash-image.bin`

To test image from an sd-card: http://wiki.macchiatobin.net/tiki-index.php?page=Setup+alternative+boot+sources

After booting from this u-boot image, I still hit the same issue I was hitting before related to the use of randomized MAC addresses, but the [u-boot documentation](https://www.denx.de/wiki/view/DULG/UBootPowerOn) made me realize what the underlying issue was...  I never configured MAC addresses for this machine as I should have. 

To be continued...