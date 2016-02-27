---
layout: post
title: "RPi Linaro cross-compiller toolchain"
date: 2013-12-02 15:04 
---

Continuing with RPi booting options, we need a way to compile both u-boot and RPi kernels.
Setting cross-compiler toolchain on x86 Debian linux box is the most optimal way.
It's quite straightforward if you just follow [instructions](http://elinux.org/RPi_Linaro_GCC_Compilation">instructions).
My log (with minor modifications) is below:

~~~
$ sudo apt-get install gperf bison flex texinfo libtool automake subversion
$ mkdir -p /lab/pi/src
$ cd /lab/pi/src
$ wget -c http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-1.19.0.tar.bz2
$ tar -jxvf crosstool-ng-1.19.0.tar.bz2
$ cd crosstool-ng-1.19.0
$ ./configure --prefix=/lab/pi/crosstool-ng-1.19.0
$ make
$ make install
$ export PATH=$PATH:/lab/pi/crosstool-ng-1.19.0/bin
$ mkdir -p /lab/pi/x-compiler/build
$ cd /lab/pi/x-compiler/build
$ ct-ng menuconfig
~~~

Configure values as per [instructions](http://elinux.org/RPi_Linaro_GCC_Compilation#Build_GCC_Linaro).
The only difference in my case was "gcc version" "linaro-4.8-2013.06-1", as "linaro-4.7-2012.10" failed to build due some conflicts.
I did menuconfig once, so now I can do:

~~~
$ wget http://dev.xff.lt/stuff/pi/crosstool-ng-config.txt -O .config
$ ct-ng oldconfig
~~~

my `CT_PREFIX_DIR` is commented out, so you'll need to provide your target directory, and then

~~~
$ ct-ng build
~~~

it took ~50 minutes on VPS with 2 x 2.40 GHz, including downloading all the sources.</p> 
