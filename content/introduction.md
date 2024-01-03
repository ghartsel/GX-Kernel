---
weight: 10
title: Introduction
---


<h1>GX/Kernel Programming Guide</h1>

# Introduction

> QUICK START

> 1: Create a new location to build a GX/Kernel application:

```shell
$ cd $HOME/development
$ mkdir src
$ cd src
```

> 2: Clone the GX/Kernel repository:

```shell
$ git clone git://git.ghworks.org/gxkernel.git gxkernel
```

> 3: Create an application file ...

```shell
$ nano myFirstApp.c
```

> ... and add the following code to make a simple application:

```c
#include <stdio.h>
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    DO_FOREVER
    {
        status = ALERT (SEC_10);   // set timeout and wait
        
        switch (status):
        {
            case TIMEOUT:           // timeout occurred
                printf("Success: timeout.");
            default:                // request failed
            case FAILURE:
                printf("Fault: system error.");
        }
    }
}
```

> 4: Edit the gxconfig.h file to match your hardware. The default settings are for the Motorola M68HC11 EVB evaluation board.

> 5: Run `$ make` to compile and link your example application with the GX/Kernel code and create a downloadable executable named main.hex.

> 6: The final output file, main.hex, can now be downloaded and tested on your hardware or emulator. Refer to the documentation provided with your target board, emulator or PROM programmer for specific instructions on downloading files.

> 7: Reset your target environment to run the myFirstApp example application. You should see the "Success: timeout." message displayed every 10 seconds. You are now set up to build a full application.

The GX/Kernel features efficient and flexible real-time facilities that
meet the demands of today's hard-deadline, embedded systems.

While the run-time considerations are important, system development and
maintenance are equally important. The complexity of real-time
applications extends to all phases of the software life cycle. The
GX/Kernel is designed to allow developers to work
at a higher level of abstraction that translates to systems completed
in less time and with fewer errors.

The following list summarizes the GX/Kernel features:

-   *Preemptive Multitasking* scheduler for real-time responsiveness.

-   *Priority Scheduler* Tasks may be prioritized by importance.

-   *Multiple-instance* Single-source code instance for multiple tasks.

-   *Task Bank Switching* Built-in facility supports bank switching.

-   *Hard-deadline* Deterministic scheduler.

-   *Fault-tolerant* Active fault detection and isolation.

-   *Nested Interrupts* Concurrent interrupts use a common stack.

-   *C-language Interface* High-level language support.

-   *Rom-able* Position-independent runtime code.

The following sections of this document describe how to install the
GX/Kernel operating system software and cover the information needed
to implement a multitasking application. This includes system
definitions, an API reference, debugging tips, and information about
resource utilization.

## The GX/Kernel Model

The GX/Kernel model is a conceptual view of the software. Specifically,
the model is concerned with system resource management.

For the GX/Kernel, the resources of interest are processing time and the
allocation of program and data space memory to processing time
intervals.

The task is the central concept for processor time
management. By allocating task code memory space to a time segment, the
kernel defines an entity that can be associated with time; and in
real-time systems, time partitioning is an inherent requirement.
Further, by assigning multiple task code memory spaces to different time
segments, the GX/Kernel achieves multi-tasking.

The work actually done by a task is independent of the kernel, although,
certain rules govern the construction of tasks, to achieve time
segmentation.

Effectively, a task is an abstraction of time, which provides a
time-associated entity that can be managed and referenced. In the
GX/Kernel model, all functional references are with respect to tasks.
These operations that may be performed within the context of a task are
called primitives.

In addition to providing a mechanism for operating with tasks,
primitives also provide higher abstractions for time and memory.

For time, primitives allow developers to design in terms of clock time,
synchronization, and atomic operations. Collectively, these time-related
primitives are called task coordination primitives.

For memory, primitives support abstractions such as queues, messages, and
partitions. Generally, the GX/Kernel model supports static, rather than
dynamic, memory. That is, all types of memory and their attributes are
defined at design time. There is no dynamic memory allocation facility
at run-time, except in a very specialized case. Such a memory model is
better suited to hard-deadline applications, because system behavior
becomes more predictable and simplified, with improved performance.

## The Task Unit

This section discusses the concept and implementation of a task. It
introduces the terminology associated with a task and
discusses the kernel abstraction supported by the task.

In implementation terms, a task is a run-time instance of the source
code. an example of a GX/Kernel task is shown below. The program gains
control of the processor, executes application-specific logic, and
explicitly releases control. A task uses coordination primitives to get
and release processing time.

```c
INT task_x ()
{
    /* variable declarations */
    INT status, msgid, msgsiz;
    CHAR *msg_p;

    /* task initialization */
    if (task_x_init () == SUCCESS)
    {
        /* task definition */
        DO_FOREVER
        {
            /* suspensive primitive */
            status = RECV (&msgid, &msg_p, &msgsiz, SEC_1);
            switch (status)
            {
                case SUCCESS: /* msg ready */
                process_msg (msgid, msgsiz, msg_p);
                break;

                case TIMEOUT: /* no msg */
                handle_timeout ();
                break;

                case FAILURE: /* fault */
                default:
                LOG_WARN (LY_0 + SS_0 + LV_U + P0 + 0, NOT_USED);
                break;
            } /* end switch */
        } /* end forever */
    }

    /* unrecoverable fault */
    LOG_FATAL (LY_0 + SS_0 + LV_U + P0 + 0, NOT_USED);
}
```

### The Task and Its Environment

#### TASK DISCIPLINE

To design a system where a task is the unit of work, the task
discipline imposed by the kernel must be understood. A main difference
between operating systems is the task discipline.

In the GX/Kernel, once a task gains control of the processor, it runs to
completion. That is, a task retains control until it explicitly releases
control by calling a suspending primitive. This discipline supports
tasks that run quickly and is consistent with the requirements of
hard-deadline, real-time systems. The GX/Kernel is suited to those
systems where only a small amount of time is needed to completely handle
an event. Naturally, the granularity of an event becomes a design issue.

Care should be taken to only use kernel facilities to affect a task\'s
behavior. For example, by using timing primitives to delay a task,
instead of instructions that cause a task to busy-wait, other tasks are
allowed to run while the task is waiting. Kernel fault management guards
against tasks that don't release control of the processor.

Although a task isn't normally preempted by the kernel or another task,
a task may be preempted by a hardware interrupt. This is transparent to
the task, however, and processing always resumes with the interrupted
task.

#### TASK ATTRIBUTES

Task capabilities and constraints can be modified for each task,
individually, by changing a task\'s attributes.

| Attribute | Description |
| --- | --- |
| Priority | Each task may be assigned a priority level relative to other tasks. Tasks that have work pending and are ready to run are given processor control according to their priority.<br><br>There are three priority levels. The first task ready at the highest priority level is the next task to run. |
| Instance | The GX/Kernel supports the concept of multiple-instance tasks. A task instance is a task that is \"known\" by the kernel at run-time. Usually, each instance is derived from a separate block of source code. With multiple instance tasks, however, more than one run-time instance may be from the same source code. This is useful for tasks that perform the same function on different entities, such as a task for each I/O channel or network link. Rather than having a single task maintain separate data for each entity, each task manages one set of data. These types of tasks are also more efficient because they don't need to continually map an entity to its data or function.<br><br>Care must be taken, however, in coding multiple instance tasks. Such tasks can't declare and reference C language-type static data because each run-time instance references these same data. Dynamic data must be allocated by each task to get its own data space, usually at initialization time. |
| Memory area | Each task has its own stack and dynamic memory areas.<br><br>The stack area is required. Any stack size may be defined, depending on expected stack usage. The stack is used by all subroutines called from the task, primitive calls, and interrupts that occur while in the TASK\_STATE.<br><br>Dynamic memory allocation is optional. If a task allocates dynamic memory, there are no restrictions on its use.<br><br>Kernel fault management continually audits stack and dynamic memory to detect corruption. Corrupted memory causes the task to be deleted. While this may limit fault propagation and increase system availability, depending on the importance of the deleted task, it is likely that neighboring task memory is also corrupted. |
| Resident bank | The GX/Kernel supports hardware memory bank switching to extend the 64-kilobyte address limitation. Bank switching only applies to code space memory, however. If the code is located in ROM, then ROM bank switching is allowed and, if the code is located in RAM, then that part of RAM with code space may be switched. Bank switching isn't supported for RAM data memory because this memory maintains the system context used by the kernel.<br><br>The task unit is a natural unit of consideration for bank switching. This is because context switching already occurs at the task level, so context switching to another bank is no different than a normal kernel context switch.<br><br>When a task is ready to run, the kernel determines the bank where the task resides, switches in the appropriate bank and resumes running the task.<br><br>A task\'s resident bank is defined in the configuration file.<br><br>The entry point of the user-provided bank switching procedure is also defined in the configuration file.<br><br>The usual cautions apply in writing the bank switching procedure to be able to return to the original bank. The initialization stack area, in internal RAM, is available to preserve information across a bank switch. In addition, the kernel and any common routines must be located at the same location in all banks. Only task code area and some specialized routines may be bank-dependent. |

### Task Identification and Creation

#### IDENTIFICATION

From a programming point of view, tasks are referenced by logical,
symbolic names. This makes the system more maintainable and extensible.
At initialization, a physical task name is assigned to each task, which
is only used, directly, by the kernel. Applications never need to use
the physical name, except as a primitive parameter, and then only to
improve efficiency. The physical name may be gotten with
the GETTID and GETMYTID primitives. This is usually done at task
initialization because logical and physical task names never change.

#### CREATION

Tasks are created by the kernel at initialization from configuration
file information. This information defines each task and its attributes,
which the kernel uses to allocate task resources.

Once created, a task is usually never deleted. Two exceptions are 1)
upon a system reset, and 2) when the kernel detected a critical fault
while in the TASK state.

## Kernel States

This section describes the states of the kernel, for overall resource
management, and the states of a task, for task resource management.

### State Descriptions

The kernel may be in any of four states, depending on the work to be
done. State is used primarily to determine how memory resources are
allocated.

| State | Description |
| --- | --- |
| INITIALIZATION | Immediately following a hardware or software reset, the kernel takes control of the processor. In this state, the only application programs that run are the hardware and software routines defined in the system configuration file. |
| TASK | If there is work for a task and the task assigned to the work is available, the kernel enters the TASK state. The intricacies of task-level processing are described in more detail in [TASK STATES](#task-states). |
| SUPERVISORY | In the SUPERVISORY state, resources are allocated from the systems resources, rather than task resources. This state may be entered when a hardware interrupt occurs during the TASK state. |
| MONITOR | When no task work or interrupt is pending, the kernel enters the MONITOR state and uses system resources. |

In the TASK and SUPERVISORY states, the watchdog (COP) monitors
execution time. If the task execution time exceeds the watchdog period,
the task is removed from the system and normal processing continues. The
task is never scheduled to run again. If the execution time exceeds the
watchdog period in the SUPERVISORY state, which occurs in an interrupt
service routine, a system restart is initiated. The default watchdog
period is 1.049 seconds and is set in the microcontroller\'s OPTION
register.

#### TASK STATES

All tasks are put in a READY state at initialization. The first task in
the task configuration table is the first task to run. Each task runs
or initializes its local data areas until it calls a suspending
primitive.

Task states change from READY to ACTIVE when work becomes available for
the task. Real-time processing occurs when tasks are allocated processor
time to perform work.

In the TASK state, with a task ACTIVE, the stack area is allocated from
the task\'s local memory. A task runs until it calls a suspending
primitive and there is no work to do or a higher priority task has work
to do. When suspended, the task is in a WAIT state, waiting for a
semaphore, event, message, or timer primitive. When a task resumes
execution, processing begins at the point where the task was suspended,
not necessarily at the task\'s entry point.

In addition to READY, ACTIVE, and WAIT states, a task may be in the
DORMANT state. This occurs if a fault was detected by the kernel during
the task\'s ACTIVE state. Once in the DORMANT state, the task can't run
again until the system is restarted; this prevents fault propagation.

#### NON-TASK STATES

The SUPERVISORY state is entered by a request to use
common system resources. See the discussion below on handling
interrupts, for more detailed information.

When there is no task work or interrupt pending, the kernel enters the
MONITOR state. This switches the active stack to the system memory area,
to be ready for an interrupt, and executes a STOP instruction.
Processing resumes with the next interrupt; an interrupt always occurs
under normal operating conditions with the Real Time Interrupt (RTII).
To eventually return to the TASK state, an interrupt service routine
must call a primitive that completes a task\'s wait condition.

### Scheduling Policy

Tasks are prioritized in the system configuration table when the system
is built. A task\'s priority never changes once the system is built,
which assumes processing requirements are understood before run-time.
With fixed task priorities, response times are predictable.

The next task to run is determined, whenever a task releases control of
the processor, by calling a suspending primitive. At that time, the
highest priority task in the READY state is run. When multiple, equal
priority tasks are ready to run, the task that first became ready runs.

Tasks may be preempted by interrupts and the interrupt service routine
may call a primitive that causes a waiting task to become READY.
Processing always resumes with the interrupt task.

### Initialization and Startup

The GX/Kernel does the basic hardware and software initialization needed
to run the kernel in the target processor environment. It then
automatically invokes initialization routines, provided by the
developer, to initialize application-specific hardware and software.

The application-specific routines are the only initialization routines
that need to be provided. Their entry points are specified in the
configuration file when the system is built.

Also, configurable microcontroller parameters that need to be set at
initialization are defined in the configuration file.

The kernel provides orderly initialization sequencing by initializing
from lower to higher levels of abstraction. First, the hardware, then
the kernel, and then application hardware and software are initialized; the idea
is the hardware must be available for the kernel to run, and the
hardware and kernel must be available for the application to run.

Entries in the configuration file provide hooks for the kernel to
initialize application hardware and software. The kernel initializes
application hardware before software.

At each step, the integrity of the supporting layer is confirmed before
the next layer is initialized. If resources aren't available for a
layer to provide the necessary services, subsequent layers aren't
initialized.

Faults that occur before kernel resource allocation and initialization
are complete are considered critical because the integrity of the system
can't be guaranteed. If a critical fault is detected, a STOP
instruction is executed to prevent fault propagation.

### Handling Interrupts

Conceptually, interrupt service routines are very similar to tasks in
that they run as a result of an external stimulus that changes the
context of the system. In the case of tasks, the stimuli may be
messages, events, or semaphores, while in the case of an interrupt
service routine, the stimulus is a hardware event detected by the
microcontroller.

Task-related context switches that change resource allocation needs are
managed by the kernel. However, there is no kernel to manage
hardware-initiated context switches in the same way. Interrupt service
routines must, therefore, explicitly interface with the tasks\' kernel
to coordinate resource allocation. A SUPERVISORY state is implemented in
the kernel for interrupt management.

SUPERVISORY state is entered by calling the primitive, ENTER\_SSTATE. In
SUPERVISORY state, the interrupt uses the stack area from the system\'s
memory area. Upon exiting from the interrupt, by calling EXIT\_SSTATE,
the interrupted task resumes at the point of interruption, again, using
the task\'s memory area stack.

The kernel supports nested interrupts. That is, an interrupt may occur,
and call ENTER\_SSTATE, while another interrupt service routine is in
progress. Control doesn't return to the task until EXIT\_SSTATE is
called by the last interrupt service routine.

Tasks may become READY to run as a result of a primitive called by the
interrupt service routine, which completes a task\'s WAITING primitive.

## Kernel Primitives

This section discusses the concepts associated with primitives. A
detailed description of each GX/Kernel primitive is found in Part 3.

GX/Kernel functions are called primitives because they are the
basic function for interacting with tasks and requesting kernel
services.

Primitives are called during task or interrupt service routine
execution. While primitive code and static data are in the kernel code
and data space, primitives use the calling task or interrupt service
routine stack area.

### Summary

GX/Kernel primitives are classified as follows. See the [API documentation](#api) for
more detailed information about each primitive.

| Classification | Primitive | Summary |
| --- | --- | --- |
| TASK MANAGEMENT | [GETTID](#gettid) | Get task identifier |
| | [GETMYTID](#getmytid) | Get current task identifier |
| TASK SYNCHRONIZATION | [GET\_CRID](#get_crid) | Get critical region identifier |
| | [ENTER\_CR](#enter_cr) | Enter critical region; semaphore |
| | [EXIT\_CR](#exit_cr) | Exit critical region |
| | [ENTER\_SCR](#enter_scr) | Enter supervisory critical region |
| | [EXIT\_SCR](#exit_scr) | Exit supervisory critical region |
| | [SIGNAL](#signal) | Signal event occurrence |
| | [WAIT](#wait) | Wait for event occurrence |
| | [SEND](#send) | Send message |
| | [RECV](#recv) | Receive message |
| TIMER SERVICES | [ALERT](#alert) | Alert after time interval |
| | [GETTIK](#gettik) | Get current timer value |
| QUEUE MANAGEMENT | [Q\_GETID](#q_getid) | Get linked list identifier |
| | [Q\_CLEAR](#q_clear) | Initialize linked list |
| | [Q\_GET](#q_get) | Get item from linked list |
| | [Q\_PUT](#q_put) | Add item to linked list |
| MEMORY MANAGEMENT | [LOCATE\_MEM](#locate_mem) | Get task dynamic memory |
| FAULT MANAGEMENT | [[LOG\_WARN](#log_warn)](#log_warn) | Report non-critical fault |
| | [[LOG\_FATAL](#log_fatal)](#log_fatal) | Report critical fault; initiate recovery |
| INTERRUPT MANAGEMENT | [ENTER\_SSTATE](#enter_sstate) | Enter supervisory state |
| | [EXIT\_SSTATE](#exit_sstate) | Exit supervisory state |

### Suspensive Primitives

There are two types of primitives. Those that may cause a task to be
preempted are called suspending primitives, and those that return
to the caller without preemption and are called non-underline suspending
primitives. The suspending primitives are,

| Primitive | Suspending Condition |
| --- | --- |
| [ENTER\_CR](#enter_cr) | Critical region isn't available.<br>Unblocking primitive: EXIT\_CR or timeout |
| [RECV](#recv) | No message is pending for the task.<br>Unblocking primitive: SEND or timeout |
| [WAIT](#wait) | No event is pending for the task for requested event condition.<br>Unblocking primitive: SIGNAL or timeout |

The suspending primitives have complementing primitives which,
when called from another task or interrupt service routine, cause the
suspended task to resume. A task resumes when the suspending condition
is satisfied.

Suspending primitives may not be called from an interrupt service
routine because an interrupt may not be suspended by the kernel.
Suspension supports real-time applications at the task level.

While tasks may be prioritized, primitives don't have a priority
attribute. For example, there is no priority assigned to messages for
the SEND and RECV primitives. All prioritization is considered with
respect to tasks.

## System Reliability

This section discusses the reliability issues associated with embedded,
real-time systems.

GX/Kernel features that promote the development of reliable systems are
also presented.

### Reliability Issues

Real-time systems are particularly error-prone because of their inherent
complexity; as complexity increases the probability of errors also
increases.

This problem is compounded by the requirement for embedded systems to
operate without intervention, possibly in the presence of errors.

Systems may be classified as fault-tolerant or fault-intolerant.
Fault-tolerant systems are characterized by anticipating that all errors
can't be removed before run-time, and mechanisms are provided to detect
and handle faults when they do occur. Complex, real-time systems,
usually, can't be tested adequately to guarantee that no faults occur.
Fault intolerant systems, on the other hand, assume that the system is
sufficiently tested, no faults occur, and no fault handling is
provided.

The GX/Kernel is a fault-tolerant. By implementing fault
management, the GX/Kernel achieves the objectives of increased system
availability and assures algorithm correctness.

### Reliability by Design

The GX/Kernel allows reliability to be designed into a system.

One method of fault management that is implemented in the system design
is software layering. A layered system allows it to be viewed in
smaller, logically related, less complex segments. This makes the
design more manageable and, by extension, more reliable. Layering is
also a facility for the isolation of faults. If a fault can be isolated
to a layer, then only that layer needs to be recovered, and system
availability is increased.

The kernel, itself, may be considered a single layer. However, because
it is the most dependent layer faults detected in the kernel usually
require complete system recovery. The kernel may not need to be
recovered if faults are detected in an application layer.

The GX/Kernel supports reliable system design by its built-in fault
management policies and by providing mechanisms for applications to
interface to the fault management system. These policies and mechanisms
are described in the sections, below.

### Fault Management

Fault management has three aspects; 1) fault detection, 2) fault
handling and reporting, and 3) fault recovery.

The kernel uses its fault analysis area for fault management. These data
are described in detail in a separate section.

#### FAULT DETECTION

B y active fault detection, the kernel can reduce fault propagation. This
is the primary mechanism that supports the fault-tolerant objectives.
The following fault detection methods are used by the GX/Kernel.

##### Memory Audits

The GX/Kernel partitions and classifies memory for fault
management. If the type of memory is known, the kernel can verify that
the data is
consistent with the memory type. While this method doesn't guarantee
fault
detection, common types of memory corruption are detectable.
System and task stacks, kernel data structures, and kernel structures,
such as queues, in the application memory area are continuously audited.

##### Guarded Primitive Access

Whenever a primitive is called, the kernel verifies that it is a valid
primitive, that the
parameters are consistent and within kernel-defined limits, and that
resources are available.

##### Software Watchdog

The hardware watchdog (COP) is used to detect tasks and interrupt
service

routines that keep control of the processor for longer than the watchdog
period.

This is an indication of faulty algorithm execution.

The above fault detection methods are used at run-time. To reduce
development time by detecting errors early in the process, system build
tools are part of GX/Kernel fault management. This supports the notion
that faults detected early in the development cycle are less expensive
to correct.

Fault detection before run-time has the added benefit of freeing the
kernel from some fault detection at run-time, resulting in a more
efficient kernel.

The following fault detection methods are used during implementation.

##### Primitive Declaration

The macro declaration of the primitive interface allows most assemblers
and compilers to detect reference and parameter errors in the primitive
call.

##### System Build Utility

This is the primary tool for fault management before run-time that
relates directly to the kernel. The system build utility, described in detail in a
separate section, tests for overall resource availability and data consistency.

The high-level language interface allows applications to be implemented
with the probability of fewer errors.

#### FAULT HANDLING AND RECOVERY

Once faults are detected they must be handled to achieve fault
tolerance. A completely handled fault includes a determination of fault
severity, reporting and logging the fault, and initiating recovery
action. Recovery action depends on the type and severity of the fault.

##### Kernel Fault Handling

Kernel-detected faults are only acted on if they are determined to be
critical to
system operation. Otherwise, the fault is only reported and logged.

Fault classification, either critical or acceptable, depends on the
kernel state and
the scope of the data if it is a data-related fault. Acceptable faults,
from the
kernel\'s point of view, are those that are limited to a single task.
Such faults occur
in the TASK state or in a task\'s stack or dynamic data. Faults in any
other
kernel state are classified as critical.

For acceptable faults, the kernel simply deletes the affected task, so
the fault isn't
repeated or propagated. The fault is reported to the fault handling
task if one is
defined. It is left to the application to determine the impact of the
deleted task and
take appropriate recovery action.

For critical faults, the kernel determines that system integrity can't
be
guaranteed. The kernel, therefore, reports the fault, then initiates a
kernel restart.
The restart causes all tasks to be restarted. The cause of the restart
is available in
the fault analysis area.

##### Application Fault Handling

Application-detected faults are independent of the kernel. Their
classification and recovery action depends on the application. Applications
use kernel primitives to report the fault and initiate recovery.

The [LOG\_WARN](#log_warn) primitive is used to report non-critical faults. This is
useful for
noteworthy faults that don't affect system operation, and for
check-pointing during debugging. The fault is only reported and logged; no
recovery is initiated by the kernel.

The [LOG\_FATAL](#log_fatal) primitive is used to report faults the application
determines adversely affect system operation. The kernel reports the fault and
initiates recovery by restarting the kernel, which also restarts the tasks.

Both [LOG\_WARN](#log_warn) and [LOG\_FATAL](#log_fatal) cause fault-related information to be
logged in the kernel\'s fault analysis area. This information is available at
run-time to report the fault and is useful during debugging to determine the
cause of the fault.

Optionally, the fault may be reported to a fault-handling task. The task
name must first be defined in the configuration file. This allows the application
to take application-dependent recovery action.

For primitive access, the kernel detects some classes of errors before a
fault occurs. An example is primitive parameter errors. The kernel
reports these faults in the status returned by the primitive. The kernel
always returns a status to a primitive call. It is the responsibility of
the application to check the return value and take the appropriate
action.
