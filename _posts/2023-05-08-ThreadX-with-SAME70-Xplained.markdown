---
layout: post
title:  "ThreadX with SAME70 Xplained!"
date:   2022-08-01 17:33:18 -0500
categories: same70 cortex-m7
---
*Article Created on May 08, 2023*

In this post, I will write about how I built ThreadX (Azure RTOS) for SAME70 Xplained Evaulation Kit. I am using Windows 11 OS host machine to setup cross-development environment for SAME70 Xplained. Here's the list of a few things that we will need to get started.

* SAME70 Xplained Kit:&nbsp;&nbsp;&nbsp;[https://www.microchip.com/en-us/development-tool/ATSAME70-XPLD][board_url]
* Microchip Studio: [https://www.microchip.com/en-us/tools-resources/develop/microchip-studio][microchip_studio]
* ThreadX: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[https://github.com/azure-rtos/threadx][threadx_url]
* PuTTY Serial Terminal:&nbsp; &nbsp;  &nbsp; &nbsp; &nbsp; [https://www.putty.org/][putty_url]
* Git: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;[https://github.com/git-guides/install-git][git_install_guide]

### Setting up the Starting Project
Create a new git repository in github with a README. Clone the repository in local setup. Open up Microchip Studio and import an example project. For this project, we will use lwIP project as the starting project. Go to `File->New->Atmel Start Example Project`. Set the filter for SAME70 Xplained Board. Select lwIP example project. Generate the project. Select the git repository we just created as the workspace for the project. Verify that the project works. On running the program, you should see prints in the UART console (baudrate is configured to be 9600 bps). The program prints the statically configured IP address to the console. Try to ping the IP address of the board from the host computer. If ping is successful, the project works. Commit the project at this stage.

### Integrating ThreadX
Clone the threadx source code with tag `v6.2.1_rel` from git repository.
```bash
$ git clone --depth 1 --single-branch --branch v6.2.1_rel https://github.com/azure-rtos/threadx.git
```

Remove unnecessary source files. We do need the common folder and the ports folder for the time being. In ports folder, retain only the cortex_m7 port folder for GNU toolchain. Add the include directories from threadx `common\inc` and `ports\cortex_m7_gnu\inc` and build the program. At this point

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

        /* Release the semaphore. */
        status = tx_semaphore_put(&semaphore_1);

        tx_thread_sleep(500); // corresponding to 5s

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

        tx_thread_sleep(500); // corresponding to 5s

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

At this point, the build process should complain about undefined references to `__RAM_segment_used_end__` and `_vectors` which are referenced in `tx_initialize_low_level.S` file. Define `__RAM_segment_used_end__` in the linker script which should point to the first memory address in RAM which is not used or, the address after heaps and stacks have been setup. `_vectors` should point to the vector table or exception table. In our project, the table is named `exception_table`. So, rename `_vectors` as `exception_table`. Modify the `SYSTEM_CLOCK` and `SYSTICK_CYCLES` to configure a 10ms systick.

```c
SYSTEM_CLOCK      =   300000000                 // 300MHz
SYSTICK_CYCLES    =   ((SYSTEM_CLOCK)/100 -1)   // for 10ms systick
```

`_tx_thread_system_stack_ptr` should point to the top of the stack. The stack address is the first member in the `exception_table` structure.

The following code in `tx_initialize_low_level.S` initializes the vector table offset register with the address of the vector table. This step is already done in the C startup code before main is called, so this code can be removed from `tx_initialize_low_level.S`.

```as
/* Setup Vector Table Offset Register.  */
MOV     r0, #0xE000E000                         // Build address of NVIC registers
LDR     r1, =_vectors                           // Pickup address of vector table
STR     r1, [r0, #0xD08]                        // Set vector table address
```

The threadx source comes with the definition of SysTick_Handler. The SysTick_Handler is also defined in the `main.c` file. So, remove the definition in `main.c` file.

Build the project. Check if the program runs as expected, i.e. the execution should alterate between the two threads that we have created. Also, check if `tx_thread_sleep` causes to sleep for correct amount of time.

For some reason, printf function does not work after this.

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