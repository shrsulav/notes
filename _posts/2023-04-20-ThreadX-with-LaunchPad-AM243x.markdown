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

[lpam243x_url]: https://www.ti.com/tool/LP-AM243
[lpam243x_mcu_sdk_url]: https://www.ti.com/tool/MCU-PLUS-SDK-AM243X
[threadx_url]: https://github.com/azure-rtos/threadx
[ccstudio_url]: https://www.ti.com/tool/CCSTUDIO
[putty_url]: https://www.putty.org/
[git_install_guide]: https://github.com/git-guides/install-git
[mcu_getting_started_guide]: https://software-dl.ti.com/mcu-plus-sdk/esd/AM243X/08_04_00_17/exports/docs/api_guide_am243x/index.html