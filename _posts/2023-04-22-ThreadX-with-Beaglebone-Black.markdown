---
layout: post
title:  "ThreadX with Beaglebone Black!"
date:   2021-12-23 17:33:18 -0500
categories: jekyll update
---
In this post, I will write about how I built ThreadX (Azure RTOS) for Beaglebone Black. I am using Windows 11 OS host machine to setup cross-development environment for Beaglebone Black. Here's the list of a few things that we will need to get started.

* Beaglebone Black: &nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;[https://beagleboard.org/black][bbb_url]
* Processor SDK for AM335x: [https://www.ti.com/tool/PROCESSOR-SDK-AM335X][starter_kit]
* TI Starterware AM335x: &nbsp; &nbsp; &nbsp; &nbsp;[https://www.ti.com/tool/download/STARTERWARE-AM335X/02.00.01.01][ti_starterware_url]
* ThreadX: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[https://github.com/azure-rtos/threadx][threadx_url]
* Code Composer Studio: &nbsp; &nbsp; &nbsp; [https://www.ti.com/tool/CCSTUDIO][ccstudio_url]
* PuTTY Serial Terminal:&nbsp; &nbsp;  &nbsp; &nbsp; &nbsp; [https://www.putty.org/][putty_url]
* Git: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;[https://github.com/git-guides/install-git][git_install_guide]

### Setting up Beaglebone Black Serial Console
The steps to setup serial console on the Beaglebone Black can be found in the references below:
* Setting up Serial Console: [https://dave.cheney.net/2013/09/22/two-point-five-ways-to-access-the-serial-console-on-your-beaglebone-black][serial_console_bbb_1]
* Setting up Serial Console: [https://elinux.org/Beagleboard:BeagleBone_Black_Serial][serial_console_bbb_2]

### Setting up U-Boot on Beaglebone Black
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

### Setting up Git Repository for the Project
Our git repository will consist of two directories - build and src. We will use build folder as the workspace for CCStudio. We will use src folder to have TI Starterware source files and ThreadX source files.

### Setting up the Development Environment
Install TI Starterware Kit for AM335x (available [here] [ti_starterware_url]). Apply Beaglebone Black support patch. Download ThreadX version 6.2.1 (available [here][threadx_url]). This guide [here][getting_started_demo] provides a demo on how to get started with Beaglebone Black development using [AM335x Starterware Kit][starter_kit]. So, go ahead and setup the tools necessary to get started with the development. The guide shows a LED blinking application. Instead of the LED blinking application, let's setup an application which uses lwIP network stack.

Import the CCS projects `drivers`, `platform`, `system`, `utils` and `enetLwip`. The projects are configured to use TI compilers. Change it to GNU Compiler.

Define the following symbols for all the projects.
```
am335x
beaglebone
SUPPORT_UNALIGNED
gcc
MMCSD
UARTCONSOLE
```

Add the include paths for all the projects.
```
${TI_STARTERWARE_HOME}\include
${TI_STARTERWARE_HOME}\include\hw
${TI_STARTERWARE_HOME}\include\armv7a
${TI_STARTERWARE_HOME}\include\armv7a\am335x
${TI_STARTERWARE_HOME}\usblib\include
```

For `enetLwip` project, add the following include paths as well
```
${LWIP_HOME}
${LWIP_HOME}\src\include
${LWIP_HOME}\src\include\ipv4
${LWIP_HOME}\src\include\lwip
${LWIP_HOME}\apps\httpserver_raw
${LWIP_HOME}\ports\cpsw\include
${TI_STARTERWARE_HOME}\examples\beaglebone\enet_lwip
```

On building `system` library, the build will fail because the project is configured to use TI Compilers. Since we are using GNU Compiler, replace the files which are failing to build with the files in "${TI_STARTERWARE_HOME}\system_config\armv7a\cgt" directory from "${TI_STARTERWARE_HOME}\system_config\armv7a\gcc" directory.

The files are:

```
exceptionhandler.asm -> exceptionhandler.S
cp15.asm -> cp15.S
init.asm  -> init.S
cpu.c -> cpu.c
```

For enetLwip project, delete the `enetLwip.cmd` file which is replaced by AM335x.lds file. The build process will fail to find the following symbols:
```
_bss_start
_bss_end
_stack
```
So, define them in AM335x.lds linker script.

Also delete the startup file from enetLwip project. Define `-specs=nosys.specs`. Otherwise, the compiler will complain saying -
````undefined reference to `_exit````.

The project will build `enetLwip.out` file. To build `enetLwip.bin` file, go to `Properties->Build->GNU Objcopy Utility` and select the option to enable the utility. Also, go to `Properties->Build->GNU Objcopy Utility->General Options`. Change to `--output-target` to binary.

As a post-build step to the project, add a command to copy the *.bin file to tftp server directory.

Turn on the board and make sure that the firmware loads and executes. At this point, the the board should request for IP address from a DHCP server.

### Integrating ThreadX
Create a library project for ThreadX. Modify the enetLwip project to link with ThreadX library. Define `tx_application_define` function to create two threads and two semaphores.

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
ThreadX runs both the kernel and application threads in SVC mode.

### Taking Care of the Interrupts
Modify the vector table in startup.c file such that the interrupt exception is handled by `__tx_irq_handler` instead of the `IRQHandler` that is defined by the Starterware source code. The interrupt handling in `IRQHandler` consists of three parts - context save, interrupt handling  and context restore. `__tx_irq_handler` already consists of code to save and restore the context. So, take the actual interrupt handling code from `IRQHandler` and use it in `__tx_irq_handler`.

```
__tx_irq_handler:

    /* Jump to context save to save system context.  */
    B       _tx_thread_context_save
__tx_irq_processing_return:
//
    /* At this point execution is still in the IRQ mode.  The CPSR, point of
       interrupt, and all C scratch registers are available for use.  In
       addition, IRQ interrupts may be re-enabled - with certain restrictions -
       if nested IRQ interrupts are desired.  Interrupts may be re-enabled over
       small code sequences where lr is saved before enabling interrupts and
       restored after interrupts are again disabled.  */

    LDR      r0, =ADDR_THRESHOLD      @ Get the IRQ Threshold
    LDR      r1, [r0, #0]
    STMFD    r13!, {r1}               @ Save the threshold value

    LDR      r2, =ADDR_IRQ_PRIORITY   @ Get the active IRQ priority
    LDR      r3, [r2, #0]
    STR      r3, [r0, #0]             @ Set the priority as threshold

    LDR      r1, =ADDR_SIR_IRQ        @ Get the Active IRQ
    LDR      r2, [r1]
    AND      r2, r2, #MASK_ACTIVE_IRQ @ Mask the Active IRQ number

    MOV      r0, #NEWIRQAGR           @ To enable new IRQ Generation
    LDR      r1, =ADDR_CONTROL

    CMP      r3, #0                   @ Check if non-maskable priority 0
    STRNE    r0, [r1]                 @ if > 0 priority, acknowledge INTC
    DSB                               @ Make sure acknowledgement is completed

    @
    @ Enable IRQ and switch to system mode. But IRQ shall be enabled
    @ only if priority level is > 0. Note that priority 0 is non maskable.
    @ Interrupt Service Routines will execute in System Mode.
    @
    MRS      r14, cpsr                @ Read cpsr
    ORR      r14, r14, #MODE_SYS
    BICNE    r14, r14, #I_BIT         @ Enable IRQ if priority > 0
    MSR      cpsr, r14


    STMFD    r13!, {r14}              @ Save lr_usr

    /* Interrupt nesting is allowed after calling _tx_thread_irq_nesting_start
       from IRQ mode with interrupts disabled.  This routine switches to the
       system mode and returns with IRQ interrupts enabled.

       NOTE:  It is very important to ensure all IRQ interrupts are cleared
       prior to enabling nested IRQ interrupts.  */
#ifdef TX_ENABLE_IRQ_NESTING
    BL      _tx_thread_irq_nesting_start
#endif

    /* For debug purpose, execute the timer interrupt processing here.  In
       a real system, some kind of status indication would have to be checked
       before the timer interrupt handler could be called.  */

    @ BL     _tx_timer_interrupt                  // Timer interrupt handler

    LDR      r0, =fnRAMVectors        @ Load the base of the vector table
    ADD      r14, pc, #0              @ Save return address in LR
    LDR      pc, [r0, r2, lsl #2]     @ Jump to the ISR

    LDMFD    r13!, {r14}              @ Restore lr_usr


    /* If interrupt nesting was started earlier, the end of interrupt nesting
       service must be called before returning to _tx_thread_context_restore.
       This routine returns in processing in IRQ mode with interrupts disabled.  */
#ifdef TX_ENABLE_IRQ_NESTING
    BL      _tx_thread_irq_nesting_end
#endif

    @
    @ Disable IRQ and change back to IRQ mode
    @
    CPSID    i, #MODE_IRQ
    LDR      r0, =ADDR_THRESHOLD      @ Get the IRQ Threshold
    LDR      r1, [r0, #0]
    CMP      r1, #0                   @ If priority 0
    MOVEQ    r2, #NEWIRQAGR           @ Enable new IRQ Generation
    LDREQ    r1, =ADDR_CONTROL
    STREQ    r2, [r1]
    LDMFD    r13!, {r1}
    STR      r1, [r0, #0]             @ Restore the threshold value

    /* Jump to context restore to restore system context.  */
    B       _tx_thread_context_restore

```
### Setting up SysTick Timer
AM335x has 7 timers. I am using DMTimer 4 to configure SysTick of 10ms. The platform library project consists of skeleton functions to configure, enable and disable the timer to be used as SysTick. The skeleton is available in timertick.c file. I am going to use the skeleton from platform library project as SysTick configurator. Also, remember to enable the interrupts before trying to use SysTick.

To learn how to use DMTimer, refer to dmtimer example that comes with the TI Starterware. Also, refer to the user guide that is available in the docs folder of TI Starterware.

### Suspending threads with `tx_thread_sleep`
Now, when we add `tx_thread_sleep(200)` into the two threads that we have created previously, the threads should sleep for 2 seconds.

## References

* [Bare Metal on the BeagleBone (Black and Green)][ref_1]
* [Running a Baremetal Beaglebone Black (Part 2)][ref_2]
* [AM335x Technical Reference Manual][ref_3]
* [Cortex-A8 Technical Reference Manual][ref_4]
* [ARM Cortex-A Series Programmers's Guide][ref_5]

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
[ref_3]: https://www.ti.com/lit/ug/spruh73q/spruh73q.pdf
[ref_4]: https://developer.arm.com/documentation/ddi0344/latest/
[ref_5]: https://developer.arm.com/documentation/den0013/latest/
[ti_starterware_url]: https://www.ti.com/tool/download/STARTERWARE-AM335X/02.00.01.01

