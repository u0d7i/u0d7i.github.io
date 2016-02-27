---
layout: post
title: "apt-cyg and .xz archives"
date: 2013-12-02 14:23
---

Stock apt-cyg has one more problem, besides the one already fixed before.
Yesterday I've noticed, that it fails to install new packages in .tar.xz format (before there were only .tar.bz2).
The problem is in these lines:

~~~
echo "Unpacking..."
cat $file | bunzip2 | tar > "/etc/setup/$pkg.lst" xvf - -C /
~~~

There are several fixes ([1](https://code.google.com/p/apt-cyg/issues/detail?id=26), [2](https://code.google.com/p/apt-cyg/issues/detail?id=32)) available,
but most have errors in them. The easiest way to fix this, is to let tar guess compression method by itself (modern tar can do it):

~~~
echo "Unpacking..."
tar -xvf $file -C / > "/etc/setup/$pkg.lst"
~~~

apt-cyg fixed the right way can be found in [this](https://github.com/Simuc/apt-cyg) fork on github.
