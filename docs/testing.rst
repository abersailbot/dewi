=======
Testing
=======

Integration testing of configuration will be done using a QEMUed jessie
raspbian image.

These steps assume you have the necessary tools installed.

Fedora::

    $ sudo dnf install qemu-system-arm 

Debian/Ubuntu::

    $ sudo apt install qemu-system-arm

Building disk image
===================

Building a new disk image to use for testing is currently a manual process. In
the future, this could be automated.

1. Grab a copy of the latest 'raspbian jessie lite' image from the raspberry pi
   website (https://www.raspberrypi.org/downloads/raspbian/). I'm using
   ``2015-11-21-raspbian-jessie-lite.img`` here.

2. Use `fdisk` to find the partition boundaries in this image::

    ~/g/dewi (master) $ fdisk -l 2015-11-21-raspbian-jessie-lite.img
    Disk 2015-11-21-raspbian-jessie-lite.img: 1.4 GiB, 1458569216 bytes, 2848768 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xb3c5e39a

    Device                               Boot  Start     End Sectors  Size Id Type
    2015-11-21-raspbian-jessie-lite.img1        8192  131071  122880   60M  c W95 FAT32 (LBA)
    2015-11-21-raspbian-jessie-lite.img2      131072 2848767 2717696  1.3G 83 Linux

3. Next, we need to mount that image. To do this, we need to use the
   information ``fdisk`` told us. The start of the main partition is block
   ``131072``, and the block size is ``512``, so the total offset is
   ``131072*512`` = ``67108864``.

   Make a temporary directory to mount the image in::

       $ mkdir raspbian-jessie-mount-point

   and mount the image::

       $ sudo mount -o loop,offset=67108864 2015-11-21-raspbian-jessie-lite.img raspbian-jessie-mount-point

4. Edit ``raspbian-jessie-mount-point/etc/ld.so.preload`` and comment out all
   lines.

5. Edit ``raspbian-jessie-mount-point/etc/fstab`` and comment out any entry
   related to ``mmcblk``.

6. Unmount the image::

       $ sudo umount raspbian-jessie-mount-point

The resulting image can be reduced in size by converting to a sparse file::

    $ fallocate -v --dig-holes 2015-11-21-raspbian-jessie-lite.img
    2015-11-21-raspbian-jessie-lite.img: 494.7 MiB (518729728 bytes) converted to sparse holes.

Or compressing::

    $ xz 2015-11-21-raspbian-jessie-lite.img


Booting test machine
====================

Now that we have an image to boot, we need a kernel to run. Sadly, a modified
kernel is required, since QEMU does not support raspberry pi hardware. Luckily
the work of patching and building has already been done by someone else.
https://github.com/polaco1782/raspberry-qemu appears to work well.

Clone a copy of that repository and copy ``kernel-qemu``::

    $ git clone git@github.com:polaco1782/raspberry-qemu.git
    $ cp raspberry-qemu/kernel-qemu .

Now a boot should be possible. Run

.. code-block:: bash

    $ qemu-system-arm -kernel kernel-qemu \
        -cpu arm1176 \
        -m 256 \
        -M versatilepb \
        -no-reboot \
        -serial stdio \
        -display none \
        -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" \
        -net user,hostfwd=tcp::10022-:22 \
        -net nic -hda \
        2015-11-21-raspbian-jessie-lite.img
