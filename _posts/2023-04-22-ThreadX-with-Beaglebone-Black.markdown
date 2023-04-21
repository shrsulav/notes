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

[starter_kit]: https://www.ti.com/tool/PROCESSOR-SDK-AM335X
[bbb_url]: https://beagleboard.org/black
[threadx_url]: https://github.com/azure-rtos/threadx
[ccstudio_url]: https://www.ti.com/tool/CCSTUDIO
[putty_url]: https://www.putty.org/
[git_install_guide]: https://github.com/git-guides/install-git
[getting_started_demo]: https://www.youtube.com/watch?v=iOQisBaDANA
