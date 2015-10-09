#ARM64 HiKey CoreOS Notes

2015.10.09

##Info

The releases here are unofficial ARM64 CoreOS disk images that I've build for testing CoreOS on the 96boards HiKey developer board.  For info on the HiKey board see https://www.96boards.org/products/ce/hikey/.

##SD Card Setup

Need 2.0 GiB or larger Micro SD card for HiKey.

    cat hikey-coreos-${version}.img.xz | xz -d > hikey-coreos-${version}.img
    dd if=hikey-coreos-${version}.img of=/dev/sdX bs=4M oflag=sync

##eMMC Setup

If not installed, install Linaro UEFI firmware, Grub bootloader, and a debian system image to the HiKey eMMC.  See https://github.com/96boards/documentation/wiki/HiKeyUEFI and https://github.com/96boards/documentation/wiki/LatestSnapshots.

Download the needed files from:

    http://builds.96boards.org/snapshots/hikey/linaro/uefi/75/
    http://builds.96boards.org/snapshots/hikey/linaro/debian/354/

These versions work for me (I now have my RS232C converter connected to the HiKey UART_3/ttyAMA3).  Other versions may or may not work.  The debian install on eMMC will be for setup and recovery.

You can try my program-hikey utility if you like.  You'll need to set the builds_root variable.  This command should install everything needed to boot to debian from the eMMC:

    program-hikey --firmware --boot --system

If, when trying to program the system partition, fastboot hangs or you get errors with the UEFI firmware, try this sequence to use the fastboot loader to program the system partition first:

    program-hikey --firmware --boot --system --fastboot
    program-hikey --firmware --boot

##Grub menu

Boot the Hikey without an SD card installed or choose the grub 'eMMC' boot option.  Add a CoreOS menu item like the following to the eMMC grub config file:

    debian # vi /boot/grub/grub.cfg

    menuentry 'CoreOS: SD Card' {
    set root='hd1,1'
    linux /coreos/vmlinuz-a console=tty0 console=ttyAMA3,115200 earlycon=pl011,0xf7113000 root=/dev/mmcblk1p9 rootwait ro efi=noruntime
    devicetree /coreos/hi6220-hikey.dtb
    }

I recommend you increase the grub.cfg 'timeout' value, and maybe set the 'default' to your 'CoreOS: SD Card' entry.

You can edit the eMMC grub config from CoreOS with something like:

    coreos # mkdir /tmp/grub
    coreos # mount /dev/mmcblk0p6 /tmp/grub/
    coreos # vi /tmp/grub/grub/grub.cfg

##Start Up

Insert SD card into HiKey, power on, choose 'CoreOS: SD Card' menu item.

##CoreOS Login

  user='core', password='c'
