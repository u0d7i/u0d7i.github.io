---
layout: post
title: "apt-cyg fix"
date: 2013-10-06 21:13
---

After cygwin repository structure change, separating architectures, old good vanilla apt-cyg does not work anymore.
Fast dirty fix:

~~~
$ sed -i 's@mirror/setup@mirror/x86/setup@' /bin/apt-cyg
~~~

Or better use one of the more elegant [patches](https://gist.github.com/rcmdnk/6189422) available.
