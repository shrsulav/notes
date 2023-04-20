---
layout: post
title:  "ThreadX with LaunchPad AM243x!"
date:   2021-12-23 17:33:18 -0500
categories: jekyll update
---
In this post, I will write about how I built ThreadX (Azure RTOS) for TI LaunchPad AM243x platform. I am using Windows 11 OS host machine to setup cross-development environment for LP-AM243x. Here's the list of a few things that we will need to get started.

* TI LaunchPad AM243x: &nbsp; &nbsp;&nbsp;&nbsp; [https://www.ti.com/tool/LP-AM243][lpam243x_url]
* MCU SDK for LP-AM243x: &nbsp;[https://www.ti.com/tool/MCU-PLUS-SDK-AM243X][lpam243x_mcu_sdk_url]
* ThreadX: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[https://github.com/azure-rtos/threadx][threadx_url]
* Code Composer Studio: &nbsp; &nbsp; [https://www.ti.com/tool/CCSTUDIO][ccstudio_url]
* PuTTY Serial Terminal:&nbsp;  &nbsp; &nbsp; &nbsp; [https://www.putty.org/][putty_url]
* Git: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;[https://github.com/git-guides/install-git][git_install_guide]

### Steps
#### Setting up the Development Environment
Setting up the LP-AM243x evaulation board involves a number of steps. We need to install the CCStudio IDE, Git and PuTTY Serial Terminal. Install the MCU SDK. By default, the MCU SDK will be installed in the path `C:\ti`. So, go ahead and install the MCU SDK in the default directory. Installing the MCU SDK in the default directory will be cumbersome when we need some version control on the source files included in the SDK. So, we will reinstall the MCU SDK in the git repository later on. We will consider that the Development Environment has been setup when we can run `hello-world` example program from the MCU SDK. To reach till that state, follow the getting-started guide from the [MCU SDK Readme Documentation][mcu_getting_started_guide]. At the time of writing, the latest version of MCU SDK for LP-AM243x is 08.05.00.24. However, I had some issues with lwIP networking stack with this version. So, the version I am using is 08.04.00.17. Follow the getting-started guide, import `hello-world-nortos` example project in CCStudio, and make sure that you are able to see "Hello World!" printed on the serial terminal.

#### Setting up Git Repository for the Project
This is an important step as we will need version control to keep track of the changes that we will be doing throught the project. So, create a project in Github, and initialze the project with a readme file. We will keep our CCStudio project directory separate from our source files so, create a directory named `ccs` in the git repository root. We will use this folder as workspace for CCStudio Project. Open CCStudio with this folder as the workspace and import the `hello-world-nortos` example project. Make sure the project builds and runs fine. Confirm the fact by checking the "Hello World!" print in the serial terminal. The directory `ccs` will have a lot of files and folders that we do not to keep track of. Add such files/folders in the .gitignore file. Commit the files in the repository. This is going to be our first commit.

Next, create a folder named `src`. This is where we will keep our source files for ThreadX and MCU SDK. Download ThreadX with tag `v6.2.1_rel` (the latest release build available for ThreadX at the time of writing) from the ThreadX Github repository into the `src` folder. Then, reinstall MCU SDK in the `src` folder. We will be modifying source files from ThreadX and MCU SDK so, let's go ahead and commit them first. This is going to be our second commit.

#### Setting up CCS project for development
As you might have noticed, at this point, we have MCU SDK installed in two directories - one in the default directory and one in the project repository. The CCS project is configured to use the MCU SDK in the default directory. Delete this CCS project from the workspace. Follow the "Installing SDK at non-default location" guide in the MCU SDK Readme Documentation to setup CCStudio to use MCU SDK installed in our repository. Build the libraries as guided in "Using SDK with Makefiles" page in the MCU SDK Readme. Import the `hello-world` project from the MCU SDK in our repository. Make sure the project builds and runs.

Editing and building the MCU SDK libraries outside of CCStudio can be cumbersome. So, we will import the MCU SDK makefile project into CCStudio. We need to make sure that debugging is enabled in the MCU SDK project and it builds right.

Go ahead and commit the changes till this point.

#### Mode of Operation for ThreadX
The MCU SDK provides a lot of examples for no-RTOS and FreeRTOS bases. If we check the source code related to bootup, we will see that when the execution of reaches `main()`, the processor is operating in the `SYSTEM` mode. The FreeRTOS kernel works in the `SYSTEM` mode, whereas the FreeRTOS threads work in the `SVC` mode. For ThreadX, the kernel and the threads both work in the `SVC` mode. So, we have two choices (maybe we have more but I am limiting myself to the two at the moment). One is that we let the mode be `SYSTEM` when execution reaches `main()` and then later change the mode to `SVC`. The other choice is that we start `main()` with `SVC` mode. I have taken the second choice. To implement the choice I have taken, change the `boot` file such that after setting up the stacks for different modes, the last mode is `SVC`. You will notice that by default, the last mode while setting up stacks is `SYSTEM`. So, make this change and run the program. When the execution reaches `main()`, check the `CPSR` register in the debug window to make sure that the mode is `SVC`.

After making the change, you will notice that the debug prints no longer works. We will fix that later on.

#### Integrating ThreadX
ThreadX supports a lot of processor architectures, as you can confirm by checking the "ports" folder. We want only Cortex-R5 so, delete the rest of the ports. In Cortex-R5, we will see support for multiple compilers. We will retain just GNU and delete the rest. Import the ThreadX source directory in the CCS project by creating a symbolic link. Add the `tx_kernel_enter()` function after the point the drivers are setup. Define `tx_application_define` function. At this point, the function can be empty. Build the project. You will see that it does not build. The error log shows that it is not able to find a few symbols. Those symbols come from the `_tx_initialize_low_level.S`.

The low level initialization in ThreadX, sets up the stacks for different processor modes and initializes a few variables. We do not need to set up stacks for different processor modes as we have already set them up before `main()`. So, we can safely comment out the code to setup the stacks. Next, we need to intialize a few variables - `_tx_thread_system_stack_ptr` and `_tx_initialize_unused_memory`. The `_tx_thread_system_stack_ptr` variable needs to point to the top of the `SVC` mode stack. The top of the `SVC` mode stack is pointed to by the variable `__SVC_STACK_END` defined in the linker script. The `_tx_initialize_unused_memory` variable needs to point to the highest RAM address which can be used to setup stacks for ThreadX threads and other ThreadX objects like queues, byte pools, etc. In our case, the highest RAM address which has can be used for ThreadX constructs is the one pointed to by `__UNDEFINED_STACK_END` defined in the linker script.

After making these changes, build the project and make sure that it builds fine.

#### Testing ThreadX with Two Threads
To test if the ThreadX kernel works, define two threads which do nothing but increment a variable. Since the UART print is not working, we cannot print anyting onto the console yet. Synchronize the threads in such a way that the threads wait for a semaphore (a different semaphore for each thread), and once the variable is incremented, releases semaphore for the other thread, so that the other thread can run. Build the program and make sure that the threads are running, by putting break points on the two threads. The threads should increment the variables alternatively.

#### Taking Care of the Interrupts
Till now we have not handled interrupts. We have not setup SysTick timer interrupt for the kernel and UART print is not working.

Interrupt is a type of exception in Cortex-R5. The interrupt exception handler can be found in the `_vectors` construct. `_vectors` is defined in the file `HwiP_armv7r_vectors_nortos_asm.S`. The interrupt handler is defined to be `HwiP_irq_handler`. The `HwiP_irq_handler` handler, saves the context of execution before actually handling the interrupt, then handles the interrupt in the `HwiP_irq_handler_c` function, and then restores the context after handling the interrupt. ThreadX has its own interrupt handler - `__tx_irq_handler`. So, we need to change the interrupt handler from `HwiP_irq_handler` to `__tx_irq_handler`. This change needs to be done `HwiP_armv7r_vim.c` file too.

The `__tx_irq_handler` handler is written to run `_tx_timer_interrupt` on every interrupt exception instead of handling the actual interrupt. To fix that, we need to branch to `HwiP_irq_handler_c` instead of `_tx_timer_interrupt`.

#### Setting up SysTick Timer

[lpam243x_url]: https://www.ti.com/tool/LP-AM243
[lpam243x_mcu_sdk_url]: https://www.ti.com/tool/MCU-PLUS-SDK-AM243X
[threadx_url]: https://github.com/azure-rtos/threadx
[ccstudio_url]: https://www.ti.com/tool/CCSTUDIO
[putty_url]: https://www.putty.org/
[git_install_guide]: https://github.com/git-guides/install-git
[mcu_getting_started_guide]: https://software-dl.ti.com/mcu-plus-sdk/esd/AM243X/08_04_00_17/exports/docs/api_guide_am243x/index.html