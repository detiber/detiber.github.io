---
title: "UEFI on a Macchiatobin"
date: 2021-02-08T12:38:46-05:00
draft: false
---

I recently broke out as Macchiatobin that I had laying around from a few years ago for testing arm64 support with [Tinkerbell](https://tinkerbell.org). Revisiting the [wiki](http://wiki.macchiatobin.net/tiki-index.php?page=Wiki+Home) it was obvious that the instructions for building the UEFI support was pretty outdated and I wanted to try and use primarily upstream sources rather than the forked and unmaintained versions used in the wiki.

## Prerequisites

You will need all the software dependencies listed in http://wiki.macchiatobin.net/tiki-index.php?page=Build+system+requirements and http://wiki.macchiatobin.net/tiki-index.php?page=Build+from+source+-+UEFI+EDK+II

```sh
mkdir -p ~/mcbin
export BASEDIR=~/mcbin
```

## Linaro Toolchain Setup

```sh
mkdir -p ~/mcbin/gcc && cd ~/mcbin/gcc
wget http://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz
tar -xvf gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz
export GCC5_AARCH64_PREFIX=${BASEDIR}/gcc/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-
cd ../
```

## Setup the edk2 repositories and build

```sh
mkdir -p ~/mcbin/Conf
git clone https://github.com/tianocore/edk2
cd edk2
git checkout edk2-stable202011
git submodule update --init
cd ../
git clone https://github.com/tianocore/edk2-platforms
cd edk2-platforms
git submodule update --init
cd ../
git clone https://github.com/tianocore/edk2-non-osi
export WORKSPACE=$PWD
export PACKAGES_PATH=$PWD/edk2:$PWD/edk2-platforms:$PWD/edk2-non-osi

make -C edk2/BaseTools
export EDK_TOOLS_PATH=~/mcbin/edk2/BaseTools

cd edk2
source edksetup.sh BaseTools
cd ../

build -a AARCH64 -t GCC5 -b RELEASE -D INCLUDE_TFTP_COMMAND -p Platform/SolidRun/Armada80x0McBin/Armada80x0McBin.dsc

export BL33=${BASEDIR}/Build/Armada80x0McBin-AARCH64/RELEASE_GCC5/FV/ARMADA_EFI.fd
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

This was able to get me to the point that I could actually PXE boot the hardware. I ran into additional issues related to the default MAC addresses on the board, and Linux assigning them random MAC addresses instead.

Next, I am going to try checking out the support for the board for UEFI booting with both mainline u-boot and ARM's fork of it here: https://github.com/ARM-software/u-boot. If neither of those work, then there is another fork of edk2/edk2-platform that may work here: https://github.com/andreiw/MacchiatoBin-edk2