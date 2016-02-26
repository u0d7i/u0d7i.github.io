---
layout: post
title: "Relays on RPi"
date: 2014-02-04 01:20
---

The other day I've got a $10 gift-card code from DX.
Didn'n need anything special, so decided to go for some random stuff, and ended up ordering 4 
[relay modules](http://dx.com/p/arduino-5v-relay-module-blue-black-121354), $2.40 each then. They arrived:

<p>
<a href="/img/rl-1.png">
<img src="/img/rl-1.png" width="400"/>
</a>
</p>

On the control side, there are 3 pins, labeled +, - and S. On the load side - NO (Normal Open),
central common not labeled, and NC (Normal Closed).

Connection schematics:

<p>
<a href="/img/rl-s.png">
<img src="/img/rl-s.png" width="400"/>
</a>
</p>

First, connected GPIO part:

<p>
<a href="/img/rl-2.png">
<img src="/img/rl-2.png" width="400"/>
</a>
</p>

On the RPi:

~~~
$ echo 4 > /sys/class/gpio/export
$ echo "out" > /sys/class/gpio/gpio4/direction
$ echo "1" > /sys/class/gpio/gpio4/value
$ echo "0" > /sys/class/gpio/gpio4/value
~~~

Relay LED goes on/off and it clicks when setting values. Worx. Time to connect the load:

<p>
<a href="/img/rl-3.png">
<img src="/img/rl-3.png" width="400"/>
</a>
</p>

~~~
$ echo "1" > /sys/class/gpio/gpio4/value
~~~

Result:

<p>
<a href="/img/rl-4.png">
<img src="/img/rl-4.png" width="400"/>
</a>
</p>

The same can be done with [WiringPi](http://wiringpi.com/). Install:

~~~
$ git clone git://git.drogon.net/wiringPi
$ cd wiringPi
$ ./build
~~~

Use:

~~~
$ gpio reset
$ gpio -g mode 4 out
$ gpio -g write 4 1
$ gpio -g write 4 0
~~~

WiringPi can [address](http://wiringpi.com/pins/) pins both by GPIO number as above, or by pin number:

~~~
$ gpio write 7 1
~~~

Status:

~~~
$ gpio readall
+----------+-Rev2-+------+--------+------+-------+
| wiringPi | GPIO | Phys | Name   | Mode | Value |
+----------+------+------+--------+------+-------+
|      0   |  17  |  11  | GPIO 0 | IN   | Low   |
|      1   |  18  |  12  | GPIO 1 | IN   | Low   |
|      2   |  27  |  13  | GPIO 2 | IN   | Low   |
|      3   |  22  |  15  | GPIO 3 | IN   | Low   |
|      4   |  23  |  16  | GPIO 4 | IN   | Low   |
|      5   |  24  |  18  | GPIO 5 | IN   | Low   |
|      6   |  25  |  22  | GPIO 6 | IN   | Low   |
|      7   |   4  |   7  | GPIO 7 | OUT  | High  |
|      8   |   2  |   3  | SDA    | IN   | High  |
|      9   |   3  |   5  | SCL    | IN   | High  |
|     10   |   8  |  24  | CE0    | IN   | Low   |
|     11   |   7  |  26  | CE1    | IN   | Low   |
|     12   |  10  |  19  | MOSI   | IN   | Low   |
|     13   |   9  |  21  | MISO   | IN   | Low   |
|     14   |  11  |  23  | SCLK   | IN   | Low   |
|     15   |  14  |   8  | TxD    | IN   | Low   |
|     16   |  15  |  10  | RxD    | IN   | Low   |
|     17   |  28  |   3  | GPIO 8 | IN   | Low   |
|     18   |  29  |   4  | GPIO 9 | IN   | Low   |
|     19   |  30  |   5  | GPIO10 | IN   | Low   |
|     20   |  31  |   6  | GPIO11 | IN   | Low   |
+----------+------+------+--------+------+-------+
~~~

Automation - relay.sh:

~~~
#!/bin/bash

# set GPIO pin 4 to OUT
gpio -g mode 4 out

while true; do
# read state
STATE=`gpio -g read 4`
case $STATE in
  0)
    echo "state: OFF, switching to ON"
    gpio -g write 4 1
    ;;
  1)
    echo "state: ON, switching to OFF"
    gpio -g write 4 0
    ;;
esac
sleep 1
done
~~~
