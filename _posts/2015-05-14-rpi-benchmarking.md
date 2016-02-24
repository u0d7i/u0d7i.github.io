---
layout: post
title: "RPi benchmarking"
date: 2015-05-14 11:03
---

There is a bunch of different Raspberry Pi boards on my desk (B, B+, 2B):
<p>
<a href="/img/DSC03079.png">
<img src="/img/DSC03079.png" width="400"/>
</a>
</p>
B and B+ have same CPU, B+ and 2B looks the same. I wanted to compare
performance, so I did some benchmarking - the old classic way, by compiling
Linux kernel onboard. First, I needed reliable SD card (I/O thingie impacts
results a lot). There were several available right away:
<p>
<a href="/img/DSC03082.png">
<img src="/img/DSC03082.png" width="400"/>
</a>
</p>
Did some tests (<a href="http://www.heise.de/download/h2testw.html">h2testw</a>,
<a href="http://oss.digirati.com.br/f3/">F3</a>, etc.):

~~~
Kingston 4G C4
Writing speed: 4.07 MByte/s
Reading speed: 11.6 MByte/s
~~~

~~~
Kingston 4G C10
Writing speed: 8.73 MByte/s
Reading speed: 11.7 MByte/s
~~~

~~~
SanDisk Ultra 16G U1
Writing speed: 4.95 MByte/s
Reading speed: 11.7 MByte/s
~~~

~~~
Sandisk Ultra 8G C10
Writing speed: 11.0 MByte/s
Reading speed: 11.7 MByte/s
~~~

~~~
Samsung EVO 8G U1
Writing speed: 8.39 MByte/s
Reading speed: 11.7 MByte/s
~~~

Sandisk Ultra 8G C10 was the best for a job, imaged 
<a href="http://downloads.raspberrypi.org/raspbian_latest">latest Raspbian</a> on it. Test prep:

~~~
$ sudo apt-get install bc
$ mkdir perftest
$ cd perftest
$ wget -c https://www.kernel.org/pub/linux/kernel/v3.x/linux-$(uname -r | cut -d- -f1).tar.xz
$ tar -xvf linux*.xz
~~~

Performed the same test with the same SD card on 3 RPi Boards:

~~~
$ cd linux*/
$ rm -f ../*.deb
$ make mrproper
$ zcat /proc/config.gz > .config
$ yes "" | make oldconfig
~~~

Used `gpio` utility from <a href="http://wiringpi.com/">Wiring Pi</a> to id the board, and timed 'debian way' of compiling kernel to benchmark:

~~~
$ gpio -v | tail -2
$ time make -j$(nproc) deb-pkg
~~~

Results:

~~~
$ gpio -v | tail -2
Raspberry Pi Details:
  Type: Model B, Revision: 2, Memory: 512MB, Maker: Sony

$ time make -j$(nproc) deb-pkg
...
real    741m37.328s
user    693m9.250s
sys     31m57.460s
~~~

~~~
$ gpio -v | tail -2
Raspberry Pi Details:
  Type: Model B+, Revision: 1.2, Memory: 512MB, Maker: Sony

$ time make -j$(nproc) deb-pkg
...
real    741m18.997s
user    692m32.580s
sys     32m7.080s
~~~

~~~
$ gpio -v | tail -2
Raspberry Pi Details:
  Type: Model 2, Revision: 1.1, Memory: 1024MB, Maker: Sony

$ time make -j$(nproc) deb-pkg
...
real    111m51.491s
user    377m23.940s
sys     23m14.370s
~~~

Summary:

<span style="color:red">
B/B+: 12 hours 21 minutes<br>
2B:   1 hour 52 minutes
</span>


