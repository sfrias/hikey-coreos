# CoreOS ARM64 Notes

2016.04.07

## Contents

[Info](#info)<br>
[License](#license)<br>
[QEMU](#qemu)<br>
[Hikey Board](#hikey-board)<br>
[APM X-Gene Mustang](#apm-x-gene-mustang)<br>
[Huawei D02 Developer Board](#huawei-d02-developer-board)<br>
[General Notes](#general-notes)<br>
[Add A UEFI Boot Option](#add-a-uefi-boot-option)<br>

## Info

The releases here are unofficial CoreOS ARM64 disk images that I've build for
testing CoreOS on various ARM64 platforms.

You can always find the latest version of this document, and some other useful
technical documents at https://github.com/glevand/hikey-coreos/

Please send comments to <geoff@infradead.org>.  I can also be reached on
freenode/#coreos as geoff-

## License

Permission is granted to copy, distribute and/or modify this document under the
terms of the
[GNU Free Documentation License, Version 1.3](http://www.gnu.org/licenses/fdl-1.3.html)
published by the Free Software Foundation; with no Invariant Sections, no
Front-Cover Texts, and no Back-Cover Texts. A copy of the license is included in
the section entitled "GNU Free Documentation License".

## QEMU

The CoreOS SDK includes qemu-system-aarch64 that you can use.  Newer host
distros should have a packaged qemu-system-aarch64.  Debian or
Ubuntu users can install it with ```apt-get install qemu-system-arm```.

To run CoreOS with QEMU you'll need UEFI firmware built for qemu-system-aarch64.
I use the binary releases provided by Linaro:

    wget http://releases.linaro.org/components/kernel/uefi-linaro/latest/release/qemu64/QEMU_EFI.fd

To run single node tests I run QEMU as shown below.  This should work with
either a pre-built ARM64 CoreOS image or the coreos_developer_image.bin build
with the CoreOS SDK using ```build_image board=arm64-usr dev```:

    qemu-system-aarch64 -machine virt -cpu cortex-a57 -machine type=virt -nographic -m 2048 \
      -bios QEMU_EFI.fd \
      -drive if=none,id=blk,file=arm64_coreos_developer_image_${version}.bin \
      -device virtio-blk-device,drive=blk \
      -net user,hostfwd=tcp::10022-:22,hostname=core-arm64 \
      -device virtio-net-device,vlan=0

SSH port 22 of the ARM64 VM will be forwarded to port 10022 on the host:
```ssh -p 10022 core@localhost```.

To run the ARM64 VM as a node in a cluster use QEMU's bridged networking:

    qemu-system-aarch64 -machine virt -cpu cortex-a57 -machine type=virt -nographic -m 2048 \
      -bios QEMU_EFI.fd \
      -drive if=none,id=blk,file=arm64_coreos_developer_image_${version}.bin \
      -device virtio-blk-device,drive=blk \
      -netdev bridge,br=br0,id=net0 \
      -device virtio-net-device,netdev=net0,id=nic0

See the [QEMU docs](http://wiki.qemu.org/Manual) on how to setup the networking
to hook into your system.

## Hikey Board

The Hikey board is supported in ```hikey-coreos-8``` and later releses.  For
info on the HiKey board see https://www.96boards.org/products/ce/hikey/.

### SD Card Setup

You'll need a 6.0 GiB or larger Micro SD card for the HiKey.

    cat arm64_coreos_developer_image_${version}.bin.xz | xz -d > arm64_coreos_developer_image_${version}.bin
    dd if=arm64_coreos_developer_image_${version}.bin of=/dev/sdX bs=4M oflag=sync

### eMMC Setup

If not installed, install UEFI firmware, Grub bootloader, and a Debian system
image to the HiKey eMMC.  The Debian install on eMMC will be for setup and
recovery.  See
[Hikey UEFI wiki](https://github.com/96boards/documentation/wiki/HiKeyUEFI) and
[96boards Snapshots](https://github.com/96boards/documentation/wiki/LatestSnapshots).

Download the needed files from:

    http://builds.96boards.org

To program the Hikey firmware you will need the Android fastboot program
installed on your host.  Debian or Ubuntu users can install with
```apt-get install android-tools-fastboot```.

Connect a Micro-USB 'SYNC' cable to the Hikey's USB OTG.

The UEFI and Debian consoles will be on the expansion header UART (ttyAMA3) at
115200 baud.

You can try my program-hikey utility included in this release if you like.
You may need to set the ```DEVICE``` and/or ```DL_ROOT``` variables.  These
commands should install everything needed to boot to Debian from the eMMC:

    program-hikey --download
    program-hikey --firmware --boot --system
    program-hikey --autoboot=n

### UEFI Boot Entry

To permanently add a UEFI boot menu entry you will need to turn off the Hikey's
autoboot feature.  This can be done with the ```--autoboot=n``` option of my
program-hikey utility, or with a ```fastboot oem autoboot 0``` command.  Note
that if autoboot is not turned off you can still add a temporary entry that will
boot CoreOS, but that entry will be gone on re-boot.  See
[Hikey UEFI wiki](https://github.com/96boards/documentation/wiki/HiKeyUEFI)
for more info.

To add a boot menu entry, boot with the CoreOS storage device installed then
follow [Add A UEFI Boot Option](#add-a-uefi-boot-option).

### Start Up

Navigate back to the main UEFI boot menu and choose the new ```CoreOS``` menu
item.  After some loading and a few 'getenv' error messages the CoreOS grub menu
should appear.  Choose ```96boards hikey```. After some delay at a blank screen
while loading the system should come up with a CoreOS system console on the
expansion header UART at ttyAMA3,115200.

## APM X-Gene Mustang

The APM Mustang is supported in ```hikey-coreos-9``` and later releses.

### Storage Setup

Write the CoreOS ARM64 disk image to either the Mustang hard disk using a USB to SATA
adapter on your host, or to a USB storage device.  Booting CoreOS from the
Mustang SD Card currently does not work:

    cat arm64_coreos_developer_image_${version}.bin.xz | xz -d > arm64_coreos_developer_image_${version}.bin
    dd if=arm64_coreos_developer_image_${version}.bin of=/dev/sdX

### UEFI Setup

Install UEFI firmware version 1.15.10 available from https://myapm.apm.com.
Newer firmware versions could also work.  For more info on installation see the
Mustang software guide that comes with the firmware update.  Older firmware
released by the Fedora project will not work with these CoreOS images.

    UpgradeFirmware.efi apm_upgrade_tianocore.cmd

To add a boot menu entry, boot with the CoreOS storage device installed then
follow [Add A UEFI Boot Option](#add-a-uefi-boot-option).

### Start Up

Navigate back to the main UEFI boot menu and choose the new ```CoreOS``` menu
item.  After some loading and a few 'getenv' error messages the CoreOS grub menu
should appear.  Choose ```APM Mustang```.  After some delay at a blank screen
while loading the system should come up with a CoreOS system console on the
UART at ttyS0,115200n8.

## Huawei D02 Developer Board

The D02 Developer board is supported in ```hikey-coreos-10``` and later releses.
For info on the D02 board see http://open-estuary.org/.

Update the D02 system flash to the latest estuary release with commands similar
to these:

    ifconfig -s eth0 192.168.10.XXX 255.255.255.0 192.168.10.YYY
    provision 192.168.10.YYY -u anonymous -p guest -f latest/UEFI_D02.fd -a 100000
    spiwfmem 100000 0000000 300000
    provision 192.168.10.YYY -u anonymous -p guest -f latest/CH02TEVBC_V03.bin -a 100000
    updatecpld 100000
    provision 192.168.10.YYY -u anonymous -p guest -f latest/hip05-d02.dtb -a 100000
    spiwfmem 100000 300000 100000

Write the CoreOS ARM64 disk image to a 8 GiB or larger USB storage device, or a
Seagate ST1000LM024 hard disk using a USB to SATA adapter on your host:

    cat arm64_coreos_developer_image_${version}.bin.xz | xz -d > arm64_coreos_developer_image_${version}.bin
    dd if=arm64_coreos_developer_image_${version}.bin of=/dev/sdX

To add a boot menu entry, boot with the CoreOS storage device installed then
follow [Add A UEFI Boot Option](#add-a-uefi-boot-option).

### Start Up

Navigate back to the main UEFI boot menu and choose the new ```CoreOS``` menu
item.  After some loading and a few 'getenv' error messages the CoreOS grub menu
should appear.  Choose ```Huawei D02```.  After some delay at a blank screen
while loading the system should come up with a CoreOS system console on the
UART at ttyS0,115200n8.

## General Notes

### CoreOS Login

  user='core', password='c'

### Add A UEFI Boot Option

Reboot into UEFI with the CoreOS storage device installed and stop the default
boot selection timer with a keystroke.  Select the menu options
```Boot Manager/Add Boot Device Entry/EFI-SYSTEM```

Set ```File path of the EFI Application: efi\boot\bootaa64.efi```.  Note that
backslash ```\``` must be used here.

Set ```EFI Application? y```.

Set ```OS loader? y```.

Set ```Arguments:``` (empty).

Set ```Description: CoreOS```, or whatever you like.

### Debugging

To inspect the CoreOS disk image use somthing like:

    sudo kpartx -v -a -s arm64_coreos_developer_image_${version}.bin
    sudo mount /dev/mapper/loopXp1 /tmp/boot
    sudo mount /dev/mapper/loopXp3 /tmp/usr
    sudo mount /dev/mapper/loopXp9 /tmp/rootfs

### Building

See The README at https://github.com/glevand/coreos--manifest for info on
building CoreOS with my development repositories.
