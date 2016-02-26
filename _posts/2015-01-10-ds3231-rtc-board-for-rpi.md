---
layout: post
title: "DS3231 RTC Board for RPi"
date: 2015-01-10 20:45
---

DS3231 RTC Board arrived from [dx.com](http://www.dx.com/p/ds3231-raspberry-pi-rtc-board-real-time-clock-module-for-arduino-black-277258).

<p>
<a href="/img/DS3231_1.png">
<img src="/img/DS3231_1.png" width="400"/>
</a>
</p>

<p>
<a href="/img/DS3231_2.png">
<img src="/img/DS3231_2.png" width="400"/>
</a>
</p>

<p>
<a href="/img/DS3231_3.png">
<img src="/img/DS3231_3.png" width="400"/>
</a>
</p>

<p>
<a href="/img/DS3231_4.png">
<img src="/img/DS3231_4.png" width="400"/>
</a>
</p>

Enable I2C:

~~~
$ sudo vi /etc/modules
~~~

Add two lines:

~~~
i2c-bcm2708
i2c-dev
~~~

Comment out i2c modules in blacklist

~~~
$ sudo vi /etc/modprobe.d/raspi-blacklist.conf
~~~

resulting:

~~~
#blacklist spi-bcm2708
#blacklist i2c-bcm2708
~~~

Load them manually:

~~~
$ sudo modprobe i2c-bcm2708
$ sudo modprobe i2c-dev
~~~

Install i2c-tools:

~~~
$ sudo apt-get install i2c-tools
~~~

Locate i2c device:

~~~
$ sudo i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- UU -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- 68 -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
~~~

Load rtc module:

~~~
$ sudo modprobe rtc-ds1307
~~~

Make it permanent:

~~~
$ sudo vi /etc/modules
~~~

Adding a line:

~~~
rtc-ds1307
~~~

Init i2c device:

~~~
$ sudo bash -c "echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device"
~~~

Check NTP status:

~~~
$ ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
-ns1.telecom.lt  212.59.3.3       2 u   36   64  377    1.678    3.862   1.745
*ntp2.litnet.lt  .GPS.            1 u   28   64  377    2.501    4.168   1.682
+ns2.telecom.lt  212.59.3.3       2 u   62   64  377    1.820    1.874   1.546
+ntp1.litnet.lt  .GPS.            1 u    2   64  377    2.527    3.299   0.921
~~~

Check date:

~~~
$ date
Sat Jan 10 20:53:58 UTC 2015
~~~

Wite system time to RTC:

~~~
$ sudo hwclock -w
~~~

Validate:

~~~
$ sudo hwclock -r
Sat 10 Jan 2015 20:56:14 UTC  -0.162439 seconds
~~~

Make it into startup scripts:

~~~
$ sudo vi  /etc/rc.local
~~~

Add before "exit 0":

~~~
# RTC
echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
sudo hwclock -s
~~~
