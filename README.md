#HiKey CoreOS Notes

2015.09.01

##Info

These are unofficial ARM64 CoreOS disk images that I've build for the 96boards HiKey developer board.  For info on the HiKey see https://www.96boards.org/products/ce/hikey/.

##Setup

Need 2.0 GiB or larger Micro SD card for HiKey.

    cat hikey-coreos-${version}.img.xz | xz -d > hikey-coreos-${version}.img
    dd if=hikey-coreos-${version}.img of=/dev/sdX bs=4M oflag=sync

If not installed, install latest Linaro firmware/bootloader:

    https://github.com/96boards/documentation/wiki/HiKeyGettingStarted#section-41

##Grub menu

Add a CoreOS menu item like the following to the eMMC grub config file:

    hikey ~ # mkdir /tmp/grub
    hikey ~ # mount /dev/mmcblk0p6 /tmp/grub/
    hikey ~ # vi /tmp/grub/grub/grub.cfg

    menuentry 'CoreOS: SD Card' {
    set root='hd1,1'
    linux /coreos/vmlinuz-a console=ttyAMA0,115200 earlycon=pl011,0xf8015000 root=/dev/mmcblk1p9 rootwait ro efi=noruntime
    devicetree /coreos/hi6220-hikey.dtb
    }

Insert SD card into HiKey, power on.

##CoreOS Login

  user='core', password='c'
