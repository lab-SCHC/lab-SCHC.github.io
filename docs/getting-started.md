<!-- Breadcrumbs -->
< [Lab.SCHC FullSDK Documentation](/#) /
<!-- /Breadcrumbs -->

*On this page:*

<!-- TOC -->

- [Getting Started](#getting-started)
    - [Requisites and Compatibility](#requisites-and-compatibility)
        - [LNS](#lns)
        - [SCHC Gateway](#schc-gateway)
    - [Development environment](#development-environment)
        - [Install the GNU ARM Toolchain](#install-the-gnu-arm-toolchain)
        - [Additionnal dependencies](#additionnal-dependencies)
    - [Building applications](#building-applications)
        - [Visual Studio Code CMake integration](#visual-studio-code-cmake-integration)
    - [Erasing the microcontroller](#erasing-the-microcontroller)
    - [Flashing the microcontroller](#flashing-the-microcontroller)
    - [Debugging](#debugging)
        - [Debugging on CLI](#debugging-on-cli)
        - [Debugging on VS Code](#debugging-on-vs-code)
        - [Debugging on STM32CubeIDE](#debugging-on-stm32cubeide)

<!-- /TOC -->

# Getting Started

In this section you will find all the information you need to start developing
applications using our SDK. Make sure you have the supported hardware and
software, although developers are encouraged to make their own integrations.

A practical and comprehensive how-to guide is also presented to help newcomers
configure, flash, and deploy their own applications implementing lab.SCHC
FullSDK.

## Requisites and Compatibility

The hardware, operating systems, libraries, and third-party software in this
section are recommendations for optimal use.

| Support | Recommendation |
| --- | --- |
| OS | The library can be integrated into any OS (Bare Metal, RTOS). The integration is made by the system integrator (or embedded system developers). |
| Hardware/Board | ST NUCLEO-L476RG micro-controller. |
| Hardware/Shield | SX1272, SX1276. |
| Hardware/Serial | Serial port terminal such as `minicom`. |
| L2A | The level-two adaptation (L2A) layer is usually provided by the integrator and should implement the (L2A) interface of the SDK. |

### LNS

The current version of lab.SCHC FullSDK focuses on enabling SCHC for LoRaWAN
networks. To test the SDK you will need to register a device on the LNS of your
preference. We recommend using Actility's [ThingPark
Community](https://community.thingpark.org/).

### SCHC Gateway

The SCHC Gateway is a cloud-based platform running within the network and is the
entity in charge of converting SCHC packets into IP packets. It acts as a
middlebox between the constrained network and the application server, enabling
the client application to communicate with the end devices using the IP
protocol.

The current version of lab.SCHC FullSDK is fully compatible with Actility's
[SCHC IPCore](#todo). Compatibility with IMT Atlantique's
[OpenSCHC](https://github.com/openschc/openschc) is currently in development.
Stay tuned!

## Development environment

Setting up a development environment is easy and flexible. You will need to
clone the [lab.SCHC FullSDK
Delivery](https://github.com/lab-SCHC/full-sdk-delivery) repository and
initialize its submodules. Then, you can open the code base using your editor of
choice, such as [Visual Studio Code](https://code.visualstudio.com/) or
[STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html).

```sh
git clone https://github.com/lab-SCHC/full-sdk-delivery.git
cd full-sdk-delivery
git submodule update --init --recursive
```

### Install the GNU ARM Toolchain

An ARM cross compiler is required in order to build the firmware that will be
flashed on the devices. It can be retrieved and installed with the following
commands:

```sh
cd /opt
su
wget -O archive.tar.bz2 "https://developer.arm.com/-/media/Files/downloads/gnu/11.2-2022.02/binrel/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi.tar.xz"
tar xf archive.tar.bz2 -C /opt
```

### Additionnal dependencies

The LoRa Semtech stack source tree is also required in order to build the
firmware. It can be retrieved at the following URL:
https://www.st.com/en/embedded-software/i-cube-lrwan.html

Once downloaded, it needs to be extracted at a place where the compiler will
find it.

```sh
cd /opt
sudo unzip en.i-cube_lrwan.zip
```

## Building applications

Lab.SCHC FullSDK is designed to build applications using
[CMake](https://cmake.org/) (>= 3.21).
[OpenOCD](https://openocd.org/pages/about.html) **version 0.10.0** needs to be
installed for our cmake files to run properly.


The `examples/` directory contains some example applications implementing
lab.SCHC FullSDK. In order to build an example application, you can run the
following command:

```sh
cmake -S . -B ./build -DAPP_NAME=<app_name> -DTOOLCHAIN=<toolchain> -DTARGET=<target> -DL2_STACK=<l2_stack> && make -C ./build
```

where:

- `app_name` is the name of the application in the examples folder
- `toolchain` and `target` corresponds to the compilation toolchain and the
  associated target to be used. It should be one of the supported
  toolchain/target in the `full-sdk/toolchains/` folder.
- `l2_stack` is the type of L2 stack to be used. It should be one of the
  supported L2 stacks listed in the `l2` folder

To compile an application in debug mode, you need to set `-DDEBUG_ENABLED=ON`.

For some applications, additional environment variables need to be set. You can
check the README file of the corresponding application for specific build
instructions.

### Visual Studio Code CMake integration

Integrating CMake into Visual Studio Code IDE will ease the compilation process,
directly through the IDE, instead of compiling through a terminal.

1. Be sure to have **CMake** and **CMake Tools** extensions installed.
2. Add an ARM GCC compiler for cross-compilation: search for “CMake: Edit
   User-Local CMake Kits” on the top tab using the `CTRL+MAJ+P` command. A
   "cmake-tools-kits.json" is opened.
3. Add the specific toolchain for your project and for your target. 

```json
  {
    "name": "GCC 11.2 ARM",
    "compilers": {
      "C": "/opt/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gcc"
    },
    "isTrusted": true
  }
```

4. Configure project compilation by editing (or creating) the
   `.vscode/settings.json` file and adding a new `cmake.configureArgs` field to
   specify compilation definitions.

```json
{
    "cmake.configureArgs": [
        "-DPLATFORM=STM32L476RG-Nucleo",
        "-DTARGET=m4",
        "-DL2_STACK=semtech",
        "-DTOOLCHAIN=gcc-arm-none"
    ]
}
```

5. Compile using the CMake commands:
    - Clean with `CTRL+MAJ+P` -> "Cmake: Clean".
    - Build with `CTRL+MAJ+P` -> "Cmake: Build".

## Erasing the microcontroller

The memory of the microcontroller can be erased by executing the following
command in the repository directory:

```sh
OPENOCD_TARGET=<openocd.cfg> make -C openocd/ erase
```

where `openocd.cfg` is the ".cfg" file in the openocd directory that is made for
the selected microcontroller architecture.

## Flashing the microcontroller

The executable file of the example can be flashed into the microcontroller by
executing the following command in the repository directory:

```sh
OPENOCD_TARGET=<openocd.cfg> BIN_FILE=build/<toolchain>/<target>/<app_name>.bin make -C openocd/ flash
```

## Debugging

> To compile an application in debug mode, you need to set `-DDEBUG_ENABLED=ON`.

It is possible to debug applications in the full-sdk-delivery repository
(full-sdk-delivery/examples/<example_name>) in three ways:

- debugging with OpenOCD and GDB on CLI.
- debugging with OpenOCD and GDB on Visual Studio Code.
- debugging with STM32CubeIDE (recommended).

The main interest of STM32CubeIDE compared to Visual Studio Code is that it
allows to set watchpoints. VS Code supports an equivalent mechanism, data
breakpoints, but it appears that this mechanism is not supported in current
debugger configuration.


> **_NOTE_**: To generate the ELF file, copy the application file
> `build/<toolchain>/<platform>/<app>` as `<app.elf>`.

### Debugging on CLI

1. Start OpenOCD GDB server:

```sh
OPENOCD_TARGET=<openocd.cfg> make -C openocd/ debug
```

2. Start GDB client with the absolute path of the executable:

```sh
/opt/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb /path/to/full-sdk-delivery/build/<platform>/<example_name>.elf
```

3. Connect to OpenOCD GDB server:

```
target remote localhost:3333
```

4. Put microcontroller into reset state:

```
monitor reset halt
```

5. Load the executable into the microcontroller:

```
load
```

6. Standard GDB commands can now be executed:

```
continue
```

### Debugging on VS Code

1. Install [Cortex-Debug](https://github.com/Marus/cortex-debug/tree/V0.4.1)
   extension on Visual Studio Code Marketplace (version 0.4.1, as the latest one
   is not working).

2. Edit or create the `.vscode/launch.json` configuration file with the
   following content:

```json
{
  "type": "cortex-debug",
  "request": "launch",
  "servertype": "openocd",
  "cwd": "${workspaceRoot}",
  "name": "STM32 Debug",
  "armToolchainPath": "/opt/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin",
  "executable": "",
  "configFiles": [
    ""
  ]
}
```

3. Set the *executable* field of the launch configuration with the relative path
   of the executable of the example application that will be debugged:

```json
"executable": "./build/<platform>/<example_name>.elf"
```

4. Set the *configFiles* field of the launch configuration with the absolute
   path of the configuration file of the related board:

- For B-L072Z-LRWAN1 board:
```json
"configFiles": [
    "/usr/share/openocd/scripts/board/stm32l0discovery.cfg"
]
```

- For NUCLEO-L476RG board:
```json
"configFiles": [
    "/usr/share/openocd/scripts/board/stm32l4discovery.cfg"
]
```

5. Start debugging by *Debug -> Start Debugging (F5) on VS Code menu bar.


### Debugging on STM32CubeIDE

The only prerequisite is STM32CubeIDE (STM32CubeMX is required only when
creating a project).

1. Build the application for Debug as explained previously.

2. Create a new, simple STM32 project for the target board being used.

3. Create a debug configuration, and verify that the project runs in debug mode.

4. In the debug configuration, set the *C/C++ Application* field to the absolute
   path to the ELF file corresponding to the application to debug.

> C/C++ Application:  
> `/path/to/full-sdk-delivery/build/<toolchain>/<platform>/<app.elf>`

The ELF file embeds links to source code files, so there is no need to declare
them in STM32CubeIDE.

To set a breakpoint, open a source file using the *File > Open File…* menu, and
mark the line by double clicking next to the line number.
