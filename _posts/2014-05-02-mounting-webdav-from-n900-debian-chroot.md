---
layout: post
title: "Mounting WebDAV from N900 Debian chroot"
date: 2014-05-02 16:47
---

Prereq:

~~~
# apt-get install davfs2
# mkdir -p /mnt/dav
~~~

Test:

~~~
# mount -t davfs https://server.net.lab/dav /mnt/dav
Please enter the username to authenticate with server
https://server.net.lab/dav or hit enter for none.
  Username: davuser
Please enter the password to authenticate user davuser with server
https://server.net.lab/dav or hit enter for none.
  Password:
/sbin/mount.davfs: the server certificate is not trusted
<...>
You only should accept this certificate, if you can
verify the fingerprint! The server might be faked
or there might be a man-in-the-middle-attack.
Accept certificate for this session? [y,N] y

# mount | grep /mnt/dav
https://server.net.lab/dav on /mnt/dav type davfs (rw,nosuid,noexec,nodev,_netdev)

# umount /mnt/dav/
/sbin/umount.davfs: waiting while mount.davfs (pid 5996) synchronizes the cache .. OK
~~~

Add server to the fstab:

~~~
# echo "https://server.net.lab/dav /mnt/dav davfs noauto,user 0 0" >> /etc/fstab
~~~

Add dav servers' credentials:

~~~
# echo "/mnt/dav davuser davpass" >> /etc/davfs2/secrets
~~~

Get self-signed cert pem, and modify cfg t use it:

~~~
# echo | openssl s_client -connect server.net.lab:443 2>/dev/null | openssl x509 > /etc/davfs2/certs/server.pem
# grep ^servercert /etc/davfs2/davfs2.conf
servercert server.pem
~~~

Mount:

~~~
# mount /mnt/dav
~~~
