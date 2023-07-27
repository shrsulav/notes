---
layout: post
title:  "Writing an Operating System with Cortex-M7!"
date:   2023-07-26 17:33:18 -0500
categories: cortex-m7
---

*Article Created on July 26, 2023*

In this post, I will write an operating system for a Cortex-M7 microcontroller from scratch. I am using SAME70-Xplained microcontroller board. I am using the Microchip Studio IDE.

* PuTTY Serial Terminal:&nbsp;  &nbsp; &nbsp; &nbsp; [https://www.putty.org/][putty_url]
* Git: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;[https://github.com/git-guides/install-git][git_install_guide]

### Getting Started
First of all, I will create a project in Microchip Studio for SAME70-Xplained board. I will use an example project which has configured the UART and LED. With this project, I can print to Serial UART and I can blink LED onboard the microcontroller. With this done, I will move on to the next phase.

### Simple Task Switching
Now that UART and LED are working, I will create a simple task switching mechanism. This does not use any form of system call. I create two tasks each with its own superloop. Each of the tasks toggle the LED and print a character to the UART. I will setup a SysTick Handler with a period of 1ms. On each SysTick interrupt, it switches the task.

To implement such task switching, I create stacks for the two tasks. I also create Task Control Block (TCB) for each of the tasks. ```stack_ptr``` points to the stack of the task. ```next_task``` points to the next task to be switched to. ```fn_entry``` points to the entry point of the task. ```task_id``` is a numerical identifier for the task. ```is_free``` indicates if the TCB has been allocated to any task or not.

```c
struct tcb
{
    uint32_t    *stack_ptr;
    struct tcb  *next_task;
    void        (*fn_entry)(void);
    uint32_t    task_id;
    bool        is_free;
};
```

### System Calls

## References
* [Task Swtiching Guide][task_switching_guide]
* [Cortex-M7 Technical Reference Manual][cortex_m7_trm]
* [System Calls Guide][system_calls_guide]

[cortex_m7_trm]: https://developer.arm.com/documentation/ddi0489/f
[task_switching_guide]: https://medium.com/@dheeptuck/building-a-real-time-operating-system-rtos-ground-up-a70640c64e93
[putty_url]: https://www.putty.org/
[git_install_guide]: https://github.com/git-guides/install-git
[system_calls_guide]: https://antoniogiacomelli.com/2022/11/06/separating-user-space-from-kernel-space-on-arm-cortex-m3/
