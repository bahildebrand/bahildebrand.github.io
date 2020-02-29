---
layout: post
author: Blake Hildebrand
title: Black Magic Debugging
excerpt: "The black magic probe is an open source JTAG/SWD debugger for ARM micro-controllers. It hosts a GDB remote server accessible over a serial connection to easily interface with GDB clients. The project provides support for debugging and flashing for a growing number of arm micro-controllers."
categories: [debugging]
comments: true
---

# GDB Debugging With Black Magic

The [black magic probe](https://github.com/blacksphere/blackmagic/wiki) is an open source JTAG/SWD debugger for ARM micro-controllers. It hosts a [GDB](https://www.gnu.org/software/gdb/) remote server accessible over a serial connection to easily interface with GDB clients. The project provides support for debugging and flashing for a growing number of arm micro-controllers.

## Prerequisites

### Acquiring Hardware

First things first you'll need to get some [debugging hardware](https://github.com/blacksphere/blackmagic/wiki#getting-hardware). There are quite a few options for pre-made probes, and some options to make your own for those feeling a bit more adventurous.

You will also need a target to debug. For this tutorial I'm using the [Particle Xenon](https://www.adafruit.com/product/3995). Note that this board has been [discontinued](https://docs.particle.io/reference/discontinued/mesh/). If you're unable to purchase it any nrf52840 board will do. I would recommend the [Adafruit Feather nRF52840 Express](https://www.adafruit.com/product/4062).

After you acquire your probe and target hardware we can start begin installing the software dependencies needed to get started. On an Ubuntu machine install the following packages:

```bash
sudo apt install gcc-arm-none-eabi gdb-multiarch make gcc-arm-none-eabi
```

### Building Our Application

Before we try to connect to a board, we're going to need to put something on it. The program that we're going to use is a simple [blinky](https://github.com/bahildebrand/nrf52840-blinky) app. This app will periodically blink the green LED on the board.

To compile the code simply run `make`. A build directory should be created containing all the compiled and linked programs.

## Connecting to Target

First we need to start our GDB session:

```
gdb-multiarch
```

As mentioned earlier the BMP provides us with a remote GDB server over serial. Now that we have started our GDB sessions we need to connect to the probe. The following command will connect to the serial port of the BMP:

```
target extended-remote /dev/ttyACM0
```

We are now connected to our probe! We're not quite done yet, however. We still need to instruct our probe to attach to our board. Running the following command will give us some information about connected device.

```
monitor swdp_scan
```

Note that this is for targets using the [SWD](https://developer.arm.com/architectures/cpu-architecture/debug-visibility-and-trace/coresight-architecture/serial-wire-debug) interface. For [JTAG](https://en.wikipedia.org/wiki/JTAG) targets use `monitor jtag_scan`. We should now see the following output:

```
Target voltage: 3.3V
Available Targets:
No. Att Driver
1      Nordic nRF52 M3/M4
2      Nordic nRF52 Access Port
```

Since we want to attach to the MCU to flash an debug code we're going to chose option 1.  Execute the following command to attach to our end device:

```
attach 1
```

## Flashing Our Program

Now that we're attached to our target, we can load our code. This command will load an [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) file onto our target.

```
load build/nrf52-blinky.elf
```

With our file loaded and running we're going to need some information if we want to peek under the hood. Loading our symbol table from our ELF file will help GDB correlate the instructions being run with the code they were generated from.

```
file build/nrf52-blinky.elf
```

Our program is running and our symbols are loaded! We can now treat this like a normal GDB session, and start unraveling whatever monstrous bug we have created! If you're not familiar with GDB there are many great tutorials out there. I would recommend this [video](https://www.youtube.com/watch?v=bWH-nL7v5F4) if you are interested in learning more.

## Automation

Executing all of these commands every time we want to flash our board would be quite cumbersome. Luckily GDB provides a method to script these commands. Create the file flash_blinky.scr with the following contents:

```
monitor swdp_scan
attach 1
load
kill
```

After creating the file execute the following command:

```
gdb-multiarch -nx --batch \
    -ex 'target extended-remote /dev/ttyACM0' \
    -x flash_blinky.scr \
    build/nrf52-blinky.elf
```

This will run all of our previous commands all at once! For bonus points consider adding this as a target to your build system!

## Conclusion

This tutorial should give you all the knowledge you need to get started using the Black Magic Probe! Have fun and happy debugging!
