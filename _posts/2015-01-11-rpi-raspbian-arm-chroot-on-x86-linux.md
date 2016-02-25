---
layout: post
title: "RPi (Raspbian arm) chroot on x86 Linux"
date: 2015-01-11 23:05
---

Different architecture system chroot using qemu-user-static and [binfmt](http://en.wikipedia.org/wiki/Binfmt_misc)
hooks is a common way to do (faster) native development of embedded system on desktop/server grade equipment.
Did it several times with several platforms, should be the same with RPi.
Raspbian is Debian, host machine is also Debian. We can use 2 scenarios:

### Official Raspbian image approach

I'm on a x86_64 Debian Wheezy

~~~
# uname -a
Linux lab0 3.14.16 #1 SMP Sat Aug 9 23:54:07 EEST 2014 x86_64 GNU/Linux
~~~

Instal prerequisites:

~~~
# apt-get install qemu-user-static kpartx
~~~

binfmt-support is installed as a dependancy to qemu-user-static.<br/>
Get the latest official raspbian image:

~~~
# wget -c http://downloads.raspberrypi.org/raspbian/images/raspbian-2014-12-25/2014-12-24-wheezy-raspbian.zip
# unzip 2014-12-24-wheezy-raspbian.zip
~~~

Image has multiple partitions in it, so we need kpartx to operate them:

~~~
# kpartx -a -v 2014-12-24-wheezy-raspbian.img
add map loop0p1 (254:7): 0 114688 linear /dev/loop0 8192
add map loop0p2 (254:8): 0 6277120 linear /dev/loop0 122880
~~~

Create chroot directory:

~~~
# mkdir -p /data/lab/pi/emu/wheezy-raspbian-img
~~~

Mount image partitions:

~~~
# mount /dev/mapper/loop0p2 /data/lab/pi/emu/wheezy-raspbian-img/
# mount /dev/mapper/loop0p1 /data/lab/pi/emu/wheezy-raspbian-img/boot/
~~~

Mount service partitions from host system into chroot directory:

~~~
# for d in dev proc sys dev/pts; do mount -o bind /${d} /data/lab/pi/emu/wheezy-raspbian-img/${d}; done
~~~

Create a list of relevant mounted filesystems (so we can use `df` for example):

~~~
# egrep "rootfs|boot" /etc/mtab | sed 's/\/data\/lab\/pi\/emu\/wheezy-raspbian-img//' > /data/lab/pi/emu/wheezy-raspbian-img/etc/mtab
~~~

Chroot into it:

~~~
# chroot /data/lab/pi/emu/wheezy-raspbian-img/
~~~

Validate setup (note the architecture change):

~~~
# uname -a
Linux lab0 3.14.16 #1 SMP Sat Aug 9 23:54:07 EEST 2014 armv7l GNU/Linux
~~~

From within chroot try some stuff:

~~~
# apt-get update
qemu: uncaught target signal 4 (Illegal instruction) - core dumped
Illegal instruction
~~~

According [this](http://www.raspberrypi.org/forums/viewtopic.php?p=298537)
and [this](http://xecdesign.com/qemu-emulating-raspberry-pi-the-easy-way)
culprit is /etc/ld.so.preload . Comment out `"/usr/lib/arm-linux-gnueabihf/libcofi_rpi.so"` line:

~~~
#/usr/lib/arm-linux-gnueabihf/libcofi_rpi.so
~~~

After that everyting works. But we are limited to the partition size of the image:

~~~
# df -h
Filesystem           Size  Used Avail Use% Mounted on
rootfs               2.9G  2.2G  533M  81% /
/dev/mapper/loop0p1   56M  9.7M   47M  18% /boot
~~~

We want to resize/expand rootfs. First - cleanup:

~~~
# umount /data/lab/pi/emu/wheezy-raspbian-img/{proc,sys,dev/pts,dev,boot,.}
# kpartx -d 2014-12-24-wheezy-raspbian.img
~~~

Add 5 more gigs to the image (should be enough):

~~~
# qemu-img resize 2014-12-24-wheezy-raspbian.img +5G
Image resized.
~~~

Rotfs is ext4, not supported by the parted at the moment, so, the only way to resize partition
is delete it via fdisk, and reate a new one with the same start boundaries,
but different size. We can run fdisk directly on an image:

~~~
# fdisk 2014-12-24-wheezy-raspbian.img

Command (m for help): p

Disk 2014-12-24-wheezy-raspbian.img: 8645 MB, 8645509120 bytes
255 heads, 63 sectors/track, 1051 cylinders, total 16885760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000c45c9

                         Device Boot      Start         End      Blocks   Id  System
2014-12-24-wheezy-raspbian.img1            8192      122879       57344    c  W95 FAT32 (LBA)
2014-12-24-wheezy-raspbian.img2          122880     6399999     3138560   83  Linux

Command (m for help): d
Partition number (1-4): 2

Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (1-4, default 2):
Using default value 2
First sector (63-16885759, default 63): 122880
Last sector, +sectors or +size{K,M,G} (122880-16885759, default 16885759):
Using default value 16885759

Command (m for help): p

Disk 2014-12-24-wheezy-raspbian.img: 8645 MB, 8645509120 bytes
255 heads, 63 sectors/track, 1051 cylinders, total 16885760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000c45c9

                         Device Boot      Start         End      Blocks   Id  System
2014-12-24-wheezy-raspbian.img1            8192      122879       57344    c  W95 FAT32 (LBA)
Partition 1 does not end on cylinder boundary.
2014-12-24-wheezy-raspbian.img2          122880    16885759     8381440   83  Linux

Command (m for help): w
The partition table has been altered!

Syncing disks.
~~~

Resize the actuall filesystem (you need to map it to the loop device again):

~~~
# kpartx -a -v 2014-12-24-wheezy-raspbian.img
# e2fsck -f /dev/mapper/loop0p2
# resize2fs /dev/mapper/loop0p2
~~~

Mount it back and use as per above.

### Debootstrap approach

Raspbian is Debian, and we can use debootstrap to create a directory tree for chroot - in this case we are not limited to any image size.

~~~
# apt-get install debootstrap
# mkdir raspbian-armhf
# wget http://archive.raspbian.org/raspbian.public.key -O - | apt-key add -
# qemu-debootstrap --keyring /etc/apt/trusted.gpg --arch armhf wheezy raspbian-armhf http://archive.raspbian.org/raspbian
~~~

The rest is the same as above. Mount:

~~~
# for d in dev proc sys dev/pts; do mount -o bind /${d} /data/lab/pi/emu/raspbian-armhf/${d}; done
~~~

Chroot:

~~~
# chroot raspbian-armhf
~~~

Validate:

~~~
# uname -a
Linux dom0 3.14.16 #1 SMP Sat Aug 9 23:54:07 EEST 2014 armv7l GNU/Linux
~~~

Tune:

~~~
# echo "deb http://mirrordirector.raspbian.org/raspbian/ wheezy main contrib non-free rpi" > /etc/apt/sources.list
# wget http://archive.raspbian.org/raspbian.public.key -O - | apt-key add -
# apt-get update
# apt-get upgrade
~~~
