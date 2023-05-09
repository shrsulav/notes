---
layout: post
title:  "Attitude Determination and Control Software for StudSat-2"
date:   2017-07-01 17:33:18 -0500
categories: stm32 cortex-m4
---
StudSat-2 [[1]][ref_1] is a nano-satellite project developed under the initiative of Centre of Excellence for Space Research [[2]][ref_2]. I worked as an intern and undergraduate research student with Attitude Determination and Control Systems (ADCS) team for nearly a year. During my time with the team, I worked on writing ADCS firmware for STM32F4-Discovery board. I used FreeRTOS kernel to implement the firmware.

The attitude determination algorithm used is a magnetometer-only attitude determination algorithm [[3]][ref_3]. The algorithm comprises of three main computational blocks:

1. Pre-Filter
2. Intertial-Frame Vector Calculation
3. Attitude Filter

![Attitude Determination Algorithm Block Diagram][{{site.url}}/docs/assets/images/Picture1.jpeg]





## References
* [[1] Wikipedia Page: StudSat-2][ref_1]
* [[2] Website: Centre of Excellence for Space Research, Bangalore, India][ref_2]
* [[3] Master Thesis: Magnetometer-only attitude determination with application to the M-SAT mission, Jason D. Searcy][ref_3]

[ref_1]: https://en.wikipedia.org/wiki/StudSat-2
[ref_2]: https://www.nmit.ac.in/center-for-space-research.php
[ref_3]: https://scholarsmine.mst.edu/masters_theses/6892/
