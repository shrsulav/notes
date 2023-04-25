---
layout: post
title:  "ThreadX with Beaglebone Black!"
date:   2021-12-23 17:33:18 -0500
categories: jekyll update
---
In this post, I will write about how I built ThreadX (Azure RTOS) for Beaglebone Black. I am using Windows 11 OS host machine to setup cross-development environment for Beaglebone Black. Here's the list of a few things that we will need to get started.

* Beaglebone Black: &nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;[https://beagleboard.org/black][bbb_url]
* Processor SDK for AM335x: [https://www.ti.com/tool/PROCESSOR-SDK-AM335X][starter_kit]
* ThreadX: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[https://github.com/azure-rtos/threadx][threadx_url]
* Code Composer Studio: &nbsp; &nbsp; &nbsp; [https://www.ti.com/tool/CCSTUDIO][ccstudio_url]
* PuTTY Serial Terminal:&nbsp; &nbsp;  &nbsp; &nbsp; &nbsp; [https://www.putty.org/][putty_url]
* Git: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;[https://github.com/git-guides/install-git][git_install_guide]

### Steps
#### Setting up Beaglebone Black Serial Console
The steps to setup serial console on the Beaglebone Black can be found in the references below:
* Setting up Serial Console: [https://dave.cheney.net/2013/09/22/two-point-five-ways-to-access-the-serial-console-on-your-beaglebone-black][serial_console_bbb_1]
* Setting up Serial Console: [https://elinux.org/Beagleboard:BeagleBone_Black_Serial][serial_console_bbb_2]

#### Setting up U-Boot on Beaglebone Black
First of all, I tried to setup U-Boot following the steps provided in the following article.
* Setting up U-Boot for BBB: [https://longervision.github.io/2018/01/10/SBCs/ARM/beaglebone-black-uboot-kernel/][uboot_bbb]

But, there was no `saveenv` command to save the U-Boot command. So, setup U-Boot in the following way.

Flash Debian console image onto microSD card. The image I have used is ["AM3358 Debian 10.3 2020-04-06 1GB SD console"][bbb_image_url]. Login to the Debian terminal. Create the file uEnv.txt at root location.

```bash
$ sudo nano /uEnv.txt
```

Paste the following contents onto the file and save the file.
```
uenvcmd=setenv ethact usb_ether;setenv ipaddr 192.168.1.2;setenv serverip 192.168.1.3; setenv loadaddr 0x80000000;setenv tftproot /;setenv bootfile firmware.bin;tftp ${loadaddr} ${bootfile};echo *** Booting to BareMetal ***;go ${loadaddr};
```

This command assumes that the Beaglebone Black board is connected to the host computer via an Ethernet cable. The IP address of the host computer is set to be 192.168.1.3. A TFTP server is configured to serve the file `firmware.bin` which is the application which we want to run on Beaglebone Black board.

Reboot the board. Now, U-Boot should fetch firmware.bin file from the TFTP server and execute the application.

#### Setting up the Development Environment
This guide [here][getting_started_demo] provides a demo on how to get started with Beaglebone Black development using [AM335x Starterware Kit][starter_kit]. So, go ahead and setup the tools necessary to get started with the development. The guide shows a LED blinking application. Instead of the LED blinking application, let's setup an application which prints to UART console.

#### Setting up Git Repository for the Project

#### Setting up CCS project for development

#### Mode of Operation for ThreadX

#### Integrating ThreadX

#### Testing ThreadX with Two Threads
To test if the ThreadX kernel works, define two threads which do nothing but increment a variable. Since the UART print is not working, we cannot print anyting onto the console yet. Synchronize the threads in such a way that the threads wait for a semaphore (a different semaphore for each thread), and once the variable is incremented, releases semaphore for the other thread, so that the other thread can run. Build the program and make sure that the threads are running, by putting break points on the two threads. The threads should increment the variables alternatively.

#### Taking Care of the Interrupts

#### Setting up SysTick Timer

#### Suspending threads with `tx_thread_sleep`

### References

* [Bare Metal on the BeagleBone (Black and Green)][ref_1]
* [Running a Baremetal Beaglebone Black (Part 2)][ref_2]

[starter_kit]: https://www.ti.com/tool/PROCESSOR-SDK-AM335X
[bbb_url]: https://beagleboard.org/black
[threadx_url]: https://github.com/azure-rtos/threadx
[ccstudio_url]: https://www.ti.com/tool/CCSTUDIO
[putty_url]: https://www.putty.org/
[git_install_guide]: https://github.com/git-guides/install-git
[getting_started_demo]: https://www.youtube.com/watch?v=iOQisBaDANA
[serial_console_bbb_1]: https://dave.cheney.net/2013/09/22/two-point-five-ways-to-access-the-serial-console-on-your-beaglebone-black
[serial_console_bbb_2]: https://elinux.org/Beagleboard:BeagleBone_Black_Serial
[uboot_bbb]: https://longervision.github.io/2018/01/10/SBCs/ARM/beaglebone-black-uboot-kernel/
[bbb_image_url]: https://beagleboard.org/latest-images
[ref_1]: https://opencoursehub.cs.sfu.ca/bfraser/grav-cms/ensc351/guides/files/BareMetalGuide.pdf
[ref_2]: https://twosixtech.com/running-a-baremetal-beaglebone-black-part-2/
