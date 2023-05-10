---
layout: post
title:  "Raspberry Pi Development Notes"
date:   2021-03-01 17:33:18 -0500
categories: raspberry-pi cortex-a53
---
*Article Created on May 10, 2023*

This article contains references that I used while developing on Raspberry Pi 3 and 4.

### Documentation relevant to Raspberry Pi

* [Raspberry Pi Pinout][ref_0_1]
* [Sparkfun: Raspberry Pi GPIO Pinout][ref_0_2]
* [Raspberri Pi Documentation: GPIO][ref_0_3]
* [Website: Raspberry Pi Documentation][ref_0_4]

[ref_0_1]: https://pinout.xyz/
[ref_0_2]: https://learn.sparkfun.com/tutorials/raspberry-gpio/gpio-pinout
[ref_0_3]: https://www.raspberrypi.com/documentation/computers/raspberry-pi.html
[ref_0_4]: https://www.raspberrypi.com/documentation/

### Creating SystemD Services
The objective is to start a python script during boot time. In the following example, the name of the python script is `service_file.py`.

Create a new file with `.service` extension in `/lib/systemd/system/` directory.

Enter the following text in the file:
```
[Unit]
Description=My New Service
After=multi-user.target

[Service]
ExecStart=/usr/bin/python3 /home/pi/service_file.py

[Install]
WantedBy=multi-user.target
```

#### Reference
* [Sparkfun: How to run a Raspberry Pi program on startup][ref_1_1]

[ref_1_1]: https://learn.sparkfun.com/tutorials/how-to-run-a-raspberry-pi-program-on-startup#method-3-systemd

### Boot-Time Optimization

Following are some of the useful articles that I went through to reduce the boot-time of Raspberry Pi.

* [Single Board Bytes: How to fast boot Raspberry Pi][ref_2_1]
* [Raspians: How to fast boot Raspberry Pi][ref_2_2]
* [Himesh's Blog: Fast boot with Raspberry Pi][ref_2_3]
* [Furkan Tokac - Raspberry Pi 3 Fastboot - Less Than 2 Seconds][ref_2_4]
* [Instant Pi OS][ref_2_5]
* [Embedded Linux boot time optimization training][ref_2_6]

[ref_2_1]: https://singleboardbytes.com/637/how-to-fast-boot-raspberry-pi.htm
[ref_2_2]: https://raspians.com/how-to-fast-boot-raspberry-pi/
[ref_2_3]: https://himeshp.blogspot.com/2018/08/fast-boot-with-raspberry-pi.html
[ref_2_4]: https://www.furkantokac.com/rpi3-fast-boot-less-than-2-seconds/
[ref_2_5]: https://github.com/IronOxidizer/instant-pi
[ref_2_6]: https://bootlin.com/doc/training/boot-time/boot-time-slides.pdf

### Bluetooth Low Energy with BlueZ
Following are the articles that were useful to me while writing an application for Raspberry Pi 4 to communicate to a smartphone using BLE. I used DBUS with Python bindings to communicate to BlueZ via DBUS. I used nRF Connect app in Android phone to test the application.

* [Bluetooth Low Energy (BTLE) Peripherals with Raspberry Pi][ref_3_1]
* [Using BLE Devices with a Raspberry Pi][ref_3_2]
* [Bluez Documentation][ref_3_3]
* [dbus-python tutorial][ref_3_4]
* [nRF Connect for Mobile][ref_3_5]

[ref_3_1]: https://www.raspberrypi-bluetooth.com/index.html
[ref_3_2]: https://www.argenox.com/library/bluetooth-low-energy/using-raspberry-pi-ble/
[ref_3_3]: https://github.com/bluez/bluez/tree/master/doc
[ref_3_4]: https://dbus.freedesktop.org/doc/dbus-python/tutorial.html
[ref_3_5]: https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp&hl=en_CA&gl=US

### Location Update using Mapbox

* [Mapbox GL JS Guide][ref_4_1]
* [Sparkfun: NMEA Reference Manual][ref_4_2]
* [Sixfab Raspberri Pi 4G/LTE Cellular Modem Kit][ref_4_3]

[ref_4_1]: https://docs.mapbox.com/mapbox-gl-js/guides/
[ref_4_2]: https://www.sparkfun.com/datasheets/GPS/NMEA%20Reference%20Manual-Rev2.1-Dec07.pdf
[ref_4_3]: https://sixfab.com/product/raspberry-pi-4g-lte-modem-kit/

### Creating HTML user interfaces
The following are the articles on creating user interface using HTML-CSS and communicating with the user interface using Python and Javascript. The framework used is Eel which uses WebSockets.

* [Eel: Communicating with HTML user interface using Python and JavaScript][ref_5_1]
* [Eel: Get simple GUI for your Python][ref_5_2]
* [Introducing WebSockets][ref_5_3]

[ref_5_1]: https://github.com/python-eel/Eel
[ref_5_2]: https://sed-paris.gitlabpages.inria.fr/developer-meetups/2019-01-22/DevMeetup-Eel-20190122.pdf
[ref_5_3]: https://web.dev/websockets-basics/
