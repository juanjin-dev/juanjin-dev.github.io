---
title: "[Learning Zephyr] 00. Before Starting Zephyr"
description: Getting started with Zephyr
authors: [ jin-iot ]
date: 2024-09-09 00:00:00 +0900
categories: [ Learning Zephyr ]
tags: [ Zephyr, ZephyrRTOS, Zephyr RTOS, IoT, Embedded Systems, RTOS, Realtime Operating System ]
pin: true
math: true
mermaid: true
---

> Hey, I'm writing a post for the first time in here. This series posts would have a lot of misinformation, maybe. Because I'm still learning Zephyr too. But I thought my thoughts about prerequisites for beginning Zephyr would help you for those who have never used the modern embedded tools even OpenOCD. If you see something wrong, give me feedback.

> You may have different development environment. I'm using [Ubuntu 24.04](https://ubuntu.com/download/desktop), [Visual Studio Code](https://code.visualstudio.com) and Zephyr 3.7 LTS. In addition, I'll use [this](https://stm32-base.org/boards/STM32F411CEU6-WeAct-Black-Pill-V2.0.html) board for practice.


 - ## What Is Zephyr
     - ### Overview
        [Zephyr](https://zephyrproject.org/) is a real-time operating system. It's being maintained by [various organizations](https://zephyrproject.org/project-members) such as [Nordic Semiconductor](https://www.nordicsemi.com/About-us), [ST Microelectronics](https://www.st.com/content/st_com/en.html), etc. This RTOS has overwhelmingly large community and contributions than any other RTOS platform so many of hardware vendors added various kinds of HALs, subsystems and device drivers.

     - ### Advantages of Using Zephyr
         - #### Modular Architecture
            Zephyr is fully configurable from scheduling algorithm to user applications itself. It is because Zephyr adopted [Kconfig](https://www.kernel.org/doc/html/next/kbuild/kconfig-language.html) from the Linux kernel project. For instance if you set the config option `CONFIG_MULTITHREADING` to `n`, We can write our Applications as if it runs itself as bare metal. 

            Also you can import external Zephyr modules with just setting the `EXTRA_ZEPHYR_MODULES` CMake variable. Through this feature, your confidential source code doesn't have to be committed to the Zephyr mainline branch.

         - #### Various IoT Communication Protocol Implementations
            First, Zephyr itself has several 7th layer protocol implementations over IP. You don't have to punch your screen or keyboard anymore while porting networking libraries made for the POSIX env to embedded platforms. It has the sockets API. Also other hardware, IPC protocols.

            You can see at following links to see docs.
             - [Application Layer Protocols Over IP](https://docs.zephyrproject.org/latest/connectivity/networking/api/protocols.html)
             - [Bluetooth/LE](https://docs.zephyrproject.org/latest/connectivity/bluetooth/index.html)
             - [LoRaWAN](https://docs.zephyrproject.org/latest/connectivity/lora_lorawan/index.html)
             - [Modbus](https://docs.zephyrproject.org/latest/services/modbus/index.html)
             - [Modem](https://docs.zephyrproject.org/latest/services/modem/index.html)
             - [USB Device](https://docs.zephyrproject.org/latest/connectivity/usb/device_next/usb_device.html)
             - [ISO-TP](https://docs.zephyrproject.org/latest/connectivity/canbus/isotp.html)
             - [ZBus](https://docs.zephyrproject.org/latest/services/zbus/index.html)
             - [Apache Thrift](https://github.com/zephyrproject-rtos/zephyr/tree/v3.7-branch/modules/thrift)
             - [Thread](https://github.com/zephyrproject-rtos/zephyr/tree/v3.7-branch/modules/openthread)
             - [CANOpen](https://github.com/zephyrproject-rtos/zephyr/tree/v3.7-branch/modules/canopennode)
             - And some other protocols on third-party external modules
                 - [micro-ROS](https://github.com/micro-ROS/micro_ros_zephyr_module)
                 - [Zigbee](https://github.com/nrfconnect/sdk-nrf/tree/main/subsys/zigbee)

         - #### Initialization Automation
            Perhaps the Linux Foundation people hated creating C header files or meaningless repetitive labor, so this dark magic was made. If you have experienced writing kernel modules, once you use some macros like `module_init(...)` or `module_exit(...)` in the module source code, this let it run so magically thanks to the linker scripts.

         - #### Relatively Easy Porting
            If you want to port a board in Zephyr, Just copy the reference board directory and modify `your_board.dts` and `your_board_defconfig` files. Sounds easy right? ~~(I'm lying.)~~ Surely some efforts and slamming on our keyboards are needed in any software projects.

     - ### Brief Architecture
        The architecture of Zephyr is not different from any other OS. You would feel the same when you see Linux or something, it's boring right. ~~(Just I wanted to practice the mermaid block diagram on it.)~~

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
         - #### [FreeRTOS](https://aws.amazon.com/freertos)
            FreeRTOS is a bit small, well maintained operating system. A quite different thing from Zephyr is device drivers. It has no common device driver model. So You need to use their SoC platform drivers or something. I think it's a pain for people who porting their own applications to other SoCs.

         - #### [NuttX](https://nuttx.apache.org)
            As far as I know, it's unrivaled POSIX-compatible real-time operating system than other ones. And countless UAVs are equipped with this. But its less stable platform drivers, especially for non-ARM SoCs, I have so bad memories regarding this OS. Once you try to port your own production board from the evaluation board, you should inspect a pile of its platform driver and macro definitions. It feels like meh.

         - #### [Linux (Embedded)](https://elinux.org)
            People who experienced Linux first, they must feel pretty comfy. Because Zephyr has similar kernel APIs. I think the retraining costs associated with switching platforms will be enormous, as you may expect.

---

 - ## Prior Knowledge
    First, I recommend you to follow [this](https://docs.zephyrproject.org/latest/develop/getting_started/index.html) guide before we practice the following contents. 

     - ### CMake
        [CMake](https://cmake.org) is the most widely used build assistant in C/C++ application development. It isn't a complete build system itself. It generates script outputs in several kinds of build script languages such as Ninja, Makefile, Visual Studio, etc. Also Zephyr adopted CMake + [Ninja](https://ninja-build.org) as its build system.

     - ### Object Oriented C
        So many people think C is obsolete and has only procedural code design. I bet they have never written, read, even one C source file. Not having `class` nor `override` reservation words doesn't mean it is impossible to write in the OOP way.

     - ### Kconfig
        It's damn cool project configuration language ever.


     - ### Device Tree
        Device Tree language is a hardware description language. It was in the Linux kernel for [Open Firmware](https://docs.kernel.org/devicetree/usage-model.html) devices and its document is explaining well.

 - ## Development Environment Configuration
     - ### Switching to the LTS kernel and modules

     - ### Installing Visual Studio Code Extensions

     - ### Building and Debugging

 - ## Appendix
     - ### Creating your own host application project with CMake
        I dare to assume that you may sometimes have to work on applications in C/C++. So I'll let you know how to create a CMake project from scratch.

         1. **Creating the project directory and source files**

            I suppose you are using bash and installed the GNU host toolchain already. Execute these commands.

            ```bash
            # set the environment variable YOUR_WORKSPACE_DIRECTORY
            cd ${YOUR_WORKSPACE_DIRECTORY}
            mkdir -p my-cmake-project/src
            cd my-cmake-project
            touch CMakeLists.txt .gitignore src/main.c src/CMakeLists.txt
            tree

            # .
            # ├── CMakeLists.txt
            # └── src
            #     ├── CMakeLists.txt
            #     └── main.c
            #
            # 2 directories, 3 files
            ```
        2. **Editing the root CMakeLists.txt file**

            > my-cmake-project/CMakeLists.txt

            ```cmake
            cmake_minimum_required(VERSION 3.25 FATAL_ERROR)
            project(my-cmake-project
                    VERSION 0.1.0
                    DESCRIPTION "My CMake Project"
                    HOMEPAGE_URL https://jin-iot.github.io
                    LANGUAGES C)
            message(STATUS "This is ${PROJECT_NAME}")
            ```

        3. **Configuring your project**

            Execute the following command.

            ```bash
            cmake -B build -DCMAKE_BUILD_TYPE=Debug -GNinja .

            # -- The C compiler identification is GNU 13.2.0
            # -- Detecting C compiler ABI info
            # -- Detecting C compiler ABI info - done
            # -- Check for working C compiler: /usr/bin/cc - skipped
            # -- Detecting C compile features
            # -- Detecting C compile features - done
            # -- This is my-cmake-project
            # -- Configuring done (4371098972.0s)
            # -- Generating done (7437981234.0s)
            # -- Build files have been written to: ♥♥♥♥♥♥
            ```

        4. **Completing the application source**

            > my-cmake-project/CMakeLists.txt

            ```cmake
            cmake_minimum_required(VERSION 3.25 FATAL_ERROR)
            project(my-cmake-project
                    VERSION 0.1.0
                    DESCRIPTION "My CMake Project"
                    HOMEPAGE_URL https://jin-iot.github.io
                    LANGUAGES C)
            message(STATUS "This is ${PROJECT_NAME}")

            # Add this line
            add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/src)
            ```

            > my-cmake-project/src/CMakeLists.txt

            ```cmake
            set(MY_APPLICATION_SOURCES main.c)
            add_executable(my-application ${MY_APPLICATION_SOURCES})
            ```

            > my-cmake-project/src/main.c

            ```c
            #include <stdio.h>
            #include <stdlib.h>

            int main(int argc, char **argv) {
                printf("This is so easy\n");
                return EXIT_SUCCESS;
            }
            ```

        5. **Building the project**

            ```bash
            cmake --build ./build/

            # [1/2] Building C object src/CMakeFiles/my-application.dir/main.c.o
            # [2/2] Linking C executable src/my-application
            ```             



 - ## Conclusion
    Have no words to say. Thank you for reading.