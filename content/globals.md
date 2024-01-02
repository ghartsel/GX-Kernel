---
weight: 30
title: Global Definitions
---


# Global Definitions

This section describes the GX/Kernel interface conventions. These
conventions have been defined to give a consistent interface, which
makes application software easier to implement and maintain.

## Primitive Access

The primitive interface to the GX/Kernel is invoked as an assembly or C
language subroutine call. Macros, defined in include files gki-k-m.h
and gki-k-m.inc implement the actual access to the kernel.

All kernel primitives return a function status. This is the status of
the execution of the algorithm for the primitive; either `SUCCESS`,
`FAILURE`, or a specialized meaning, such as `TIMEOUT`. No data associated with the
primitive are ever returned as a status. Data are passed and returned as
parameters.

## Literals Used with Primitives

Include files provide a common, portable interface to the GX/Kernel.
Include files gki-l.h, gki-l.inc, gki-k-l.h and
gki-k-l.inc contain the literals used with the kernel primitives
described in this section. These include files that might be changed to adapt
the kernel to the application.

These literals provide a method to reference kernel attributes
symbolically. This higher level of abstraction makes the interface more
maintainable and portable.

> Example using GX/Kernel literals:

```c
status = ALERT (SEC_10); // timer literal: SEC_10
    
switch (status):
{
    case TIMEOUT:        // return status literal: TIMEOUT
        handle_timeout();
    default:
    case FAILURE:        // return status literal: FAILURE
        handle_error();
}
```

### General

| Literal | Description |
| --- | --- |
| `TRUE` | General literal for TRUE conditional. |
| `FALSE` | General literal for FALSE conditional. |
| `DO_FOREVER` | This is used within a task to define the start of the main task execution loop. |
| `DO_NOTHING` | A statement place-holder, which generates no code. |
| `INV_ADDR` | This literal is used to set pointer variables to an invalid address, which may be used for fault detection. |
| `NOT_USED` | This is a parameter literal to indicate a parameter isn't used or has no significance. |
| `SUCCESS` | This value is returned by primitives to indicate no errors occurred. |
| `FAILURE` | If an error occurred during primitive execution, a FAILURE status is returned. |
| `TIMEOUT` | This literal is returned by primitives to indicate a timeout condition occurred while waiting for the requested operation. |
| `NO_TOUT` | If no timeout is desired, this literal is used for the time-specification parameter. |

### Task Attributes

| Literal | Description |
| --- | --- |
| `MAX_TSK` | For task management operations, this defines the maximum number of task instances, including multiple-instance tasks, that may be referenced. The range is 0 to 63. |
| `MAX_TNAME` | For task management operations, this defines the limit for task names. The range is 0 to 63. |
| `INV_TNAME` | Invalid task name indication. |
| `INV_TID` | Invalid task identifier. |
| `STKSIZ_L` | Stack sizes: large |
| `STKSIZ_M` | Stack sizes: medium |
| `STKSIZ_S` | Stack sizes: small |
| `MEMSIZ_L` | Dynamic memory sizes: large |
| `MEMSIZ_M` | Dynamic memory sizes: medium |
| `MEMSIZ_S` | Dynamic memory sizes: small |
| `PRI_HIGH` | Task relative priorities: high |
| `PRI_NORM` | Task relative priorities: normal |
| `PRI_NONE` | Task relative priorities: none |
| `BANK_0` through `BANK_3` | Task resident bank location: bank 0 through bank 3 |

### Queue Management

| Literal | Description |
| --- | --- |
| `MAX_QUE` | For queue management operations, this defines the maximum number of queues that may be referenced. The range is 0 to 255. |
| `INV_QNAME` | Invalid queue name reference. |
| `INV_QID` | Invalid queue identifier. |
| `Q_FULL` | Queue is full indication. |
| `Q_EMPTY` | Queue is empty indication. |

### Semaphore Management

| Literal | Description |
| --- | --- |
| `MAX_SEM` | For semaphore-related operations, this defines the maximum number of semaphores that may be referenced. The range is 0 to 255. |
| `INV_SEMNAME` | Invalid semaphore name reference. |
| `INV_SEMID` | Invalid semaphore identifier. |

### Event Management

| Literal | Description |
| --- | --- |
| `EVT_OR` | Conditionally wait for any event specified. |
| `EVT_AND` | Conditionally wait for all events specified. |
| `EVT_0`  through `EVT_15` | Specific event identifiers: event number 0 to event number 15, the maximum number of events. These symbolic names may be replaced by names meaningful to the application. |

### Timer Management

The following time interval literals are defined:

| Literal | Description |
| --- | --- |
| `MS_100` | 100 milliseconds |
| `MS_500` | 500 milliseconds |
| `SEC_1` | One second |
| `SEC_10` | Ten seconds |
| `MIN_1` | One minute |
| `MIN_10` | Ten minutes |
| `HR_1` | One hour |

<aside class="notice">
The timer management literal values are for a real-time clock
interrupt rate of 32.77 milliseconds. If this rate is changed, these
literals must also be changed. Other time reference symbols may be
added, within the limitation of the maximum interval that can be
represented in 16 bits.
</aside>

> LOG_FATAL example:

```c
LOG_FATAL (LY_3 + SS_0 + LV_L + P2 + 3, NOT_USED)
```

### Fault Management

When used as the error location parameter with the [LOG\_FATAL](#log-fatal) and
[LOG\_WARN](#log-warn) primitives these literals have the following bit position significance:

| Bit Position | Description |
| --- | --- |
| 15 | reserved |
| 14‑12 | layer identifier |
| 11‑10 | subsystem identifier |
| 9‑8 | level identifier |
| 7‑4 | procedure identifier |
| 3‑0 | fault identifier |

Here is an example of how the literals may be used with the fault
management primitives. The example reports the third
fault in layer 3, subsystem 0, level 2, and procedure 2.

The error location is logged in the Fault Analysis Area.

| Literal | Description |
| --- | --- |
| `LY_0` through `LY_7` | System layer: 0 to 7 |
| `SS_0` through `SS_3` | Layer subsystem: 0 to 3 |
| `LV_C` | Procedure/level: common |
| `LV_L` | Procedure/level: lower |
| `LV_M` | Procedure/level: middle |
| `LV_U` | Procedure/level: upper |
| `P0` through `P15` | Location in procedure/level: 1 to 15 |
