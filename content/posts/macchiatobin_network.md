---
title: "Properly configuring network devices on a Macchiatobin"
date: 2021-02-11T14:27:00-05:00
draft: false
categories:
- Tinkerbell
- Hardware
tags:
- ARM
- Macchiatobin
---

I recently broke out as Macchiatobin that I had laying around from a few years ago for testing arm64 support with [Tinkerbell](https://tinkerbell.org). I started with getting an [updated version of EDK2 running on it.]({{< ref "/posts/macchiatobin_uefi" >}})  Followed by getting an updated version of [mainline u-boot running on it]({{< ref "/posts/macchiatobin_uefi" >}}). It was during the process of running a newer version of u-boot that I realized I never properly configured this board after getting it. 

https://www.denx.de/wiki/view/DULG/UBootPowerOn made me realize that I needed to configure the network MAC addresses after installing a new version of u-boot, so I thought I would dump the variables from the original u-boot image on the SPI ROM before doing just that.

```sh
mv_ddr: completed successfully
NOTICE:  Cold boot
NOTICE:  Booting Trusted Firmware
NOTICE:  BL1: v1.3(release):armada-17.10.3:4af6d73
NOTICE:  BL1: Built : 13:40:06, Oct 15 2017
NOTICE:  BL1: Booting BL2
lNOTICE:  BL2: v1.3(release):armada-17.10.3:4af6d73
NOTICE:  BL2: Built : 13:40:06, Oct 15 2017
NOTICE:  BL1: Booting BL31
lNOTICE:  MSS PM is not supported in this build
NOTICE:  BL31: v1.3(release):armada-17.10.3:4af6d73
NOTICE:  BL31: Built : 13:40:07, Oct 15 2017
l

U-Boot 2017.03-armada-17.10.1 (Oct 15 2017 - 14:03:12 +0300)

Model: MACCHIATOBin-8040
Clock:  CPU     2000 [MHz]
        DDR     1050 [MHz]
        FABRIC  1050 [MHz]
        MSS     200  [MHz]
DRAM:  16 GiB
U-Boot DT blob at : 000000007f70fc38
EEPROM configuration pattern not detected.
Comphy chip #0:
Comphy-0: PEX0         
Comphy-1: PEX0         
Comphy-2: PEX0         
Comphy-3: PEX0         
Comphy-4: SFI          
Comphy-5: SATA1        
Comphy chip #1:
Comphy-0: SGMII1        1.25 Gbps 
Comphy-1: SATA0        
Comphy-2: USB3_HOST0   
Comphy-3: SATA1        
Comphy-4: SFI          
Comphy-5: SGMII2        3.125 Gbps
UTMI PHY 0 initialized to USB Host0
SATA link 0 timeout.
SATA link 1 timeout.
AHCI 0001.0000 32 slots 2 ports 6 Gbps 0x3 impl SATA mode
flags: 64bit ncq led only pmp fbss pio slum part sxs 
Target spinup took 0 ms.
SATA link 1 timeout.
AHCI 0001.0000 32 slots 2 ports 6 Gbps 0x3 impl SATA mode
flags: 64bit ncq led only pmp fbss pio slum part sxs 
PCIE-0: Link down
MMC:   sdhci@6e0000: 0, sdhci@780000: 1
SF: Detected w25q32bv with page size 256 Bytes, erase size 4 KiB, total 4 MiB
Net:   eth0: mvpp2-0 [PRIME]mdio_register: non unique device name 'ethernet@0'
, eth1: mvpp2-3, eth2: mvpp2-4, eth3: mvpp2-5
Hit any key to stop autoboot:  0 
Marvell>> env print
baudrate=115200
bootargs=console=ttyS0,115200 root=/dev/mmcblk1p1 rw rootwait
bootcmd=run get_env; run get_images; run set_bootargs; booti $kernel_addr $ramfs_addr $fdt_addr
bootdelay=2
bootmmc=mmc dev 1; ext4load mmc 1:1 $kernel_addr $image_name;ext4load mmc 1:1 $fdt_addr $fdt_name;setenv bootargs $console root=/dev/mmcblk1p1 rw rootwait; booti $kerner
console=console=ttyS0,115200
eth1addr=00:51:82:11:22:01
eth2addr=00:51:82:11:22:02
eth3addr=00:51:82:11:22:03
ethact=mvpp2-0
ethaddr=00:51:82:11:22:00
ethprime=eth0
fdt_addr=0x4f00000
fdt_high=0xffffffffffffffff
fdt_name=boot/armada-8040-mcbin.dtb
fdtcontroladdr=7f70fc38
fileaddr=5000000
filesize=c2e600
ftd_name=boot/armada-8040-mcbin.dtb
gatewayip=10.4.50.254                                                                               
get_env=mw 0x01700000 0 0x1000; fatload mmc 1:1 0x01700000 /uenv.txt; if test "$?" = "0"; then env import -t 0x01700000; else ext4load mmc 1:1 0x01700000 /uenv.txt; if i
get_images=tftpboot $kernel_addr $image_name; tftpboot $fdt_addr $fdt_name; run get_ramfs           
get_ramfs=if test "${ramfs_name}" != "-"; then setenv ramfs_addr 0x8000000; tftpboot $ramfs_addr $ramfs_name; else setenv ramfs_addr -;fi
hostname=marvell                                                                                    
image_name=boot/Image
initrd_addr=0xa00000
initrd_size=0x2000000
ipaddr=0.0.0.0
kernel_addr=0x5000000
loadaddr=0x5000000
netdev=eth0
netmask=255.255.255.0
ramfs_addr=-
ramfs_name=-
root=root=/dev/nfs rw
rootpath=/srv/nfs/
serverip=0.0.0.0
set_bootargs=setenv bootargs $console $root ip=$ipaddr:$serverip:$gatewayip:$netmask:$hostname:$netdev:none nfsroot=$serverip:$rootpath $extra_params
stderr=serial@512000
stdin=serial@512000
stdout=serial@512000

Environment size: 2193/65532 bytes
```

Digging into more u-boot documentation, I found
https://gitlab.denx.de/u-boot/u-boot/-/blob/c7182c02cefb11431a79a8abb4d8a821e4a478b5/doc/README.marvell, which points out that the MAC addresses configured for the u-boot in SPI where hardcoded defaults. Looks like it's time to boot back into the updated u-boot from my SDCard and set these to some better values.

To try and avoid any potential conflicts with real hardware, I need to create some Unicast Local MAC addresses. More info on MAC Addresses [here](https://en.wikipedia.org/wiki/MAC_address), but basically I need to ensure that the second least significant bit of the first octet is 1 and the least significant bit of the first octet is 0.

Since I'm going to use this for demos, and having something memorable and recognizable will help, I decided to use be:de:ad:be:ef:00 through be:de:ad:be:ef:03 as the MAC addresses.

```sh
setenv ethaddr be:de:ad:be:ef:00
setenv eth1addr be:de:ad:be:ef:01
setenv eth2addr be:de:ad:be:ef:02
setenv eth3addr be:de:ad:be:ef:03
saveenv
```
