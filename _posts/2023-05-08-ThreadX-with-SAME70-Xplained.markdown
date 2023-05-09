---
layout: post
title:  "ThreadX with SAME70 Xplained!"
date:   2022-08-01 17:33:18 -0500
categories: same70 cortex-m7
---
In this post, I will write about how I built ThreadX (Azure RTOS) for SAME70 Xplained Evaulation Kit. I am using Windows 11 OS host machine to setup cross-development environment for SAME70 Xplained. Here's the list of a few things that we will need to get started.

* SAME70 Xplained Kit:&nbsp;&nbsp;&nbsp;[https://www.microchip.com/en-us/development-tool/ATSAME70-XPLD][board_url]
* Microchip Studio: [https://www.microchip.com/en-us/tools-resources/develop/microchip-studio][microchip_studio]
* ThreadX: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[https://github.com/azure-rtos/threadx][threadx_url]
* PuTTY Serial Terminal:&nbsp; &nbsp;  &nbsp; &nbsp; &nbsp; [https://www.putty.org/][putty_url]
* Git: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;[https://github.com/git-guides/install-git][git_install_guide]

### Setting up the Starting Project
Create a new git repository in github with a README. Clone the repository in local setup. Open up Microchip Studio and import an example project. For this project, we will use lwIP project as the starting project. Go to `File->New->Atmel Start Example Project`. Set the filter for SAME70 Xplained Board. Select lwIP example project. Generate the project. Select the git repository we just created as the workspace for the project. Verify that the project works. Commit the project at this stage.

### Integrating ThreadX
Define `tx_application_define` function to create two threads and two semaphores.

```c
#include "consoleUtils.h"
#include "tx_api.h"
#include "tx_port.h"

TX_BYTE_POOL byte_pool_0;
TX_SEMAPHORE semaphore_0, semaphore_1;
TX_THREAD my_thread, my_thread_2;

void my_thread_entry(ULONG thread_input)
{
    UINT status;
    UINT thread_counter = 0;
    /* Enter into a forever loop. */

    while(1)
    {
        /* Get the semaphore with suspension. */
        status = tx_semaphore_get(&semaphore_0, TX_WAIT_FOREVER);

        /* Check status. */
        if (status != TX_SUCCESS) break;

        /* Increment thread counter. */
        thread_counter++;

        ConsoleUtilsPrintf("\r\nThread 1 Count: %d", thread_counter);

        /* Release the semaphore. */
        status = tx_semaphore_put(&semaphore_1);

        /* Check status. */
        if (status != TX_SUCCESS) break;
    }
}

void my_thread_entry_2(ULONG thread_input)
{
    UINT status;
    UINT thread_counter = 0;

    /* Enter into a forever loop. */
    while(1)
    {
        /* Get the semaphore with suspension. */
        status = tx_semaphore_get(&semaphore_1, TX_WAIT_FOREVER);

        /* Check status. */
        if (status != TX_SUCCESS) break;

        /* Increment thread counter. */
        thread_counter++;

        ConsoleUtilsPrintf("\r\nThread 2 Count: %d", thread_counter);

        /* Release the semaphore. */
        status = tx_semaphore_put(&semaphore_0);

        /* Check status. */
        if (status != TX_SUCCESS) break;
    }
}

void tx_application_define(void *first_unused_memory)
{
    CHAR *pointer;

    /* Create a byte memory pool from which to allocate the thread stacks. */
    tx_byte_pool_create(&byte_pool_0, "byte pool 0", first_unused_memory, 8192);

    /* Allocate the stack for thread 0. */
    tx_byte_allocate(&byte_pool_0, &pointer, 1024, TX_NO_WAIT);

    /* Create my_thread! */
    tx_thread_create(&my_thread, "My Thread",
                     my_thread_entry, 0x1234, pointer, 1024,
                     3, 3, TX_NO_TIME_SLICE, TX_AUTO_START);

    /* Allocate the stack for thread 1. */
    tx_byte_allocate(&byte_pool_0, &pointer, 1024, TX_NO_WAIT);

    /* Create my_thread! */
    tx_thread_create(&my_thread_2, "My Thread 2",
                     my_thread_entry_2, 0x1234, pointer, 1024,
                     3, 3, TX_NO_TIME_SLICE, TX_AUTO_START);

    /* Create the semaphore. */
    tx_semaphore_create(&semaphore_0, "semaphore 0", 1);

    /* Create the semaphore. */
    tx_semaphore_create(&semaphore_1, "semaphore 1", 0);
}
```

Call `tx_kernel_enter()` in `main()`. Build the ThreadX library and enetLwip project.

At this point, the build process should complain about undefined references to `_sp`, `_stack_bottom` and `_end` which are referenced in `tx_initialize_low_level.S` file. Comment out the code in the file that sets up stacks for different modes of operation for Cortex-A8. Stacks are setup early in the boot process before main is called. This resolves the issue for `_stack_bottom` as it is not being used anymore. We also do not need to get the value of `_sp`, which is the top of the allocated stack region in linker script. What we do need is to initialize the variables `_tx_thread_system_stack_ptr` and `_tx_initialize_unused_memory`.

`_tx_thread_system_stack_ptr` should point to the top of the SVC stack. So, make sure that the execution is in SVC mode and get the stack pointer. Then, store the stack pointer at that point to the variable `_tx_thread_system_stack_ptr`. To ensure that the execution is in SVC mode, modify the stack setup procedure in `init.S` file from `system` library. The stack setup process initializes stack for SYSTEM mode at last and SVC mode before that. With that, the `main()` executions starts with SYSTEM mode. Swap the initialization for SVC mode stack and SYSTEM mode to ensure that the `main()` execution starts with SVC mode.

`_tx_initialize_unused_memory` should point to the highest RAM address that has been used till that point. For this we define `_end` variable in the linker script to point the address after stack has been allocated.

Build all the libraries and the enetLwip project. Restart the board. Make sure that the execution is as expected. Now, lwIP should not start. Rather, the execution should go to the two ThreadX threads.

### Mode of Operation for ThreadX

### Taking Care of the Interrupts

### Setting up SysTick Timer


### Suspending threads with `tx_thread_sleep`
Now, when we add `tx_thread_sleep(200)` into the two threads that we have created previously, the threads should sleep for 2 seconds.

### Starting lwIP from ThreadX thread


## References

* [SAME70-Xplained User Guide][ref_1]
* [ARM Cortex-M7 Processor Technical Reference Manual][ref_2]
* [ThreadX User Guide][threadx_guide]

[microchip_studio]: https://www.microchip.com/en-us/tools-resources/develop/microchip-studio
[board_url]: https://www.microchip.com/en-us/development-tool/ATSAME70-XPLD
[threadx_url]: https://github.com/azure-rtos/threadx
[putty_url]: https://www.putty.org/
[git_install_guide]: https://github.com/git-guides/install-git
[threadx_guide]: https://learn.microsoft.com/en-us/azure/rtos/threadx/about-this-guide
[ref_1]: https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-44050-Cortex-M7-Microcontroller-SAM-E70-XPLD-Xplained_User-guide.pdf
[ref_2]: https://developer.arm.com/documentation/ddi0489/b/