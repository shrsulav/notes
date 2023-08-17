---
layout: post
title:  "Cooperative Round-Robin Task Scheduler with Cortex-M7!"
date:   2023-08-16 17:33:18 -0500
categories: cortex-m7
---

*Article Created on August 16, 2023*

In this post, I will implement contex switching for a Cortex-M7 microcontroller from scratch. I am using SAME70-Xplained microcontroller board. I am using the Microchip Studio IDE. I will implement a cooperative round-robin task scheduler.


### Getting Started
First of all, I will create a project in Microchip Studio for SAME70-Xplained board. I will use an example project which has configured the UART and LED. With this project, I can print to Serial UART and I can blink LED onboard the microcontroller. With this done, I will move on to the next phase.

### Simple Task Switching
Now that UART and LED are working, I will create a simple task switching mechanism. I create two tasks each with its own superloop. Each of the tasks toggle the LED and print a character to the UART. I will use system calls to create tasks, start the tasks and switch tasks.

To implement such task switching, I create stacks for the two tasks. Each task consists of two stacks - user stack and kernel stack. The user stack is used when the task is being executed in thread mode. When the task executes a system call, kernel stack is used. I also create Task Control Block (TCB) for each of the tasks. The code snippet below shows the TCB structure.

```c
struct tcb
{
    uint32_t    *usp;               // user stack pointer
    uint32_t    *ksp;               // kernel stack pointer
    void        (*fn_entry)(void);  // function for the task to be executed
    uint32_t    task_id;            // numerical identifier for a task
    bool        is_free;            // flag to check if a TCB contains valid task definition or not
};
```

It is important that the first member in the TCB be the user stack pointer and second member be the kernel stack pointer. We will use this information or placement of member in task switching procedure.

In this project, I have defined three SVCs. The code snippet below shows the system calls and userspace equivalents for the system calls.
```c
// system calls
void k_run_scheduler(void);
void k_task_yield(void);
int32_t k_task_create(void (*fn_task_entry)(void));

// userspace equivalents
void run_scheduler(void);
void task_yield(void);
void task_create(void (*fn_task_entry)(void));
```

To create a task, we find an unused TCB (using the ```is_free``` flag) and populate the members of the TCB appropriately. In the userstack, we push ```xPSR```, ```PC```, ```LR```, ```R12```, ```R3```, ```R2```, ```R1``` and ```R0```. The inital ```xPSR``` value should be ```0x01000000``` and ```PC``` should be the function entry point. Other registers can have dummy values. In the kernel stack, we push dummy values for ```R4```-```R11```.

I also define two global pointers to TCBs: ```g_current_task``` and ```g_next_task```. Initially, both pointers are ```NULL```. Once the tasks to be run are defined, I call ```run_scheduler```. The function finds the first available task from the TCB and sets ```g_next_task``` to the TCB found. Then the function triggers PendSV exception. PendSV is where the context switching is performed. PendSV handler, saves the context of ```g_current_task``` (if it's not NULL) and then, loads the context of ```g_next_task```. Upon return from PendSV exception, the control goes to the task pointed to by ```g_next_task```. The code snippet below shows the context switching procedure.

```c
__attribute__((naked)) void PendSV_Handler(void)
{
    __asm volatile(
        "LDR R0, =g_current_task                \n"
        "LDR R1, [R0]                           \n" // R1 points to the TCB
        "CBZ R1, LOAD_CONTEXT                   \n"

        "SAVE_CONTEXT:                          \n"
        "PUSH {R4-R11}                          \n" // Push R4-R11 onto MSP
        "MRS R2, PSP                            \n"
        "STR R2, [R1]                           \n" // update usp in the TCB
        "MRS R2, MSP                            \n"
        "STR R2, [R1,4]                         \n" // update ksp in the TCB

        "LOAD_CONTEXT:                          \n"
        "LDR R1, =g_next_task                   \n"
        "LDR R2, [R1]                           \n" // R2 points to g_next_task
        "LDR R4, [R2]                           \n" // R4 points to g_next_task->usp
        "MSR PSP, R4                            \n"
        "LDR R4, [R2, #4]                       \n" // R4 points to g_next_task->ksp
        "MSR MSP, R4                            \n"
        "POP {R4-R11}                           \n"
        "STR R2, [R0]                           \n" // update g_current_task to be same as g_next_task
        "MVN LR, #2                             \n" // load 0xFFFFFFFD onto LR
        "BX  LR                                 \n");
}
```
The context switching procedure above does the following:
1. check if ```g_current_task``` is NULL or not
2. if the ```g_current_task``` is not null, save the context of the task
* push the registers ```R4``` to ```R11``` to the kernel stack
* get current PSP and update usp in the TCB
* get current MSP and update ksp in the TCB
3. load the context of the next task
* load usp onto the PSP
* load ksp onto the MSP
* pop ```R4``` - ```R11``` from MSP
* update ```g_current_task``` to point to the same TCB as ```g_next_task```
* load 0xFFFFFFFD onto LR (thread mode with PSP)
* return from the exception

Upon returning from the exception, since the LR value was set to be 0xFFFFFFFD, the processor uses PSP to pop the values from the stack ```xPSR```, ```PC```, ```R0-R3``` and ```R12```.

I have implemented a cooperative round-robin scheduler. Each task calls ```task_yield```. When a task calls ```task_yield```, the scheduler finds another task available and switches the context to the next task.

### Future Work
The TCB can be enhanced to store more information about the task such as the access level. Information such as the maximum stack depth can also be stored in the TCB. This information will be helpful in figuring out if the allocated stack for a task is enough or not.

## References
* [Cortex-M7 Technical Reference Manual][cortex_m7_trm]

[cortex_m7_trm]: https://developer.arm.com/documentation/ddi0489/f
[task_switching_guide]: https://medium.com/@dheeptuck/building-a-real-time-operating-system-rtos-ground-up-a70640c64e93
[putty_url]: https://www.putty.org/
[git_install_guide]: https://github.com/git-guides/install-git
[system_calls_guide]: https://antoniogiacomelli.com/2022/11/06/separating-user-space-from-kernel-space-on-arm-cortex-m3/
