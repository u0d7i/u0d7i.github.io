---
layout: post
title: "HTC One (M8) dual sim"
date: 2015-04-14 00:25
---
I own Nokia E71 as my primary phone since 2009.
In 6 years I've replaced LCD twice ([once in a <s>field</s> pub](http://s21.postimg.org/idfw0yq2f/20130511_144930.png)),
screen glass twice, and full cover housing once.

Now it's time for a change.

<p>
<a href="/img/phone-cmp.png">
<img src="/img/phone-cmp.png" width="400"/>
</a>
</p>

The new phone happened to be HTC One (M8) dual sim variant
(for practical reasons I prefer last but one versions of everything).

Links:

* [Official website](http://www.htc.com/us/smartphones/htc-one-m8/)
* [Wikipedia on HTC One (M8)](http://en.wikipedia.org/wiki/HTC_One_%28M8%29)
* [Specs on gsmarena](http://www.gsmarena.com/htc_one_%28m8%29_dual_sim-6483.php)
* [Device page on XDA Developers forum](http://forum.xda-developers.com/htc-one-m8">Device page on XDA Developers forum)

Phone uses [nano-SIM](http://en.wikipedia.org/wiki/Subscriber_identity_module#Formats) cards -
I had to [cut](http://htc-one.wonderhowto.com/how-to/convert-micro-sim-card-fit-nano-slot-your-htc-one-m8-0154010/) my regular SIMs.

I expected to run [CyanogenMod](http://wiki.cyanogenmod.org/w/M8_Info">CyanogenMod) on it,
but found out dual-sim variant is [not supported](https://jira.cyanogenmod.org/browse/CYAN-6210).

Before starting doing anithing with the device, you should read at least two XDA forum threads:

* [HTC One (M8) FAQ](http://forum.xda-developers.com/showthread.php?t=2711073)
* [Everything Explained (concepts and definitions)](http://forum.xda-developers.com/showthread.php?t=2744194)
* [Russian thread on 4pda](http://4pda.ru/forum/index.php?showtopic=605886)

HTC One (M8) comes with locked bootloader, but HTC is kind enough to provide
an official way to [unlock it](http://www.htcdev.com/bootloader)
(you need to register on htcdev.com, but John Doe with a disposable e-mail
goes fine). The process itself and related topics, like ROM flashing and
building custom kernels are documented there as well, so it's worth read.

Some info about the phone in stock condition:<br>
Model number on the case:

~~~
Model: 0P6B640 M8e
~~~

Software info from OS:

~~~
Amdroid version: 4.4.2
HTC Sense version: 6.0
Software number: 1.45.401.12
HTC SDK API level: 6.24
Kernel version: 3.4.0-g6544e50 and@ABM102#1 SMP PREEMPT
Baseband version: 1.18.30306251.05G_30.57.306251.00L_F
Build number: 1.45.401.12 CL352881 release-keys
~~~

Stock recovery info:<b>
Untick "Settings -> Pover -> Fast boot".<br>
Power-off the device.<br>
Hold VolDown hardware key, press and hold power key.<br>
Phone boots into recovery mode:

<p>
<a href="/img/phone-stock-recovery.png">
<img src="/img/phone-stock-recovery.png" width="200"/>
</a>
</p>

Recovery mode info in txt format:

~~~
M8_DUGL PVT SHIP S-ON
HBOOT-3.18.0.0000
RADIO-1.18.30306251.05G
OpenDSP-v38.2.2-00542-M8974.0311
OS-1.45.401.12
eMMC-boot 2048MB
Jun 10 2014,19:51:53.0
~~~

Connect it to the PC, and press power button while "FASTBOOT" selected.
It'll change to "FASTBOOT USB", PC will recognise the device and install
drivers (if on win). Execute fastboot to get phone info:

~~~
>fastboot getvar all
(bootloader) version: 0.5
(bootloader) version-bootloader: 3.18.0.0000
(bootloader) version-baseband: 1.18.30306251.05G
(bootloader) version-cpld: None
(bootloader) version-microp: None
(bootloader) version-main: 1.45.401.12
(bootloader) version-misc: PVT SHIP S-ON
(bootloader) serialno: ************
(bootloader) imei: ***************
(bootloader) imei2: ***************
(bootloader) meid: 00000000000000
(bootloader) product: m8_dugl
(bootloader) platform: hTCBmsm8974
(bootloader) modelid: 0P6B64000
(bootloader) cidnum: HTC__032
(bootloader) battery-status: good
(bootloader) battery-voltage: 0mV
(bootloader) partition-layout: Generic
(bootloader) security: on
(bootloader) build-mode: SHIP
(bootloader) boot-mode: FASTBOOT
(bootloader) commitno-bootloader: 6b903f73
(bootloader) hbootpreupdate: 11
(bootloader) gencheckpt: 0
all: Done!
finished. total time: 0.027s
~~~

According to this, CID/MID of this device are HTC__032/0P6B64000<br>
Then I proceeded with the [unlock](http://www.htcdev.com/bootloader) procedure.<br>
Unlock wipes the device to factory reset.

After the unlock, I rooted device following
[this](http://forum.xda-developers.com/htc-one-m8/general/guide-how-to-root-m8dugl-m8-dual-sim-t2831154) guide -
tried several recovery images, but only `TWRP_Recovery_2.8.2.1_M8_CPTB.img`
worked (booting recovery image is enough, you don't need to flash it).
After playing with rooted phone for some time I finally figured out
that for real control over the device I do need S-OFF. At the time of writting
the only confirmed way to S-OFF for dual sim vaiant (m8_dugl) was
[SunShine](http://theroot.ninja). My last Chinese purchase did
not pass airline security, I got refunded to PayPall and  had some spare money.
So, investing into S-OFF and supporting the software was not a bad idea.

After enabling S-OFF  I've updated OTA towards latest 3.33.401.6.
Finally ended up with `m8dugl_3.33.401.6_stock_rooted_deodexed_cptb.zip` from
[here](http://forum.xda-developers.com/showpost.php?p=56694363&postcount=128), featuring:

* Rooted
* Busybox
* init.d support
* wp_mod for system write support
* symlink for external storage added for Titanium Backup/legacy app compatibility
* boot image unsecured

As a good base for further modifications.<br>
3.33.401.6 requires different recovery image, and latest official `twrp-2.8.6.0-m8.img` worked fine.
