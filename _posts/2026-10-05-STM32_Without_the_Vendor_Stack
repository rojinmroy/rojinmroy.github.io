# STM32 Without the Vendor Stack

*An experiment in startup code, linker scripts, direct register access, and command-line tooling.*

Project repository: stm32-baremetal

## Introduction

Most embedded projects make it easy to get started. Create a project, select a board, configure a few peripherals, and write the application.

But what happens underneath all that?

I wanted to understand what it takes to get a microcontroller from reset to executing a C program, without relying on a vendor-provided framework.

So I started with a simple experiment: blink the onboard LED of an STM32F103C8T6 Blue Pill.

The LED blinking wasn't the interesting part. Getting the firmware to run was.

## Starting from the Registers

The first step was configuring the GPIO peripheral directly.

On the STM32F103, GPIOC is connected to the onboard LED on PC13. Before accessing the pin, its peripheral clock needs to be enabled through the RCC register.

The pin is then configured as a push-pull output through the GPIO configuration register.

```c
#define RCC_APB2ENR (*(volatile unsigned int *)0x40021018)
#define GPIOC_CRH   (*(volatile unsigned int *)0x40011004)
#define GPIOC_ODR   (*(volatile unsigned int *)0x4001100C)

RCC_APB2ENR |= (1 << 4);

GPIOC_CRH |= (1 << 20);
GPIOC_CRH &= ~(3 << 22);

while (1) {
    GPIOC_ODR ^= (1 << 13);
}
```

There is no HAL involved here. The registers are accessed directly through their memory-mapped addresses.

This is a small amount of code, but it also means that the application is responsible for knowing how the peripheral is configured and accessed.

## But How Does the Processor Reach `main()`?

Writing the application was straightforward. The more interesting part was understanding how the processor gets there.

A Cortex-M processor doesn't start by calling main(). After reset, it obtains its initial stack pointer and reset handler address from the vector table.

I therefore needed to provide a vector table and a reset handler.

The reset handler performs two important initialization steps before calling the application:

- Copies initialized global and static variables from Flash into RAM.
- Clears the `.bss` section.

These are normally handled by the C runtime startup code. In this experiment, I implemented the basic initialization myself.

This was probably the most useful part of the exercise: understanding that main() is not the entry point of the firmware image.

##The Linker Has a Role Too

The startup code depends on knowing where the different sections of the program are located.

That information comes from the linker script.

For this particular STM32F103C8T6 configuration, the memory map is:

| Region | Start address | Size |
|---|---|---|
| Flash | `0x08000000` | 64 KB |
| RAM | `0x20000000` | 20 KB |

The linker script places the vector table and executable code in Flash, while arranging for initialized and uninitialized data to occupy RAM.

It also defines symbols used by the reset handler to initialize these sections.

This made the relationship between the compiler, linker and startup code much clearer. They aren't independent pieces of the build process; they have to agree on the firmware's memory layout.
Building Without a Vendor IDE

I used the GNU Arm Embedded Toolchain for compilation and linking, with a small Makefile to automate the process.

OpenOCD handles programming through the ST-Link V2. For debugging, I used gdb-multiarch.

The resulting workflow is simple:

```bash
make
make flash
gdb-multiarch firmware.elf
```

The ELF file is particularly useful because it contains debugging information alongside the executable code. This lets GDB map machine instructions back to the source.

The entire workflow is command-line driven, without depending on a vendor IDE or project generator.

##What I Took Away

Blinking an LED is hardly a challenging embedded application. But implementing the pieces normally hidden by a development framework was a useful exercise.

It exposed the responsibilities of the different components:

- The processor's reset sequence determines how execution begins.
- Startup code prepares the C environment.
- The linker determines where code and data reside.
- The application configures and controls the hardware.
- The toolchain turns the source into an executable firmware image.

It also raised another question.

How much of this implementation is specific to the STM32F1 family, and how much is common to other microcontrollers?

The next experiment will be to repeat the exercise on a different MCU family: the RP2350.

Rather than introducing a common abstraction layer, I'll keep both implementations independent and compare the actual changes required to get the same basic application running.

AI Disclaimer

No AI tools were harmed in the making of this project. Some were consulted, some were confused, and a few were probably more useful than others. All in all, a productive experiment!
