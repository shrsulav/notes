---
layout: post
title:  "Attitude Determination and Control Software for StudSat-2"
date:   2017-07-01 17:33:18 -0500
categories: stm32 cortex-m4
---
StudSat-2 [[1]][ref_1] is a nano-satellite project developed under the initiative of Centre of Excellence for Space Research [[2]][ref_2]. I worked as an intern and undergraduate research student with Attitude Determination and Control Systems (ADCS) team for nearly a year. During my time with the team, I worked on writing ADCS firmware for STM32F4-Discovery board. I used FreeRTOS kernel to implement the firmware.

### Attitude Determination Algorithm

The attitude determination algorithm used is a magnetometer-only attitude determination algorithm [[3]][ref_3]. The algorithm comprises of three main computational blocks:

1. Pre-Filter
2. Intertial-Frame Vector Calculation
3. Attitude Filter

![Attitude Determination Algorithm Block Diagram][{{site.url}}/notes/docs/assets/images/Picture1.jpg]

The algorithm uses Kalman Filter and involves a lot of matrix multiplication. STM32F407 microcontroller based on ARM Cortex-M4 [[4]][ref_4] has a floating point unit to accelerate computations involving floating point numbers. I used CMSIS DSP library [[5]][ref_5] to optimize the matrix multiplications.

### Firmware
As a part of the project, I interfaced Magnetometer, Gyroscope, GPS and Reaction Wheels to the STM32F4-Discovery Board.

The firmware is implemented using FreeRTOS [[6]][ref_6] as the kernel. The firmware consists of following tasks:

| Task Name  | Description |
| ------------------- | ------------- |
| Bootup  | Periodically informs C&DH microcontroller board that it is awake until C&DH acknowledges  |
| IPC  | Acknowledges the message coming from C&DH board and takes necessary action according to the message received  |
|Gyro | Checks the angular velocity of the satellite either on request from C&DH or periodically |
| Detumble | Detumbles the satellite |
| AttDet | Determines the attitude of the satellite |
| AttControl | Controls the attitude of the satellite  |

I used Keil IDE for the development using Windows 7 host system. For visualizing the FreeRTOS objects, I used Precipio Tracelyzer [[7]].
![Tracelyzer Snapshot][{{site.url}}/notes/docs/assets/images/Picture2.png]



## References
* [[1] Wikipedia Page: StudSat-2][ref_1]
* [[2] Website: Centre of Excellence for Space Research, Bangalore, India][ref_2]
* [[3] Master Thesis: Magnetometer-only attitude determination with application to the M-SAT mission, Jason D. Searcy][ref_3]
* [[4] ARM Cortex-M4 Processor Technical Reference Manual][ref_4]
* [[5] CMSIS DSP Software Library][ref_5]
* [[6] Website: FreeRTOS][ref_6]
* [[7] Website: Precipio Tracelyzer][ref_7]

[ref_1]: https://en.wikipedia.org/wiki/StudSat-2
[ref_2]: https://www.nmit.ac.in/center-for-space-research.php
[ref_3]: https://scholarsmine.mst.edu/masters_theses/6892/
[ref_4]: https://developer.arm.com/documentation/100166/0001?lang=en
[ref_5]: https://www.keil.com/pack/doc/CMSIS/DSP/html/index.html
[ref_6]: https://www.freertos.org/
[ref_7]: https://percepio.com/tracealyzer/
