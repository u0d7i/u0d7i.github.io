---
layout: post
title: "RPi, rtl_sdr and DIY discone"
date: 2013-12-26 12:20
---

RPi works great as [Software Defined Radio](https://en.wikipedia.org/wiki/Software-defined_radio) station. Here is my setup:
cheap USB DVB-T dongle with RTL2832U chipset and Elonics E4000 tuner (more info on RTL-SDR and supported hardware can be found
[here](http://sdr.osmocom.org/trac/wiki/rtl-sdr), I've bought mine [here](http://dx.com/p/e4000-2832u-usb-dvb-t-tv-receiver-stick-white-black-172105)),
connected to Raspbery Pi USB port. 

<p>
<a href="/img/sdr-1.png">
<img src="/img/sdr-1.png" width="400"/>
</a>
</p>

RPi uses default [raspbian](http://www.raspbian.org) image from [here](http://www.raspberrypi.org/downloads) on a 4GB micro-SD card.

Kernel identifies device and loads apropriate module:

~~~
[    3.179476] usb 1-1.3: new high-speed USB device number 4 using dwc_otg
[    3.302080] usb 1-1.3: New USB device found, idVendor=0ccd, idProduct=00d7
[    3.310923] usb 1-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[    3.319909] usb 1-1.3: Product: RTL2838UHIDIR
[    3.325762] usb 1-1.3: Manufacturer: Realtek
[    3.331517] usb 1-1.3: SerialNumber: 00000001

[    8.555931] usb 1-1.3: dvb_usb_v2: found a 'TerraTec Cinergy T Stick+' in warm state
[    8.567425] usbcore: registered new interface driver dvb_usb_rtl28xxu
[    8.640382] usb 1-1.3: dvb_usb_v2: will pass the complete MPEG2 transport stream to the software demuxer
[    8.665831] DVB: registering new adapter (TerraTec Cinergy T Stick+)
[    8.718103] usb 1-1.3: DVB: registering adapter 0 frontend 0 (Realtek RTL2832 (DVB-T))...
[    8.811593] i2c i2c-0: e4000: Elonics E4000 successfully identified
[    8.837842] Registered IR keymap rc-empty
[    8.849773] input: TerraTec Cinergy T Stick+ as /devices/platform/bcm2708_usb/usb1/1-1/1-1.3/rc/rc0/input0
[    8.876496] rc0: TerraTec Cinergy T Stick+ as /devices/platform/bcm2708_usb/usb1/1-1/1-1.3/rc/rc0
[    8.904216] usb 1-1.3: dvb_usb_v2: schedule remote query interval to 400 msecs
[    8.931812] usb 1-1.3: dvb_usb_v2: 'TerraTec Cinergy T Stick+' successfully initialized and connected


# lsusb -s 001:004
Bus 001 Device 004: ID 0ccd:00d7 TerraTec Electronic GmbH
~~~

Actually, if I connect USB dongle to the live Pi, it reboots, probably due some power issues, but if it's booted attached, everything's fine.
RPi is networked and connected to the LAN.

Building rtl-sdr:

~~~
# apt-get install cmake libusb-1.0-0-dev
# mkdir -p /lab/srd
# cd /lab/srd
# git clone git://git.osmocom.org/rtl-sdr.git
# cd rtl-sdr/
# mkdir build
# cd build
# cmake ../
# make
# make install
# ldconfig
~~~

Testing:

~~~
# rtl_test -t
Found 1 device(s):
  0:  Terratec T Stick PLUS

Using device 0: Terratec T Stick PLUS

Kernel driver is active, or device is claimed by second instance of librtlsdr.
In the first case, please either detach or blacklist the kernel module
(dvb_usb_rtl28xxu), or enable automatic detaching at compile time.

usb_claim_interface error -6
Failed to open rtlsdr device #0.
~~~

Removing and blacklisting `dvb_usb_rtl28xxu` module:

~~~
# modprobe -r dvb_usb_rtl28xxu
# echo '# blacklist dvb_usb_rtl28xxu for rtl-sdr' > /etc/modprobe.d/rtl-sdr.conf
# echo 'blacklist dvb_usb_rtl28xxu' >> /etc/modprobe.d/rtl-sdr.conf
~~~

And testing again:

~~~
# rtl_test -t
Found 1 device(s):
  0:  Terratec T Stick PLUS

Using device 0: Terratec T Stick PLUS
Found Elonics E4000 tuner
Supported gain values (14): -1.0 1.5 4.0 6.5 9.0 11.5 14.0 16.5 19.0 21.5 24.0 29.0 34.0 42.0
Benchmarking E4000 PLL...
[E4K] PLL not locked for 52000000 Hz!
[E4K] PLL not locked for 2223000000 Hz!
[E4K] PLL not locked for 1112000000 Hz!
[E4K] PLL not locked for 1252000000 Hz!
E4K range: 53 to 2222 MHz
E4K L-band gap: 1112 to 1252 MHz
~~~

So, we have a wide range, 53-2222 MHz receiver. And now we need an antenna.
We are hitting a situation from an old ham radio saying: "Buy a $10 radio and a $100 antenna".
[Discone](https://en.wikipedia.org/wiki/Discone_antenna) is a good choice for our purposes,
because it's wideband and we can create one out of scrap.

First, some visual understanding, how it should look like [here](https://www.google.com/search?tbm=isch&q=discone).

Second, several DIY variants:
[here](http://helix.air.net.au/index.php/d-i-y-discone-for-rtlsdr/),
[here](http://www.rtl-sdr.com/home-made-coat-hanger-discone/),
and [here](http://www.radioscanner.ru/forum/topic16114.html)(ru).

Third - [Discone Antenna Calculato](http://www.changpuak.ch/electronics/calc_11.php)

To build mine I used:

* 8x 30cm stainless steel rulers, like [this](http://www.ebay.com/sch/i.html?_nkw=stainless+steel+ruler+30cm+-angle)
* 8pcs of 1m galvanized hanger wire hooks for suspension celing, like [this](http://www.alibaba.com/product-gs/406350696/Hanger_Wire_for_Suspension_Ceiling.html)
* 32mm PVC bell-end drainage/sewer pipe like [this](http://www.alibaba.com/product-gs/1571492667/bell_end_and_plain_end_bulk.html) - two pieces, 25cm and 1m, and 32mm socket plug
* M6 screw with DIN-315 wing nut
* 2x hose metal clamps like [this](http://www.amazon.com/Precision-Brand-B20HS-Stainless-Clamp/dp/B007Q4YD38)
* ~10m of RG58 50Ω coax cable

Stainless steel rulers form a disc part of the discone, and are connected with a screw through the plug to the core of the coax cable.
Hanger wires, mounted with clamps to the pipe, forms the cone part, and are connected to the shield.

<p><a href="/img/sdr-2a.png">
<img src="/img/sdr-2a.png" width="400"/></a></p>

<p><a href="/img/sdr-6.png">
<img src="/img/sdr-6.png" width="400"/></a></p>

<p><a href="/img/sdr-5.png">
<img src="/img/sdr-5.png" width="400"/></a></p>

<p><a href="/img/sdr-3.png">
<img src="/img/sdr-3.png" width="400"/></a></p>

After connecting the antenna to the USB dongle, on the RPi we can launch

~~~
# rtl_tcp -a 0.0.0.0
~~~

dowload latest build of the [SDR#](http://sdrsharp.com) to some PC on the same network, choose "RTL-SDR/TCP" as a source, configure IP of the RPi and port 1234 in it's settings, and play around.

<p><a href="/img/sdr-4.png">
<img src="/img/sdr-4.png" width="400"/></a></p>

On this screen we see SDR# tuned to [Vilnius Approach](http://www.liveatc.net/search/?icao=vno) Air Traffic frequiency (120.700 MHz AM)
with active voice communication with RyanAir board taking place.

