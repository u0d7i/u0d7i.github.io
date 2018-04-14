---
layout: post
title: "Cryptocontainer entropy tests"
date: 2018-04-14 21:00
---

Today I've got my new 128G USB flash drive from Hong Kong, intending to use it as an additional
encrypted storage. Setting up LUKS volume implies filling all the drive with random data first, to
avoid possible statistical attack. I've plugged USB in, created random data key file, formatted the stick as
LUKS volume, and started feeding it with /dev/zero contents via dd, using LUKS infra as an entropy source
(it's faster than dd'ing /dev/urandom). But 128G is kind'a lot for an usb drive, so, even using this trick
it took several hours. I was wondering if it's really that necessary, so, decided to do some syntheric tests.

There are plenty of tools for binary data and entropy visualisation out there, I was using the
[first google hit](http://binvis.io) for that.

case 1:<br>
Empty file, filled with zeros, resembling "factory format".
~~~
$ dd if=/dev/zero of=test.img bs=1M count=6
~~~
result: [entropy-test1.png](/img/entropy-test1.png)

case 2:<br>
Adding LUKS header.
~~~
$ sudo cryptsetup luksFormat -q -v -d /tmp/keyfile test.img
~~~
result: [entropy-test2.png](/img/entropy-test2.png)

case 3:<br>
Creating filesystem within encrypted container.
~~~
$ sudo cryptsetup luksOpen -d /tmp/keyfile test.img test_crypt
$ sudo mkfs.ext4 /dev/mapper/test_crypt
$ sudo cryptsetup luksClose test_crypt
~~~
result: [entropy-test3.png](/img/entropy-test3.png)

case 4:<br>
Filling HALF of the container size with zeros via LUKS, creating FS on top.
~~~
$ sudo cryptsetup luksOpen -d /tmp/keyfile test.img test_crypt
$ sudo dd if=/dev/zero of=/dev/mapper/test_crypt bs=1M count=3
$ sudo mkfs.ext4 /dev/mapper/test_crypt
$ sudo cryptsetup luksClose test_crypt
~~~
result: [entropy-test4.png](/img/entropy-test4.png)

case 5 (the right way):<br>
Formatting LUKS, filling ALL the storage vis zeros via LUKS,
killing LUKS header space via /dev/urandom, rebuilding LUKS,
creating FS on top.
~~~
$ dd if=/dev/zero of=test.img bs=1M count=6
$ sudo losetup /dev/loop0 test.img 
$ sudo cryptsetup luksFormat -q -v -d /tmp/keyfile /dev/loop0
$ sudo cryptsetup luksOpen -d /tmp/keyfile /dev/loop0 test_crypt
$ sudo dd if=/dev/zero of=/dev/mapper/test_crypt
$ sudo cryptsetup luksClose test_crypt
$ sudo dd if=/dev/urandom of=/dev/loop0 bs=512 count=4096
$ sudo losetup -d /dev/loop0
$ sudo cryptsetup luksFormat -q -v -d /tmp/keyfile test.img
$ sudo cryptsetup luksOpen -d /tmp/keyfile test.img test_crypt
$ sudo mkfs.ext4 /dev/mapper/test_crypt
$ sudo cryptsetup luksClose test_crypt
~~~
result: [entropy-test5.png](/img/entropy-test5.png)

Compare:

<p>
<table border="0">
<tr>
<td>entropy-test1.png<br><a href="/img/entropy-test1.png"><img src="/img/entropy-test1.png" width="100"/></a></td>
<td>entropy-test2.png<br><a href="/img/entropy-test2.png"><img src="/img/entropy-test2.png" width="100"/></a></td>
<td>entropy-test3.png<br><a href="/img/entropy-test3.png"><img src="/img/entropy-test3.png" width="100"/></a></td>
<td>entropy-test4.png<br><a href="/img/entropy-test4.png"><img src="/img/entropy-test4.png" width="100"/></a></td>
<td>entropy-test5.png<br><a href="/img/entropy-test5.png"><img src="/img/entropy-test5.png" width="100"/></a></td>
</tr>
</table>
</p>

