---
layout: post
title: "Pi serial console"
date: 2013-11-19 15:46
---

RPi has HDMI and composite video out and 2 USB ports for keyboard/mouse, but this is not very convinient,
when sitting most of the time by a normal workstation. In this case optimal way for user i/o would be serial
console port with USB interface towards PC.

Raspberry Pi has built-in [TTL](http://en.wikipedia.org/wiki/Transistor-transistor_logic)
(Transistor-transistor logic) [UART](http://en.wikipedia.org/wiki/UART) (Universal Asynchronous
Receiver/Transmitter) for serial communication, and it can be connected to PC via external USB-to serial
converter. I bought [this one](http://dx.com/p/usb-2-0-to-ttl-uart-5-pin-cp2102-module-serial-converter-blue-152317)
from DX. It's based on [CP210](http://www.silabs.com/Support%20Documents/TechnicalDocs/CP2102-9.pdf)
chip, has 5 pins (+3.3V, TX, RX, GND and +5V). It does not have RTS or DTR line for resetting Arduino, but we don't need it fo RPi.

Linux kernel recognises it out of the box on the PC side by cp210x driver, and creates ttyUSBx interface:

~~~
[ 4998.908243] usb 2-2: new full-speed USB device number 3 using ohci_hcd
[ 4999.363425] usb 2-2: New USB device found, idVendor=10c4, idProduct=ea60
[ 4999.363433] usb 2-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 4999.363437] usb 2-2: Product: CP2102 USB to UART Bridge Controller
[ 4999.363441] usb 2-2: Manufacturer: Silicon Labs
[ 4999.363444] usb 2-2: SerialNumber: 0001
[ 4999.418936] usbcore: registered new interface driver usbserial
[ 4999.418962] USB Serial support registered for generic
[ 4999.419425] usbcore: registered new interface driver usbserial_generic
[ 4999.419430] usbserial: USB Serial Driver core
[ 4999.425099] USB Serial support registered for cp210x
[ 4999.425360] cp210x 2-2:1.0: cp210x converter detected
[ 4999.820368] usb 2-2: reset full-speed USB device number 3 using ohci_hcd
[ 5000.279312] usb 2-2: cp210x converter now attached to ttyUSB0
[ 5000.279312] usbcore: registered new interface driver cp210x
[ 5000.279312] cp210x: v0.09:Silicon Labs CP210x RS232 serial adaptor driver
~~~

~~~
$ lsusb | grep CP210x
Bus 002 Device 003: ID 10c4:ea60 Cygnal Integrated Products, Inc. CP210x UART Bridge / myAVR mySmartUSB light
~~~

For Windows you need drivers avalable from [here](http://www.silabs.com/products/mcu/pages/usbtouartbridgevcpdrivers.aspx).

Serial port settings are:

~~~
Speed (baud rate): 115200
Bits: 8
Parity: None
Stop Bits: 1
Flow Control: None
~~~

Connect GPIO TXD pin on the board to the converters RXD pin,  GPIO RXD pin on the board to converters TXD pin,
and GND to GND on both board and converter, leaving both power pins on the converter alone:

<p><a href="/img/pi-serial.png">
<img src="/img/pi-serial.png" width="400"/>
</a></p>

We actually can use +5V pin on a converter, connecting it to 5V GPIO pin on the board to feed RPi
from PC's USB via the same connection, instead of using external power via micro-usb connector
on the board - it's sufficient to boot RPi without external USB periferials attached, but the
problem here - we will not see the start of the boot process, as serial interface device is non
existant prior to connecting converter to PC and launching terminal emulator takes time.
Or, alternatively, we can add power switch to the wire between 5V pins.

More info on connecting Pi via serial port is available [here](http://elinux.org/RPi_Serial_Connection).
