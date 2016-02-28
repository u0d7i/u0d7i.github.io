---
layout: post
title: "PC setup: XP/Debian dualboot with full disk encryption (C:/D: TrueCrypt + dm-crypt)"
date: 2013-04-27 19:53
---

Got Samsung NC110 netbook, decommissioned at work, perfectly fitting my current reqs for a mobile workplace.
Onboard Win7 Starter too heavy for netbooks anyway, but I still need windows for some tasks.
Decided to wipe it out, and replace with XP SP3 (surprisingly legal one, hologrammic hardcopy found in legacy box),
dualbooted with Debian Linux. Disk encryption is mandatory for both OS's.

Netbook lacks CD/DVD, and I don't like playing with this kind of media, so first I needed bootable USB stick,
with an ability to direct boot installation and livecd iso images of any kind, just by dropping them on USB flash.
RMPrepUSB and Easy2Boot from [rmprepusb.com](http://www.rmprepusb.com) are one of the options.
After creating bootable USB flash we are back to our setup.

First install Windows XP (vanilla XP installs without any problems). Create 2 ntfs partitions for c: and d: disks, leaving space for linux install.
Then download and install hardware-specific drivers for XP from [Samsung site](http://www.samsung.com/us/support/owners/product/NP-NC110-A02US)
Now Windows has 2 partitions - disk C: for system, and D: for data.

Install [TrueCrypt](http://www.truecrypt.org/). Encrypt System on C: disk - In Start -> run, type:

~~~
"C:\Program Files\TrueCrypt\TrueCrypt Format.exe" /n
~~~

(cmdline option to skip rescue disk iso burning verification later)
 
From wizard: "Encrypt the system partition or entire system drive" -> "Normal" ->
"Encrypt the Windows system partition" -> "Multi-boot" -> "Warning:Yes" -> "BootDrive:Yes" ->
"Number of System Drives: 1" -> "Non-Windows Boot Loader: No" ->
"Encryption/Hash Alhorythms (by benchmark/default)" -> "Password (choose a goooood one)" ->
"Collect Random Data" -> "Keys Generated" -> "Rescue Disk (generate iso and copy to usb flash)" ->
"Wipe Mode(choose)" -> "System Encryption Pretest" (restart Computer) -> (Type in your passphrase on boot) ->
"Pretest Completed: Encrypt" -> "Encryption (drink some coffee)" -> "Finish". 

Reboot to verify.

Encrypt empty D: disk for data: Doble click on TrueCrypt tray icon, 
"Create volume" ->"Encrypt a non-system partition/drive" -> Standard TrueCrypt volume ->
"Select Device" -> "Drive D:" -> "Are you sure? Yes" -> "Next" -> "Create encryptrd volume and format i it" ->
"Encryption options: same as for drive C:" -> "Volume Password: same as for drive C" ->
"Filesystem NTFS / Cluster Default / Quick format" -> "Format" -> "Are you sure: Yes" -> "OK" -> "Exit"

Right click "My Computer" -> "Manage" -> "Storage" -> "Disk Management"

Right vlick on "D:" -> "Change Drive Letter and Paths" -> "Remove" -> "Yes"

Doble click on TrueCrypt tray icon, select "D:" -> "Select device" -> "\Device'Harddisk0\Partition2" (ex. D: drive) -> "Mount"

From TrueCrypt menu -> Favorities -> "Add Mounted Volume to System Favorities..." -> "D:" ->
check "Mount system favorite volumes when Windows starts" and "Allow only administrators..." -> "Ok" -> Exit

Reboot to verify both C: and D: encrypted volumes mounts on boot with single preboot auth password.

Install Debian 7 "wheezy" - since at the time of wtitting it's yet in testing, and netbook network hardware
requires non-free firmware components, download [custom netinst iso](http://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/wheezy_di_rc1/i386/iso-cd/firmware-wheezy-DI-rc1-i386-netinst.iso) and boot install from it. In "Partition disks" menu choose "Manual":

"FREE SPACE" -> "Create a new partition" -> "size: 300 MB" -> "Primary" (TrueCrypt in MBR needs it to be primary) ->
"Beginning" -> "Use as: Ext4, Mount point /boot, Label: boot, Bootable flag: on" -> Done.

"FREE SPACE" -> "Create a new partition" -> "size: agree with all that left" -> Logical ->
"Use as: physical volume for encryption" -> "Erase data: no, others: agree with defaults" -> Done

"Configure encrypted volumes" -> "Write changes" -> "Yes" -> "Create encrypted volumes" ->
choose one with "crypto" in it -> Finish -> Choose passphrase
Select "Encrypted Volume #1 -> "Use as: physical volume for LVM" -> Done

"Configure the Logical Volume Manager" -> "Create volume group" -> "name: volumes"-> select mapper crypt device ->
"Create logical volume" -> "volumes" -> "name: swap" -> "size: 2G" -> "Create logical volume" -> "volumes" ->
"name: root" -> "size: agree with the rest" -> Finish"

"LV root #1" -> "Use as: Ext4, Mount point: /" -> Done

"LV swap #1" -> "Use as: swap area" -> Done

Finish partitioning and write changes to disk. Then istall as usual, with "SSH server, Laptop and Standard system utilities"
Software selection. When it offers to instal GRUB to MBR, say No, and install it to PBR of the "boot" partition
(/dev/sda3 in our case). After reboot, press escape on TrueCrypt boot loader pre-boot auth, and it will continue to GRUB.
Boot linux with dm-crypt passphrase. As root start fdisk, and toggle boot flag on windows C: partiion (/dev/sda1):

~~~
# fdisk /dev/sda
Command (m for help): a
Partition number (1-5): 1
Command (m for help): w
~~~

So, now we have 2 bootable partitions: windows c: and linux "/boot".
On boot enter TrueCrypt passphrase if you want to boot Windows, or press Escape to GRUB/linux.
