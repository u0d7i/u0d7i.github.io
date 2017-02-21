---
layout: post
title: "Expandinig storage"
date: 2017-02-21 23:00
---

Never thought I could fill out 2TB of local storage (I am not a multimedia person), but I finally did.
Most of space is used by virtual machines and various lab setups. So, had to go shopping, and found out
WD Red NAS 2TB HDDs are still most silent and price/performance option, so, bought x2.

Short log of expanding existing sw RAID1 + LVM setup with these:

Create second md mirror out of two new disks:

~~~
$ sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde
~~~

in separate terminal watch it sync (you can continue while it does):

~~~
$ watch -n 2 cat /proc/mdstat
Personalities : [raid1] 
md1 : active raid1 sde[1] sdd[0]
      1953383488 blocks super 1.2 [2/2] [UU]
      [===>.................]  resync = 18.7% (365601920/1953383488) finish=1210.4min speed=21860K/sec
      bitmap: 13/15 pages [52KB], 65536KB chunk
...
~~~

add md1 as a new physycal volume for LVM:

~~~
$ sudo pvcreate /dev/md1
~~~

extend existing volume group with a new volume:

~~~
$ sudo vgextend vg1 /dev/md1
~~~

check for logical volume mountpoint usage, stop services and umount:

~~~
$ sudo lsof /data
$ sudo /etc/init.d/samba stop
$ sudo /etc/init.d/headlessVM stop
$ sudo umount /data
~~~

extend logical volume:

~~~
$ sudo lvextend -l +100%FREE /dev/vg1/data
~~~

extend partition:

~~~
$ sudo e2fsck -f /dev/vg1/data
$ sudo resize2fs /dev/vg1/data
~~~

remount (it's in /etc/fstab), and start services:

~~~
$ sudo mount /data
$ sudo /etc/init.d/samba start
$ sudo /etc/init.d/headlessVM start
~~~

that's it:

~~~
$ df -h /data
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg1-data  3.4T  1.2T  2.1T  36% /data
~~~
