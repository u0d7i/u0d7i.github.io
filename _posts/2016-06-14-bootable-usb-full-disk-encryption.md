---
layout: post
title: "Bootable USB linux with full disk encryption"
date: 2016-06-14 10:22
---

Needed a single OS environment, being able to boot on x64 corporate laptop, x86 personal netbook,
and VirtualBox both on corporate Windows and home Linux.

Below is a log of installing i386 Debian jessie on fully encrypted (LVM on LUKS including /boot).

32G Kingston DataTraveler Micro 3.1 USB flash drive happened to be the media.

Installation took place on amd64 Debian jessie.


~~~
$ sudo su -
# lsusb  | grep Kingston
Bus 004 Device 003: ID 0951:1666 Kingston Technology

# readlink -e /dev/disk/by-id/usb-Kingston_DataTraveler_3*:0
/dev/sdd
~~~

Fill the media with random data. Simple 

~~~
# dd if=/dev/urandom of=/dev/sdd
~~~

would take too long, so can do a trick spotted [here](https://www.linux.com/blog/how-full-encrypt-your-linux-system-lvm-luks) -
luks-encrypting it, and then filling it with zeroes, using LUKS as an entropy source which is much faster, and wiping only
LUKS header afterwards.

~~~
# dd bs=512 count=4 if=/dev/urandom of=/tmp/keyfile iflag=fullblock
# cryptsetup luksFormat -q -v -d /tmp/keyfile /dev/sdd
# cryptsetup luksOpen -d /tmp/keyfile /dev/sdd sdd_crypt
# dd if=/dev/zero of=/dev/mapper/sdd_crypt
# cryptsetup luksClose sdd_crypt
# dd if=/dev/urandom of=/dev/sdd bs=512 count=20480
# rm /tmp/keyfile
~~~

It took less than an hour with 10.0 MB/s average write speed.
Older dd is missing 'status=progress' option, check dd status with 'pkill -USR1 ^dd'.

Now create  LUKS volume on USB, LVM on it, swap and root partitions on LVM:

~~~
# parted -s /dev/sdd mklabel msdos
# parted -s /dev/sdd mkpart primary 0% 100%
# cryptsetup luksFormat /dev/sdd1
# cryptsetup luksOpen /dev/sdd1 lvm
# pvcreate /dev/mapper/lvm
# vgcreate vg /dev/mapper/lvm
# lvcreate -L 1G vg -n swap
# lvcreate -l +100%FREE vg -n root
# mkswap -L swap /dev/mapper/vg-swap
# mkfs.ext4 /dev/mapper/vg-root
~~~

After mounting root partition, final hierarchy looks like this (note UUID's for future use):

~~~
# mount /dev/mapper/vg-root /mnt/
# lsblk /dev/sdd -o NAME,RM,SIZE,TYPE,MOUNTPOINT,UUID
NAME          RM  SIZE TYPE  MOUNTPOINT UUID
sdd            1 28.9G disk             
└─sdd1         1 28.9G part             08b3e5cd-f54f-4e06-929b-63b3083d4e5d
  └─lvm        0 28.9G crypt            2EZ3oh-4HdZ-jEO1-Y2eL-3kWF-3Arg-wbLNhn
    ├─vg-swap  0    1G lvm              9ac82845-e844-48cd-b792-4f9899c109f1 
    └─vg-root  0 27.9G lvm   /mnt       e5a91b19-fb4c-4e0a-b1f0-55e7207b1bcc
~~~

Install debian into /mnt via debootstrap as described [here](install debian as described in https://www.debian.org/releases/stable/i386/apds03.html):

~~~
# apt-get install debootstrap
# debootstrap --arch i386 jessie /mnt http://ftp.debian.org/debian
~~~

Do some magic:

~~~
# cd /mnt/
# mount -t proc none proc/
# mount -t sysfs sys sys/
# LANG=C.UTF-8 chroot . /bin/bash
# apt-get install makedev
# cd /dev/
# MAKEDEV generic
# exit
# mount -o bind /dev dev/
# LANG=C.UTF-8 chroot . /bin/bash
~~~

~~~
# echo debian > /etc/hostname
# passwd
# vi /etc/apt/sources.list
deb http://ftp.debian.org/debian jessie main contrib non-free
deb http://security.debian.org/ jessie/updates main contrib non-free

# apt-get update
# apt-get install locales linux-image-686-pae firmware-linux cryptsetup lvm2
# /etc/init.d/irqbalance stop
# dpkg-reconfigure locales
# dpkg-reconfigure tzdata
~~~

~~~
# vi /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system>                             <mount point> <type>      <options>         <dump> <pass>
# /dev/mapper/vg-root
UUID=e5a91b19-fb4c-4e0a-b1f0-55e7207b1bcc /             ext4        errors=remount-ro 0       1
# /dev/mapper/vg-swap
UUID=9ac82845-e844-48cd-b792-4f9899c109f1 none          swap        sw                0       0
/dev/sr0                                  /media/cdrom0 udf,iso9660 user,noauto       0       0
~~~

~~~
# vi /etc/adjtime 
0.0 0 0.0
0
UTC
~~~

Install, configure and install (heh, yes) grub into MBR of USB device:

~~~
# apt-get install grub-pc
# echo 'GRUB_ENABLE_CRYPTODISK=y' >> /etc/default/grub
# echo 'GRUB_CMDLINE_LINUX="cryptdevice=/dev/disk/by-uuid/08b3e5cd-f54f-4e06-929b-63b3083d4e5d:lvm"' >> /etc/default/grub
# grub-mkconfig -o /boot/grub/grub.cfg
# grub-install /dev/sdd
~~~

Setup initrd:

~~~
# dd if=/dev/urandom of=/luks_keyfile.bin bs=512 count=4
# chmod 000 /luks_keyfile.bin
# cryptsetup luksAddKey /dev/sdd1 /luks_keyfile.bin 
Enter any passphrase: (existing pass here)

# vi /etc/initramfs-tools/hooks/crypto_keyfile
#!/bin/sh
cp /luks_keyfile.bin "${DESTDIR}"

# chmod +x /etc/initramfs-tools/hooks/crypto_keyfile

# vi /etc/crypttab 
lvm UUID=08b3e5cd-f54f-4e06-929b-63b3083d4e5d /luks_keyfile.bin luks,keyscript=/bin/cat

# update-initramfs -u
~~~

Cleanup:

~~~
# exit
# umount ./{dev,sys,proc}
# cd
# umount /mnt
# vgchange -a n vg
# cryptsetup luksClose lvm
~~~

References: [1](http://www.pavelkogan.com/2014/05/23/luks-full-disk-encryption/), [2](http://www.pavelkogan.com/2015/01/25/linux-mint-encryption/), [3](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system).
