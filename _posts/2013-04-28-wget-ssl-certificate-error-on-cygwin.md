---
layout: post
title: "wget SSL certificate error on cygwin"
date: 2013-04-28 20:01
---

An issue:

~~~
$ wget -O - https://www.kernel.org
...
ERROR: The certificate of `www.kernel.org' is not trusted.
ERROR: The certificate of `www.kernel.org' hasn't got a known issuer.
~~~

Solution:

~~~
$ apt-cyg install ca-certificates
$ cygcheck -l ca-certificates
/usr/ssl/certs/ca-bundle.crt
/usr/ssl/certs/ca-bundle.trust.crt
$ echo "ca_certificate = /usr/ssl/certs/ca-bundle.crt" >> /etc/wgetrc
~~~
