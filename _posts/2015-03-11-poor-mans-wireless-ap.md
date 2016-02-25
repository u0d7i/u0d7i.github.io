---
layout: post
title: "Poor man's wireless AP"
date: 2015-03-11 22:36
---

This hotel happened to be conservative one: "Internet access is available. Network cable is required".
So, here is a short log how I converted my netbook into a wireless access point (yes, I do have a network cable):

~~~
$ sudo apt-get install hostapd dnsmasq
$ sudo /etc/init.d/dnsmasq stop
$ sudo sed -i 's/^ENABLED=1/ENABLED=0/' /etc/default/dnsmasq

$ sudo mkdir /data/ap

$ sudo tee -a /data/ap/dnsmasq.cfg > /dev/null <<EOF
interface=wlan0
dhcp-range=192.168.111.20,192.168.111.254,255.255.255.0,12h
EOF

$ sudo tee -a /data/ap/hostapd.conf > /dev/null <<EOF
interface=wlan0
driver=nl80211
hw_mode=g
ssid=MyAP
channel=9
wpa=2
wpa_passphrase=MyPassword
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
EOF

$ sudo tee -a /data/ap/ap.sh > /dev/null <<EOF
#!/bin/sh
echo "hostapd"
hostapd -B /data/ap/hostapd.conf
echo "dnsmasq"
dnsmasq -C /data/ap/dnsmasq.cfg
echo "ip"
ifconfig wlan0 inet 192.168.111.1 netmask 255.255.255.0
echo "forwarding"
echo 1 > /proc/sys/net/ipv4/ip_forward
echo "nat"
iptables -t nat -A POSTROUTING -s 192.168.111.0/24 ! -d 192.168.111.0/24 -j MASQUERADE
echo "done"
EOF

$ sudo chmod +x /data/ap/ap.sh
$ sudo /data/ap/ap.sh
~~~

Channel selection assisted by [Wifi Analyzer](https://play.google.com/store/apps/details?id=com.farproc.wifi.analyzer).

