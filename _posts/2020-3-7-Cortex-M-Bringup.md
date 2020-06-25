---
layout: post
author: Blake Hildebrand
title: Cortex-M Bringup
excerpt: "The black magic probe is an open source JTAG/SWD debugger for ARM micro-controllers. It hosts a GDB remote server accessible over a serial connection to easily interface with GDB clients. The project provides support for debugging and flashing for a growing number of arm micro-controllers."
categories: [cortex-m]
comments: true
---

# Getting Started With Cortex-M Development

ARM's [Cortex-M](https://developer.arm.com/ip-products/processors/cortex-m) line of micro-controllers is one of the most popular choices for embedded systems. With extensive tooling support, and large breadth of options balancing computing power and cost this is no accident. This tutorial will focus on Nordic's [nrf52840](https://www.nordicsemi.com/Software-and-Tools/Development-Kits/nRF52840-DK) [Cortex-M4](https://developer.arm.com/ip-products/processors/cortex-m/cortex-m4), but most of the core principles will apply to anything in the Cortex-M family.

## MCU Boot Procedure

When a Cortex-M MCU first receives power it will attempt two things:

1. Load the value stored at address 0x0 into the MSP (Main Stack Pointer).
2. Load the value stored at address 0x4 into the PC (Program Counter).

We now know what our MCU will do when it boots, but what should we put in these locations? The name of the load location for address 0x0 gives us a pretty big hint. We're going to load in the top of our system stack! Since address 0x4 will be loaded into the program counter it represents our first point of code execution. This is called the reset vector, and for Cortex-M processors it will be the address of a function pointer.

We know the what and the where of our boot process, but how do we get the values we want into their respective addresses? With the help of some [linker](https://ftp.gnu.org/old-gnu/Manuals/ld-2.9.1/html_chapter/ld_3.html) magic!

## GNU Linker Scripts

GNU linker scripts allow us to direct the linker to put memory in specific locations.

## Exception Model

The Cortex-M exception model dictates how the MCU handles various situations. Below is an excerpt from the [Cortex-M4 Reference Manual](https://static.docs.arm.com/ddi0403/eb/DDI0403E_B_armv7m_arm.pdf). There are many exceptions here, but for the sake of scope we are going to focus on the reset exception. For a good reference on the other exceptions I would check out [this](https://interrupt.memfault.com/blog/arm-cortex-m-exceptions-and-nvic) article.

The reset exception represents the first function executed on boot of the processor. It has the responsibility of initializing our program, and setting up processor specific information. We're going to focus on what is needed to initialize our program.

![Exception Model](https://lh3.googleusercontent.com/INo-eVE31RM9vx8JdrB_rtQ85btMuJPWiFHYsGy_vVG9T1EaUbElNQhqx2Xf6QQ9g5o0l_vpTF4q1nSrBUnbHMqOhCOwnz9V7q1xX3ONnQ0CpGipYzIj5aNjZ810ilmz-DneLrwZ-Sbm54SM4XQyZlF32D6xtiDpjo_ISYehMZ7D-Kwo5Yu1da01oY6p5SbQsI7do2ttwHoQ4K9elrtnw_obSXRu1FPcnqXqDND-Q2s1so5vEfRzhoagn6DXkqGWmSyEKuG0DKxrzp7lKP5UfkCOZss0iqhpiri14aY2bH5YKAEoLeEy7YXIPCN7LxJthaV5RfsaxqUjCDpKHctIvlOZ8iygcx76WkL6Wqqje2oACVKhpgDs-tmxA0LhSLEb6zXOWm8bwAlPSaiXxwwlnCuoeYDhYY9jPSSClnrGvsnFirrqorwmJBGjZIoK2jxPw7zGk4zrSSa3XS2VDl_hgtDREuTdOf7IiovwCMtDicOl21p2ZNpPC2mlyw1ExLSF_lorTX7ADFiiAbsr-yHGIzjsgY3Me7pDwgikYQp7x6BFu25MbQ-2znVXWt4gt9PntbuF71uEGe0NSZTnNROBzt1Xllq7Bta5Zc03DCARQ54RgYMoU4mOounuyYgF5NJHSPmvHkDGxs7v7t4AolI_Rz6V6qo8N_deyCaWPuobBOE52f13PqIH=w292-h580-no)