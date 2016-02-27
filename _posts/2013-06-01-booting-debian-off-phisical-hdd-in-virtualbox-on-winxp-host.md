---
layout: post
title: "Booting Debian off phisical HDD in VirtualBox on WinXP host"
date: 2013-06-01 22:23
---

While working in windows on dualboot system, it's handy to have access to linux system, installed on the same phisical machine.
It'spossible by booting linux off the phicical hard drive in a virtual machine.
I use [VirtualBox](https://www.virtualbox.org) with raw disk image for this:

First, look how VirtualBox sees the phisical partition:

~~~
$ VBoxManage.exe internalcommands listpartitions -rawdisk '\\.\PhysicalDrive0'
Number  Type   StartCHS       EndCHS      Size (MiB)  Start (Sect)
1       0x07  0   /1  /1   1023/254/63         30718           63
2       0x07  1023/254/63  1023/254/63        149997     62910540
3       0x83  1023/254/63  1023/254/63           286    370106368
5       0x83  1023/254/63  1023/254/63         57472    370694144
~~~

Here we see partitions 1,2 as WinXP C: and D: drives, partition 3 as /boot, and partition 5 as LVM.
Now we create raw disk image with only needed linux partitions 3,5 (we don't want to mess with windows disks used by live windows host).

~~~
$ VBoxManage.exe internalcommands createrawvmdk -rawdisk '\\.\PhysicalDrive0' -partitions 3,5 -filename raw-disk.vmdk
RAW host disk access VMDK file raw-disk.vmdk created successfully.
~~~

There are 2 resulting files:<br>
raw-disk.vmdk -  text file virtual disk description<br>
raw-disk-pt.vmdk - MBR and partition data from phisical disk

~~~
$ file -b raw-disk.vmdk
ASCII text
$ file -b raw-disk-pt.vmdk | sed 's/\; /\;\n/g'
x86 boot sector;
partition 1: ID=0x7, active, starthead 1, startsector 63, 62910477 sectors;
partition 2: ID=0x7, starthead 254, startsector 62910540, 307194930 sectors;
partition 3: ID=0x83, active, starthead 254, startsector 370106368, 585728 sectors;
partition 4: ID=0x5, starthead 254, startsector 370694142, 117702658 sectors, code offset 0x1e
~~~

Now in VirtualBox we create a standard linux Debian virtual machine.
In System -> Processor tab, enable PAE/NX. Add raw-disk.vmdk as SATA disk.
Enable second network adapter, and set it attached to Host-only Adapter
(it's going to be used for network communication between guest and host, so,
 configure it via /etc/network/interfaces on guest later).

We can boot it now, but we will be facing a TrueCrypt boot loader, sitting in the MBR:

~~~
$ hexdump -C -n 32 raw-disk-pt.vmdk
00000000  ea 1e 7c 00 00 20 54 72  75 65 43 72 79 70 74 20  |..|.. TrueCrypt |
00000010  42 6f 6f 74 20 4c 6f 61  64 65 72 0d 0a 00 fa 33  |Boot Loader....3|
00000020
~~~

So, pressing escape needed to get to the grub2 in PBR of /boot partition.
To eleminate TrueCrypt boot loader when booting in virtual machine, it's possible to play around
with replacing bootstrap code in raw-disk-pt.vmdk image (like described 
[here](https://systemoverlord.com/blog/2013/04/04/booting-raw-partitions-virtualbox-grub2)),
but more simple shortcut would be to create a grub4dos boot floppy image (script and instructions
[here](http://ptspts.blogspot.se/2009/07/how-to-create-bootable-floppy-running.html)),
and chainload grub2 on /boot partition via it, by modifying menu.lst on floppy root:

~~~
timeout 0
# chainload grub2 from /boot partition (/dev/sda3)
title grub2
kernel (hd0,2)/grub/core.img
~~~

and adding resulting floppy image to our virtual machine, as primary boot device.
