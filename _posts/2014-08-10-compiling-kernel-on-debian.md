---
layout: post
title: "Compiling kernel on Debian"
date: 2014-08-10 10:05
---

On a Debian wheezy I needed a newer kernel.

If you just try to install a package from testing on stable, you will run into errors:

~~~
# wget -c http://ftp.debian.org/debian/pool/main/l/linux/linux-image-3.14-2-amd64_3.14.13-2_amd64.deb

# dpkg -i linux-image-3.14-2-amd64_3.14.13-2_amd64.deb
dpkg: regarding linux-image-3.14-2-amd64_3.14.13-2_amd64.deb containing linux-image-3.14-2-amd64:
 linux-image-3.14-2-amd64 breaks initramfs-tools (<< 0.110~)
  initramfs-tools (version 0.109.1) is present and installed.

dpkg: error processing linux-image-3.14-2-amd64_3.14.13-2_amd64.deb (--install):
 installing linux-image-3.14-2-amd64 would break initramfs-tools, and
 deconfiguration is not permitted (--auto-deconfigure might help)
Errors were encountered while processing:
 linux-image-3.14-2-amd64_3.14.13-2_amd64.deb
~~~

Compiling vanilla kernel the "debian way":

~~~
# apt-get build-dep linux-image-`uname -r`
# wget -c https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.14.16.tar.xz
# tar -xvf linux-3.14.16.tar.xz
# cd linux-3.14.16
# make mrproper
# ar p ../linux-image-3.14-2-amd64_3.14.13-2_amd64.deb data.tar.xz  | tar -JxO ./boot/config-3.14-2-amd64 > .config
# make oldconfig
# make -j$(nproc) deb-pkg
~~~

~~~
# cd ..
# ls -1 *3.14.16*.deb
linux-firmware-image-3.14.16_3.14.16-1_amd64.deb
linux-headers-3.14.16_3.14.16-1_amd64.deb
linux-image-3.14.16_3.14.16-1_amd64.deb
linux-image-3.14.16-dbg_3.14.16-1_amd64.deb
linux-libc-dev_3.14.16-1_amd64.deb
~~~
