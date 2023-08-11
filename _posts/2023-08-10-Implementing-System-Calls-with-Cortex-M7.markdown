---
layout: post
title:  "Implementing System Calls with Cortex-M7!"
date:   2023-08-10 17:06:18 -0500
categories: cortex-m7 operating-systems
---

*Article Created on August 10, 2023*

In this post, I will implement system calls for a Cortex-M7 microcontroller. I am using SAME70-Xplained microcontroller board, and the Microchip Studio IDE.

* PuTTY Serial Terminal:&nbsp;  &nbsp; &nbsp; &nbsp; [https://www.putty.org/][putty_url]
* Git: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;[https://github.com/git-guides/install-git][git_install_guide]

### Introduction to System Calls

### System Calls with ARM Cortex-M7

### Implementation Details
First of all, I am going to define the system call. Here, I am going to use a print functionality as an example. The function ```k_print``` is a function in the kernel-space to print to the UART console.

```c
void k_print(char* str, len str_len)
{
    // code to print the string 'str' of lenght 'str_len' to the UART console
}
```

Now, I am going to define a function which can be called by the user/application to initiate the system call. The ```u_print``` function below is a user-space equivalent function call of the ```k_print``` function. The ```u_print``` function will initiate the system call by executing the ```svc``` assembly instruction with ```0x01``` as the immediate value to the assembly instruction. Later, we will have to make sure that the value ```0x01``` corresponds to the ```k_print``` system call.

```c
void u_print(char* str, len str_len)
{
    asm volatile ("svc 0x01");
}
```

The call to ```u_print``` initiates an exception. Let's say that the exception handler to the exception initiated by ```svc``` assembly instruction is ```SVCall_Handler```. In the exception handler, we need to do the following:
1. Check if the stack in use before the supervisor call was made was ```MSP``` or ```PSP```. To check the mode from where, the supervisor call was initiated, we need to check the 4th bit of ```LR``` register.
2. Load the stack pointer (```MSP``` or ```PSP```) to ```R0``` based on the check in the step above.
3. Branch to a function (```SVCall_Handler_Main```) where the actual system call will be handled. The value in the register ```R0``` will be the first argument to the function.

The code snippet below shows the implementation of the supervisor call exception handler.

```c
void __attribute__ (( naked )) SVCall_Handler(void)
{
    asm volatile(
        "tst lr, #4                                         \n"
        "ite eq                                             \n"
        "mrseq r0, msp                                      \n"
        "mrsne r0, psp                                      \n"
        "b %[SVCall_Handler_Main]                           \n"
        :                                                   /* no output */
        : [SVCall_Handler_Main] "i" (SVCall_Handler_Main)   /* input */
        : "r0"                                              /* clobber */
    );
}

```
The ```__attribute__ (( naked ))``` makes sure that no stack is created for the function.

Now, I will implement the function where the supervisor call will be handled. The function should extract the immediate operand of the svc call (which in our case is 0x01). The ```svc_args``` argument points to the exception stack frame. From Image 1, we can see that the program counter in the exception stack frame is at an offset of 6 elements from the top. To extract the immediate value, we need to add an offset of -2 to the program counter value.

![Exception Stack Frame without floating-point storage]({{site.url}}/notes/docs/assets/images/Picture3.png)
*Image 1: Exception Stack Frame without Floating-Point Storage*

```c
void SVCall_Handler_Main(unsigned int *svc_args)
{
    unsigned int svc_number;

    svc_number = ((char *)svc_args[6])[-2];

    switch(svc_number)
    {
        case 0:
            //
            break;

        case 1:
            k_print((const char *)svc_args[0], (int)svc_args[1]);
            break;

        default:
            //
            break;
    }
}
```


### References
* [SAME70 Datasheet][same70-datasheet]
* [ARM Cortex-M7 Generic User Guide][m7-user-guide]
* [GCC Assembly Syntax][gcc-assembly-syntax]
* [GCC Assembly Syntax - Consolidated][gcc-assembly-syntax-consolidated]

[same70-datasheet]: https://www.mouser.com/datasheet/2/268/60001527A-1284321.pdf
[m7-user-guide]: https://developer.arm.com/documentation/dui0646/latest/
[gcc-assembly-syntax]: https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.
[gcc-assembly-syntax-consolidated]: https://www.felixcloutier.com/documents/gcc-asm.html