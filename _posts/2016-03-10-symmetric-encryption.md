---
layout: post
title: "Symmetric encryption"
date: 2016-03-10 23:00
---

For a long time I've been using [ccrypt](http://ccrypt.sourceforge.net/) utility for symmetric
encryption on linux. The only problem with it - it's not in default install on most debian-based
distros I use.

There are 2 possible alternatives present in default install:

Openssl:

~~~
Nokia-N900:~# echo "this is plain text" > /tmp/123

Nokia-N900:~# openssl enc -aes-192-cbc -in /tmp/123 -out /tmp/123.sslenc
enter aes-192-cbc encryption password:
Verifying - enter aes-192-cbc encryption password:

Nokia-N900:~# rm /tmp/123

Nokia-N900:~# openssl enc -d -aes-192-cbc -in /tmp/123.sslenc -out /tmp/123
enter aes-192-cbc decryption password:

Nokia-N900:~# cat /tmp/123
this is plain text
~~~

GnuPG:

~~~
Nokia-N900:~# echo "this is plain text" > /tmp/123
Nokia-N900:~# gpg --symmetric --output /tmp/123.gpg /tmp/123 
Enter passphrase: 
Repeat passphrase:

Nokia-N900:~# rm /tmp/123

Nokia-N900:~# gpg /tmp/123.gpg 
gpg: CAST5 encrypted data
gpg: encrypted with 1 passphrase
gpg: WARNING: message was not integrity protected

Nokia-N900:~# cat /tmp/123
this is plain text
~~~

