---
weight: 20
title: Installation and Integration
---


# Installation and Integration

## The Release Package

The release package contains all required G/Kernel software, the system build
utility and samples for tasks, configuration files, and development
tools.

The G/Kernel software is compatible with Archimedes development tools.
Source code and include file syntax are also compatible with the
Archimedes assembler, although minor editing changes allow the software
to be used in other development environments.

### Release Directories

The release contains the G/Kernel code and utility software in the
following directories.

#### /tools

The /tools directory has the executable file, sysbld.sh, used
to build the target system configuration file.

#### /obj

This directory contains the following relocatable object
files compatible with the Archimedes M68HC11 linker. To create object
files compatible with other linkers, the source code must be assembled
with the desired assembler.

#### /inc

This directory contains the following include files for assembly and C
language source code. These files contain the declarations needed to
interface to the G/Kernel.

| Assembly | C language | Contents
| --- | --- | --- |
| gki-l.inc | gki-l.h | Global literals |
| | gki-t.h | Global types |
| gki-k-l.inc | gki-k-l.h | Kernel literals |
| gki-k-m.inc | gki-k-m.h | Kernel macros |
| gki-k-e.inc | gki-k-e.h | Kernel externals |

#### /examples

Example files are provided for first-time users of the
G/Kernel. These files provide direction for integrating the kernel
into a system.

| Filename | Description |
| --- | --- |
| gxk000.xcl | This is a linker command file. It is useful for showing the memory assignments defined during linking, and for the inclusion and ordering of G/Kernel files. |
| gkc-stt.asm | This is an example configuration source file, which may be created with a text editor or by using the sysbld.sh utility. |
| gkc-iv.asm | This is the default interrupt vector table distributed with the kernel. This file needs to be changed to support the interrupts used by an application; those interrupts used by the kernel should not be changed, however. |

### Kernel Installation

Install G/Kernel with the following steps:

```shell
$ cd $HOME/development
$ mkdir src
$ cd src
```

1. Create a new location to build a G/Kernel application:

```shell
$ git clone git://git.ghworks.org/gxkernel.git gxkernel
```
2. Clone the G/Kernel respository:

The files needed to build and debug an application are in the following directories:

| Directory | Description |
| --- | --- |
| /examples | Development examples |
| /inc |  G/Kernel include files |
| /obj | G/Kernel object files |
| /src | G/Kernel source files|
| /tools | System build utility |

### Kernel Integration

A number of simple steps are needed to integrate the kernel and
application. These steps may vary, depending on the development
environment and tools used.

If the Archimedes tool set is used, only the object files are needed, to
link the G/Kernel with the rest of the system.

If other tools are used, the source code files need to be edited to be compatible with the
assembler used. Generally, this only means changing assembler DIRECTIVE
statements to those recognized by the assembler. All include files, both
those used by the kernel and those used to interface to the kernel, may
also need to be edited.

The C language include files do not need to be changed, because no
vendor-dependent constructs or library functions are used.

1.  Assemble the kernel source code files; this requires the include
    files in the source code directory and the /inc directory.

    The include files are provided in assembly and C language source
    code. They must be declared in the order shown in the example to the right,
    which is done by including the gxos.h and gxos.inc include files.

    It is required that you set the correct include file pathname in
    the assembler and compiler command lines, or in the source code
    declarations.

2.  Edit and assemble the interrupt vector table source code file, gkc-iv, to
    include the interrupt vectors used by the application.

    If the application uses interrupts, the default interrupt vector table,
    gkc-iv, needs to be changed to add the entry points for application
    interrupt service routines.

    Care should be taken that interrupt vectors used by the kernel are
    not modified. See [Interrupt Vectors](#interrupt-vectors).

3.  Create and assemble the system configuration source code file, using either a
    text editor or the sysbld.sh utility.

    The system configuration file provides the link between the kernel and
    application. It is a program source file, which needs to be created then
    assembled. It may be created using a text editor or the sysbld.sh utility.
    The example configuration file may serve as a template for a new file.

4.  Create a linker command file that includes the kernel object files,
    as shown in example the gxk000.xcl file.

    <aside class="notice">
    If you use Archimedes development tools, link using the
    .obj files from the release package. Otherwise, use the object
    files created in step 1.
    </aside>

5.  Link the system to produce a binary or hex file for the target
    hardware. This step is the same as for a system that does not
    include the G/Kernel.

After all kernel files have been edited and successfully assembled, the
kernel object files may be linked with your application source files. See [language](#language-considerations)

### Language Considerations

The G/Kernel implementation allows it to be portable across different
development environments. This is done by using generic language
features and constructs defined by well-established standards.
Constructs and features particular to a specific vendor are avoided.

The assembly language instruction syntax follows that defined in the
Motorola technical documents for the M68HC11. However, the assembly
language source code also contains directives for such things as
symbols, macro definitions, and memory location specifications. For these
directives, Archimedes-defined constructs have been used. However, their
function is common across most assemblers and the directive may be
replaced by the corresponding one for the assembler in use.

No compiler vendor-dependent externals, including library functions, are
referenced. All external references are resolved within the kernel, with
links to the application through the configuration file.

Many compilers require special initialization code and library
functions, depending on how the C code is written. If the application
invokes these, they must be specified in the linker command file.

Because the kernel manages the initialization sequence, any
compiler-required initialization must be done in the software
initialization procedure, which is accessed through the configuration
file entry point, `sw-init-ep`. For example, a procedure required by
some compilers to initialize constant data at run-time provides this
function. Alternatively, initialize the data in an assembly language
file, which eliminates the need for compiler-specific procedures and
makes the software portable to other compilers.

If bank switching is used to extend the addressing limitation, bank
switching must be done through the kernel. This supports the abstraction
that the kernel manages system resources and memory is a resource.
Language support for bank switching cannot be used in the kernel
environment.

## Hardware Interface

This section discusses the hardware interface as it applies to the
G/Kernel. Refer to Motorola technical documentation for a detailed
description of the M68HC11.

### M68HC11 Configuration

#### INTERNAL RAM

The kernel uses the microcontroller internal RAM for its primary control
data. This allows the kernel to take advantage of the direct addressing
mode for memory reference instructions. The result is increased
performance because most of the program execution occurs within the
kernel.

After initialization, some of this area is made available to the
application. See the section on memory layout for a description of
internal RAM.

#### INTERNAL REGISTERS

Only those internal registers listed below are used by the kernel. All
others are available for the application.

The kernel guarantees that the microcontroller time-protected
registers are initialized within the required time limit.

The 64-byte register block may be located on any 4K boundary. Default
register settings may be used or other values may be used for special
application requirements. The settings are defined in the configuration
file.

| Register | Address | Default Value | Usage |
| --- | --- | --- | --- |
| HPRIO | 03CH | 07H | This sets the real-time interrupt as the highest priority interrupt source. This is to assure the accuracy of the kernel timer services. |
| INIT | 03DH | 01H | This positions the 64-byte internal register block at address 1000H. |
| OPTION | 039H | 03H | his sets the watchdog (COP) timeout rate to 1.049 seconds, for an 8MHz crystal.<br><br>The watchdog is pulsed by the kernel whenever a task switch occurs, or periodically if there is no task work pending. This means that a task must not retain control of the processor for greater than the watchdog timeout period. |
| TMSK2 | 024H | 03H | This sets the timer prescale factor to 16 for a real-time interrupt rate of 32.77 milliseconds. |

#### INTERRUPT VECTORS

The kernel handles the following interrupt vectors:

| IRQ | Address | Usage |
| --- | --- | --- |
| RTII | FFF0H | The real-time clock interrupt supports kernel timer services. |
| SWI | FFF6H | The software interrupt (TRAP) is used for the application interface to the kernel. |
| RESET | FFFEH | The reset interrupt is used to vector execution to the start of the kernel initialization sequence. |

If the default interrupt vector table is used, the following interrupts report a fatal fault then vector to the start of the kernel initialization sequence, the same as a RESET interrupt:

| IRQ | Address | Usage |
| --- | --- | --- |
| OPCODE | FFF8H | Illegal opcode detected.
| COP | FFFAH | Watchdog timeout. |
| CME | FFFCH | Clock monitor failure. |

The default interrupt vector table causes all other interrupts to be
reported as warning faults, then returns from the interrupt.

### Hardware Configuration

The G/Kernel requires that the microcontroller be configured for
Expanded Multiplexed Operation (MODE 1).

Memory locations may be configured for any location allowed by the
microcontroller. An exception is internal RAM, which must always be
located at address 0000H.

All microcontroller interfaces not described above are available to the
application, with no restriction on their use by the kernel.

Any combination of RAM and ROM options is allowed. The kernel code
is ROMable.

The kernel supports up to four memory banks for task code space. These
may be either RAM or ROM memory.

The kernel automatically switches in the bank where the task code
resides by calling the user-provided bank switching routine. The entry
point for this routine is defined in the configuration file.
