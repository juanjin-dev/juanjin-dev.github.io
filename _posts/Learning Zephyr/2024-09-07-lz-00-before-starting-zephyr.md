---
title: "[Learning Zephyr] 00. Before Starting Zephyr"
description: Getting started with Zephyr
authors: [ jin-iot ]
date: 2024-09-03 00:00:00 +0900
categories: [ Learning Zephyr ]
tags: [ Zephyr, ZephyrRTOS, Zephyr RTOS, IoT, Embedded Systems, RTOS, Realtime Operating System ]
pin: true
math: true
mermaid: true
---

 <!-- - ## Table of Contents 
     * [What Is Zephyr](#what-is-zephyr)
         * [Overview](#overview)
         * [Key Features](#key-features)
             * [Modular Architecture](#modular-architecture)
             * [Various IoT Communication Protocol Implementations](#various-iot-communication-protocol-implementations)
             * [Linux alike APIs](#linux-alike-apis)
         * [Brief Architecture](#brief-architecture)
         * [Comparison To Other Operating Systems](#comparison-to-other-operating-systems)
             * [FreeRTOS](#freertos)
             * [NuttX](#nuttx)
             * [Linux](#linux)
     * [Getting Zephyr](#getting-zephyr)
        * [Introduction To West](#downloading-the-kernel)
        * [Downloading The Kernel](#downloading-the-kernel)
        * [Installing SDKs](#installing-sdks)
        * [Switching To The LTS Kernel](#switching-to-the-lts-kernel)
     * [Prior Knowledge](#prior-knowledge)
         * [CMake](#cmake)
         * [Object Oriented C](#object-oriented-c)
         * [Kconfig](#kconfig)
         * [Device Tree](#device-tree)
         * [Source Tree](#source-tree)
             * [kernel](#kernel)
             * [arch](#arch)
             * [boards](#boards)
             * [drivers](#drivers)
             * [subsys](#subsys) -->

> Hey, I'm writing a post for the first time in here. This series posts would have a lot of misinformation, maybe. Because I'm still learning Zephyr too. But I thought my thoughts about prerequisites for beginning Zephyr would help you for those who have never used the modern embedded tools even CMake.


 - ## What Is Zephyr
     - ### Overview
        [Zephyr](https://zephyrproject.org/) is a real-time operating system. It's being maintained by [various organizations](https://zephyrproject.org/project-members) such as [Nordic Semiconductor](https://www.nordicsemi.com/About-us), [ST Microelectronics](https://www.st.com/content/st_com/en.html), etc. This RTOS has overwhelmingly large community and contributions than any other RTOS platform so many of hardware vendors added various kinds of HALs, subsystems and device drivers.

     - ### Advantages of Using Zephyr
         - #### Modular Architecture
            Zephyr is fully configurable from scheduling algorithm to user applications itself. It is because Zephyr adopted [Kconfig](https://www.kernel.org/doc/html/next/kbuild/kconfig-language.html) from the Linux kernel project. For instance if you set the config option `CONFIG_MULTITHREADING` to `n`, We can write our Applications as if it runs itself as bare metal. 
         - #### Various IoT Communication Protocol Implementations
            Zephyr itself has several protocol implementations in any of layers not only over IP. 

         - #### Linux alike APIs

     - ### Brief Architecture
        The architecture of Zephyr is not different from any other OS. You would feel the same when you see Linux or something, it's boring right.

        ```mermaid
        block-beta
            columns 4
            eapp("Embedded Application"):4
            block:kernmware("Kernel Middleware"):2
                columns 2
                syswq("System Work Queue") 
                tmr("Timer")
                drv("Device Drivers")
                pm("Power Manager")
                ipc("IPC")
                etc1("...")
            end
            block:subsys("Kernel Subsystems"):2
                columns 2
                logger("Logging System")
                netstk("Networking Stack")
                modem("RF/GNSS Modem")
                fs("File System")
                sh("Shell")
                etc2("...")
            end
            block:arch("Architecture Interface"):4
                %% columns auto (default)
                int("Interrupts") init("Privillage Management") flt("Fault Handling") etc3("...")
            end
            bl("Bootloader(MCUBoot)"):4
        ```


     - ### Comparison To Other Operating Systems
         - #### FreeRTOS
         - #### NuttX
         - #### Linux

---

 - ## Prior Knowledge
     - ### CMake
        [CMake](https://cmake.org) is the most widely used build assistant in C/C++ application development. It isn't a complete build script itself. It generates scripts to many kinds of build script languages such as Ninja, Makefile, Visual Studio, etc.

     - ### Object Oriented C
        So many people think C is obsolete and having only procedural code design. I bet they have never written, read, even one C source file.

     - ### Kconfig
        It's damn cool project configuration language ever.


     - ### Device Tree

---
 - ## Getting Zephyr
     - ### Introduction To West
     - ### Downloading The Kernel
     - ### Installing SDKs
     - ### Switching To The LTS Kernel
     - ### Source Tree
         - #### kernel
         - #### arch
         - #### boards
         - #### drivers
         - #### subsys
