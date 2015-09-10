#ARM64 HiKey CoreOS Notes

2015.09.10

##Info

The releases here are unofficial ARM64 CoreOS disk images that I've build for testing CoreOS on the 96boards HiKey developer board.  For info on the HiKey board see https://www.96boards.org/products/ce/hikey/.

##SD Card Setup

Need 2.0 GiB or larger Micro SD card for HiKey.

    cat hikey-coreos-${version}.img.xz | xz -d > hikey-coreos-${version}.img
    dd if=hikey-coreos-${version}.img of=/dev/sdX bs=4M oflag=sync

##eMMC Setup

If not installed, install Linaro UEFI firmware and Grub bootloader to the HiKey.  See https://github.com/96boards/documentation/wiki/HiKeyUEFI.

The versions below work for me (I use uart0).  Other versions may or may not work.  I tried the uefi/63 build and it could boot OK, but I think grub is on uart3.

You can try my program-hikey utility which does the same as the commands below.

If, when trying to program the system partition, fastboot hangs or you get errors with the UEFI firmware, try this sequence to use the fastboot loader to program the system partition first:

    program-hikey --firmware --boot --system --fastboot
    program-hikey --firmware --boot --uefi

Download the needed files from http://builds.96boards.org/.

With pins 3-4 jumpered:

    python hisi-idt.py -d /dev/ttyUSB1 --img1=builds.96boards.org/snapshots/hikey/linaro/uefi/46/l-loader.bin
    fastboot flash ptable builds.96boards.org/snapshots/hikey/linaro/uefi/46/ptable-linux.img
    fastboot flash fastboot builds.96boards.org/snapshots/hikey/linaro/uefi/46/fip.bin
    fastboot flash nvme builds.96boards.org/releases/hikey/linaro/binaries/15.06/nvme.img
    fastboot reboot
    
With pins 5-6 jumpered:

    fastboot flash boot builds.96boards.org/snapshots/hikey/linaro/debian/339/boot-fat.uefi.img
    fastboot flash system builds.96boards.org/releases/hikey/linaro/debian/15.06/hikey-jessie_developer_20150701-323.emmc.img
    fastboot reboot

##Grub menu

Boot the Hikey without an SD card installed or choose the grub 'eMMC' boot option.  Add a CoreOS menu item like the following to the eMMC grub config file:

    hikey ~ # mkdir /tmp/grub
    hikey ~ # mount /dev/mmcblk0p6 /tmp/grub/
    hikey ~ # vi /tmp/grub/grub/grub.cfg

    menuentry 'CoreOS: SD Card' {
    set root='hd1,1'
    linux /coreos/vmlinuz-a console=ttyAMA0,115200 console=ttyAMA3,115200 earlycon=pl011,0xf8015000 root=/dev/mmcblk1p9 rootwait ro efi=noruntime
    devicetree /coreos/hi6220-hikey.dtb
    }

I recommend you increase the grub.cfg 'timeout' value, and maybe set the 'default' to your 'CoreOS: SD Card' entry.

##Start Up

Insert SD card into HiKey, power on, choose 'CoreOS: SD Card' menu item.

##CoreOS Login

  user='core', password='c'
