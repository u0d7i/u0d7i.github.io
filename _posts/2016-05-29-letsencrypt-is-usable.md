---
layout: post
title: "letsencrypt"
date: 2016-05-29 22:15
---

Short log on using [letsencrypt](https://letsencrypt.org/)
with a [3rd party bash client](https://github.com/lukas2511/letsencrypt.sh):

~~~
$ sudo su -
# apt-get install git openssl curl
# cd /opt
# git clone https://github.com/lukas2511/letsencrypt.sh
# echo "example.com www.example.com" > /opt/letsencrypt.sh/domains.txt
# mkdir -p /var/www/html/.well-known/acme-challenge
# echo 'WELLKNOWN="/var/www/html/.well-known/acme-challenge"' > /opt/letsencrypt.sh/config
# /opt/letsencrypt.sh/letsencrypt.sh -c
# echo "38 4 * * 7 root /opt/letsencrypt.sh/letsencrypt.sh -c" > /etc/cron.d/letsencrypt
# vi /etc/apache2/sites-enabled/010-default-ssl.conf 
# grep SSLCertificate /etc/apache2/sites-enabled/010-default-ssl.conf
  SSLCertificateKeyFile   /opt/letsencrypt.sh/certs/example.com/privkey.pem
  SSLCertificateFile      /opt/letsencrypt.sh/certs/example.com/cert.pem
  SSLCertificateChainFile /opt/letsencrypt.sh/certs/example.com/chain.pem
# apachectl restart

~~~

