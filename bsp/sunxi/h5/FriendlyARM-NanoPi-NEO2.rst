FriendlyARM-NanoPi-NEO2
#######################

This tutorial will show the details of Nanopi NEO2 board mainline support and other needed details, for more information about `hardware <http://nanopi.io/nanopi-neo2.html>`_ and `linux-sunxi <http://linux-sunxi.org/FriendlyARM_NanoPi_NEO2>`_

Hardware Access
***************

.. image:: /images/nanopi_neo2.jpeg 

Serial debug and Power connections



BSP Build
*********
Manual Build
============
Image building need host to ready with all necessary tools ready, refer here

Below are the details of Image build for Nanopi NE02 board.

ATF
---
::

        $ git clone https://github.com/apritzel/arm-trusted-firmware.git
        $ cd arm-trusted-firmware
        $ make PLAT=sun50iw1p1 bl31
        $ export BL31=/path/to/arm-trusted-firmware/build/sun50iw1p1/release/bl31.bin
        
U-Boot
------
::

        $ git clone git://git.denx.de/u-boot.git
        $ cd u-boot
        $ make nanopi_neo2_defconfig
        $ make 
        
Linux
-----
::

        $ git clone git://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git
        $ cd linux-next
        $ make mrproper
        $ ARCH=arm64 make defconfig
        $ ARCH=arm64 make -j 4 Image dtbs

Buildroot
=========
It's easy to build entire system using buildroot and mainline supported nanopi-neo2 already. See read this `readme.txt <https://git.buildroot.net/buildroot/tree/board/friendlyarm/nanopi-neo2/readme.txt>`_ for more info.

::

        $ git clone git://git.busybox.net/buildroot
        $ cd buildroot
        $ make friendlyarm_nanopi_neo2_defconfig
        $ make

Booting
*******
SD Boot
=======
U-Boot
******
USB Mass Storage gadget
=======================
We can use the board as a USB Mass Storage device:

You will be able to access all the partitions of any block device that is on the board or connected to it,

from your host PC - You will see them as /dev/sdXX, just like connecting a regular USB storage to your PC,

and you'll be able to mount them, and have full read/write access to them.

We can even use it to flash a new U-Boot, re-partition the storage, re-format it, etc.

This is especially useful for updating the internal eMMC.

To do this you need to connect a USB cable between the OTG/Client port of the board and a regular USB Host port on your PC,

and use U-Boot's ums command.

Linux
*****
USB Mass Storage gadget
=======================
Build otg mass storage as statically linked module with

`CONFIG_USB_GADGET_LEGACY=y`

`CONFIG_USB_MASS_STORAGE=y`

Append bootargs with 'g_mass_storage.removable=1 g_mass_storage.luns=1'

::

        [    1.676222] usb_phy_generic usb_phy_generic.0.auto: usb_phy_generic.0.auto supply vcc not found, using dummy regulator
        [    1.687279] musb-hdrc musb-hdrc.1.auto: MUSB HDRC host driver
        [    1.693023] musb-hdrc musb-hdrc.1.auto: new USB bus registered, assigned bus number 9
        [    1.701483] hub 9-0:1.0: USB hub found
        [    1.705290] hub 9-0:1.0: 1 port detected
        [    1.709841] Mass Storage Function, version: 2009/09/11
        [    1.715010] LUN: removable file: (no medium)
        [    1.719330] LUN: removable file: (no medium)
        [    1.723613] Number of LUNs=1
        [    1.726619] g_mass_storage gadget: Mass Storage Gadget, version: 2009/09/11
        [    1.733592] g_mass_storage gadget: userspace failed to provide iSerialNumber
        [    1.740629] g_mass_storage gadget: g_mass_storage ready
        # cat /proc/cmdline
        console=ttyS0,115200 earlyprintk root=/dev/mmcblk0p1 rootwait g_mass_storage.removable=1 g_mass_storage.luns=1
        # fdisk -l
        Disk /dev/mmcblk0: 15 GB, 15931539456 bytes, 31116288 sectors
        486192 cylinders, 4 heads, 16 sectors/track
        Units: cylinders of 64 * 512 = 32768 bytes

        Device       Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
        /dev/mmcblk0p1    320,0,1     815,3,16         20480   31116287   31095808 14.8G 83 Linux
        # echo /dev/mmcblk0 > /sys/devices/platform/soc/1c19000.usb/musb-hdrc.1.auto/gadget/lun0/file

Access the disk and write and umount
