---
title: "[Learning Zephyr] 000. Before Starting Zephyr"
description: Setting up the development environment
authors: [ juanjin-dev ]
date: 2025-02-11 00:00:00 +0900
categories: [ Learning Zephyr ]
tags: [ Zephyr, ZephyrRTOS, Zephyr RTOS, IoT, Embedded Systems, RTOS, Realtime Operating System ]
pin: true
math: true
mermaid: true
---

> Hey, I'm writing a post for the first time in here. If you see something wrong, give me feedback.

> You may have different development environment. I'm using [Ubuntu 24.04](https://ubuntu.com/download/desktop), [Visual Studio Code](https://code.visualstudio.com) and [Zephyr 4.0](https://github.com/zephyrproject-rtos/zephyr/tree/v4.0-branch). In addition, I'll use [Blackpill F411CE](https://stm32-base.org/boards/STM32F411CEU6-WeAct-Black-Pill-V2.0.html) board for practice.

---

## Overview

[Zephyr](https://zephyrproject.org) is a real-time operating system. It's being maintained by [various organizations](https://www.linuxfoundation.org) and vendors such as [Nordic Semiconductor](https://www.nordicsemi.com/About-us), [ST Microelectronics](https://www.st.com/content/st_com/en.html), etc. This RTOS has overwhelmingly large community and contributions than any other RTOS platform so many of hardware vendors added various kinds of HALs, subsystems and device drivers.

Before starting Zephyr, we need to set up our development environment. Despite of the official [Getting Started Guide](https://docs.zephyrproject.org/latest/develop/getting_started/index.html) already exists, I found the better way for myself. It's up to you whether you want to follow the guide above or not.

---

## 1. Getting Ready

---

### 1.1 Installing West

West is a meta CLI tool for Zephyr written in Python. once you install it, you can build applications, flash firmware and anything else. At first you need to enter a Python virtual environment to prevent python library collision. If you don't have the venv package, install it with the shell command `sudo apt install python3-dev python3-pip python3-setuptools python3-tk python3-wheel python3-venv`.

```bash
python3 -m venv /path/to/create/zephyr-venv
source /path/to/create/zephyr-venv/bin/activate
pip install west
west --help
# Now you see the help output like this
# usage: west [-h] [-z ZEPHYR_BASE] [-v] [-q] [-V] <command> ...
#
# The Zephyr RTOS meta-tool.
#
# optional arguments:
#   -h, --help            get help for west or a command
#   -z ZEPHYR_BASE, --zephyr-base ZEPHYR_BASE
#                         Override the Zephyr base directory. The default is
#                         the manifest project with path "zephyr".
#   -v, --verbose         Display verbose output. May be given multiple times
#                         to increase verbosity.
#   -q, --quiet           Display less verbose output. May be given multiple
#                         times to decrease verbosity.
#   -V, --version         print the program version and exit
# ...
```
---

### 1.2 Installing the Kernel and Modules

Now we're ready to download the kernel and SDKs. The west command will manage the kernel and modules with git. In addition, To build applications we need CMake and some tools such as devicetree compiler.

```bash
wget https://apt.kitware.com/kitware-archive.sh
sudo bash kitware-archive.sh

sudo apt install --no-install-recommends git cmake ninja-build gperf \
    ccache dfu-util device-tree-compiler wget xz-utils file \
    make gcc gcc-multilib g++-multilib libsdl2-dev libmagic1

cmake --version
python3 --version
dtc --version
```

And, Create a base directory and then initialize at there.

```bash
mkdir /path/to/zephyrproject
cd /path/to/zephyrproject
west init # Downloading the kernel
west update # Initializing modules
west zephyr-export # Exporting cmake modules
# Installing additional Python packages the additional west commands depends on
pip install -r ./zephyr/scripts/requirements.txt 
```
---

### 1.3 Installing the Zephyr SDK

The official guide tells us to install the SDK via the west command `west sdk install`, but I'm going to download it myself and install it wherever I want. It's up to you whether you follow the official guide or not.

```bash
mkdir /path/to/zephyrproject/sdk
cd /path/to/zephyrproject/sdk

# Downloading toolchains for all cpu architecture and host tools
wget wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.17.0/zephyr-sdk-0.17.0_linux-x86_64.tar.xz
wget -O - https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.17.0/sha256.sum | shasum --check --ignore-missing

tar -xvf zephyr-sdk-0.17.0_linux-x86_64.tar.xz
rm zephyr-sdk-0.17.0
cd zephyr-sdk-0.17.0
bash setup.sh -t all -h -c
```

---

### 1.4 Building and Installing OpenOCD

[OpenOCD](https://openocd.org) is an opensource GDB server for hardware debuggers like ST-Link or something. We can run debug over this server. I'm not gonna use OpenOCD which I downloaded from the Zephyr SDK. Instead, I'm gonna build from source.

```bash
# Some libraries driving debug probes
sudo apt install libusb-1.0-0-dev libftdi-dev libstlink-dev libjaylink-dev libgpiod-dev
cd /path/to/install
git clone git://git.code.sf.net/p/openocd/code openocd -b v0.12.0
cd openocd
./bootstrap

# Enable all debuggers
./configure --prefix=/usr --enable-rshim --enable-ftdi --enable-stlink --enable-ti-icdi \
  --enable-ulink --enable-usb-blaster-2 --enable-ft232r --enable-vsllink \ 
  --enable-xds110 --enable-cmsis-dap-v2 --enable-osbdm --enable-opendous \
  --enable-armjtagew --enable-rlink --enable-usbprog --enable-esp-usb-jtag \
  --enable-cmsis-dap --enable-nulink --enable-kitprog --enable-usb-blaster \
  --enable-presto --enable-openjtag --enable-buspirate --enable-jlink --enable-aice \
  --enable-parport --disable-parport-ppdev --enable-parport-giveio --enable-jtag_vpi \
  --enable-vdebug --enable-jtag_dpi --enable-amtjtagaccel --enable-bcm2835gpio \
  --enable-imx_gpio --enable-am335xgpio --enable-ep93xx --enable-at91rm9200 \
  --enable-gw16012 --enable-sysfsgpio --enable-xlnx-pcie-xvc --enable-jimtcl-maintainer \
  --disable-internal-libjaylink --enable-remote-bitbang

make -j$(nproc)
sudo make install

# udev rules for detecting debuggers
sudo cp ./contrib/60-openocd.rules /etc/udev/rules.d/
sudo udevadm control --reload
```

All done. Now we're gonna test via building and flashing a sample application. 

---

## 2. Running a Sample Application

---

### 2.1 Building a Sample Application

You are now in the Zephyr python virtual environment, An environment variable named `$ZEPHYR_BASE` indicates where's the Zephyr kernel at. If you find it annoying to have to enter the Python virtual environment every time, set your shell alias like `alias zenv="source /path/to/zephyr-env/bin/activate"`.

Look around the Zephyr kernel directory. You see the `samples` directory? If you have problems to code applications or don't know use cases of modules, Get helped from the samples. There are several basic samples like blinking LEDs at `samples/basic`.

We're now gonna build the `samples/basic/blinky` application. The official guide uses the `west build` command, but I rather prefer usnig `cmake`. Build with the following commands.

```
cd $ZEPHYR_BASE

# You can see the board info I currently use
# at the boards/weact/blackpill_f411ce directory.
cmake -GNinja -DNO_BUILD_TYPE_WARNING=ON -DBOARD=blackpill_f411ce \
  -S./samples/basic/blinky -Bbuild
# Loading Zephyr default modules (Zephyr base).
# -- Application: /home/jin/Applications/zephyrproject/zephyr/samples/basic/blinky
# -- CMake version: 3.31.5
# -- Found Python3: /home/jin/Applications/venvs/zephyr/bin/python (found suitable version "3.12.3", minimum required is "3.10") found components: Interpreter
# -- Cache files will be written to: /home/jin/.cache/zephyr
# -- Zephyr version: 4.0.99 (/home/jin/Applications/zephyrproject/zephyr)
# ...

cmake --build build -j$(nproc)
# [1/142] Preparing syscall dependency handling
#
# [3/142] Generating include/generated/zephyr/version.h
# -- Zephyr version: 4.0.99 (/home/jin/Applications/zephyrproject/zephyr), build: v4.0.0-4972-ge4c16b5433cd
# [4/142] Generating ../build_info.yml
# ...
```

Done. If you have built the application without problems, you can see the output binaries are located at `build/zephyr`.

 - `build/zephyr/zephyr.bin`
 - `build/zephyr/zephyr.hex`
 - `build/zephyr/zephyr.elf`

Be noticed the `zephyr.bin` file is used for flashing. the `zephyr.elf` file contains sections, stataic data, symbol names and debug symbols if you have built with debug option. So it's used for debugging.

---

### 2.2 Flashing Firmware Binary

Now we can flash the binary. The official guide uses the `west flash` command, but I rather prefer usnig `openocd`. OpenOCD is a debug server but it's also used to flash firmware. Flash with the following command.

```bash
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg \
  -c "program build/zephyr/zephyr.bin 0x08000000 reset verify exit"
# Open On-Chip Debugger 0.12.0-g9ea7f3d-dirty
# Licensed under GNU GPL v2
# For bug reports, read
#         http://openocd.org/doc/doxygen/bugs.html
# Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
# ...
# ** Programming Started **
# Info : device id = 0x10006431
# Info : flash size = 512 KiB
# ** Programming Finished **
# ** Verify Started **
# ** Verified OK **
# ** Resetting Target **
# Info : Unable to match requested speed 2000 kHz, using 1800 kHz
# Info : Unable to match requested speed 2000 kHz, using 1800 kHz
# shutdown command invoked
```

You see the board's LED blinking? Nice work.

---

## Conclusion

Thanks for reading my article. Next time I'll let you know how to set up Visual Studio Code development environment for Zephyr.

---
