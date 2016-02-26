---
layout: post
title: "RPi B+ GPIO reference"
date: 2014-10-05 11:31
---

New setup:

* [Raspberry Pi Model B+](http://www.dx.com/p/raspberry-pi-project-board-mode-b-made-in-uk-green-334720)
* [RPi B+ GPIO T-Cobbler](http://www.dx.com/p/type-t-gpio-expansion-board-accessory-for-raspberry-pi-b-black-338708)
* [RPi B+ 40-Pin F-F Data Cable](http://www.dx.com/p/rpi40p-raspberry-pi-b-dedicated-40-pin-female-to-female-data-cable-22cm-336411)
* [Mini/Half(400p) Breadboard](http://www.dx.com/p/mini-prototype-printed-circuit-board-breadboard-white-140716)

<p>
<a href="/img/rpi-bplus-1.png">
<img src="/img/rpi-bplus-1.png" width="400"/>
</a>
</p>

There are different ways to reference GPIO pins, so, here is a table,
resembling [WiringPi](http://wiringpi.com/) `gpio readall` output:

~~~
 +-----+-----+---------+----+--B Plus--+----+---------+-----+-----+
 | BCM | wPi |   Name  | BL | Physical | BR | Name    | wPi | BCM |
 +-----+-----+---------+----+----++----+----+---------+-----+-----+
 |     |     |     3V3 |  1 |  1 || 2  |  1 | 5v0     |     |     |
 |   2 |   8 |    SDA1 |  2 |  3 || 4  |  2 | 5V0     |     |     |
 |   3 |   9 |    SCL1 |  3 |  5 || 6  |  3 | GND     |     |     |
 |   4 |   7 |   GPIO4 |  4 |  7 || 8  |  4 | TXD0    | 15  | 14  |
 |     |     |     GND |  5 |  9 || 10 |  5 | RXD0    | 16  | 15  |
 |  17 |   0 |  GPIO17 |  6 | 11 || 12 |  6 | GPIO18  | 1   | 18  |
 |  27 |   2 |  GPIO27 |  7 | 13 || 14 |  7 | GND     |     |     |
 |  22 |   3 |  GPIO22 |  8 | 15 || 16 |  8 | GPIO23  | 4   | 23  |
 |     |     |     3V3 |  9 | 17 || 18 |  9 | GPIO24  | 5   | 24  |
 |  10 |  12 | SPIMOSI | 10 | 19 || 20 | 10 | GND     |     |     |
 |   9 |  13 | SPIMISO | 11 | 21 || 22 | 11 | GPIO25  | 6   | 25  |
 |  11 |  14 | SPISCLK | 12 | 23 || 24 | 12 | SPICS0  | 10  | 8   |
 |     |     |     GND | 13 | 25 || 26 | 13 | SPICS1  | 11  | 7   |
 |   0 |  30 |  EEDATA | 14 | 27 || 28 | 14 | EECLK   | 31  | 1   |
 |   5 |  21 |   GPIO5 | 15 | 29 || 30 | 15 | GND     |     |     |
 |   6 |  22 |   GPIO6 | 16 | 31 || 32 | 16 | GPIO12  | 26  | 12  |
 |  13 |  23 |  GPIO13 | 17 | 33 || 34 | 17 | GND     |     |     |
 |  19 |  24 |  GPIO19 | 18 | 35 || 36 | 18 | GPIO16  | 27  | 16  |
 |  26 |  25 |  GPIO26 | 19 | 37 || 38 | 19 | GPIO20  | 28  | 20  |
 |     |     |     GND | 20 | 39 || 40 | 20 | GPIO21  | 29  | 21  |
 +-----+-----+---------+----+----++----+----+---------+-----+-----+
 | BCM | wPi |   Name  | BL | Physical | BR | Name    | wPi | BCM |
 +-----+-----+---------+----+--B Plus--+----+---------+-----+-----+
~~~

Where:

* Physical - Board pin number
* BL/BR - Breadbord Left/Right pin row number
* Name - T-Cobbler pin name
* wPi - WiringPi library GPIO pin number
* BCM - Broadcom BCM chip GPIO number

And below is RPi B+ GPIO header pinout details, stolen from
[here](http://www.raspberrypi-spy.co.uk/2014/07/raspberry-pi-b-gpio-header-details-and-pinout/)

<p>
<a href="/img/Raspberry-Pi-GPIO-Layout-Model-B-Plus-312x1024.png">
<img src="/img/Raspberry-Pi-GPIO-Layout-Model-B-Plus-312x1024.png" width="200"/>
</a>
</p>
