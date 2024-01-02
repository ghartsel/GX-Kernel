---
weight: 100
title: Application Programming Interface
---


# API Reference

This section gives a detailed description of the GX/Kernel API primitives
used to invoke kernel services.

For each primitive, a functional description is given along with
operational considerations particular to the primitive. This is followed
by a description of the actual subroutine call.

The description shows the C and assembly language macro interface with
the required parameters and return value. All parameters are required,
although, parameters not applicable for a particular call contain a
place-holding literal such as NOT\_USED.

For the assembly language interface, all registers are preserved, except
the D register, which returns the primitive status. The returned
D-register status has the same meaning as the C- language interface
status.

For both C and assembly language, parameters shown in upper case are
literal constants, while those in lowercase are memory address
references.

## ALERT

Request Wakeup after Time Interval

### SYNTAX

status = ALERT (tout\_val);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    status = ALERT (SEC_10);   // set 10-second timer and wait
    
    switch (status):
    {
        case TIMEOUT:           // timeout occurred
            handle_timeout();
            break;
        default:                // request failed
        case FAILURE:
            LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
   }
}
```

### DESCRIPTION
Request for the task to be signaled after a specified amount of time has
elapsed.

This is a suspending primitive.

When the timeout occurs, the task is scheduled to run according to its
assigned priority.

There is no provision to cancel an alert request before the timeout
occurs.

If NO\_TOUT is requested, this primitive has no effect, and the task
continues as the active task.

The time interval is associated with the real-time clock interrupt rate
and is accurate to within one clock increment.

The alert request applies to the currently active task and may not be
called from an ISR.

The time interval parameter is expressed in 100-millisecond units. The
minimum time interval that may be requested is 100-milliseconds and the
maximum is 6553.5 seconds. Accuracy is +/- 100-milliseconds.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *tout\_val* | INT | Elapsed time value in 100-millisecond increments. |

### RETURN

| Value | Description |
| --- | --- |
| TIMEOUT | *tout\_val* requested time has expired. |
| FAILURE | Unable to handle alert request. |

## ENTER\_CR

Get Access to Resource

### SYNTAX

status = ENTER\_CR (cr\_id);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    BYTE app_cr;

    // get critical region ID defined in config file
    if (GET_CRID (RESOURCE_A, &app_cr) == SUCCESS)
    {
        status = ENTER_CR (&app_cr);
        
        switch (status):
        {
            case SUCCESS:                   // region acquired
                process_resource();
                status = EXIT_CR (&app_cr); // release region
                break;
            default:
            case FAILURE:                   // invalid cr ID
                LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
        }
    }

}
```

### DESCRIPTION

This primitive provides for synchronization on a user-defined resource
using a semaphore and guarantees mutually exclusive access to a
resource.

The resource id must first be gotten using the CR\_GETID primitive.

Resource access is managed by a semaphore, and tasks are queued to the
semaphore on a first-come-first-serve basis. If the resource isn't
locked
by another task, the resource is locked and the calling task continues
as
the active task. If the resource isn't available, the calling task is
suspended. The task remains suspended until all previous tasks have
released exclusive access to the resource.

If the current task has already locked the resource, this primitive
has no
effect, although an error status is returned.

This primitive may not be called from an interrupt service routine.

A resource is released by the EXIT\_CR primitive.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *cr\_id* | BYTE\* | Resource id |


### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Access granted and resource locked. |
| FAILURE | Invalid resource id or task has already acquired resource. |

## ENTER\_SCR

Get Exclusive Access to Processor

### SYNTAX

status = ENTER\_SCR ();

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    // get exclusive access to the processor
    if (status = ENTER_SCR () == SUCCESS)
    {
        exclusive_action();
        status = EXIT_SCR (); // release exclusive access
    }
    else
    {
        alternate_action();   // wait or alternate processing
    }

}
```

### DESCRIPTION

Get mutually exclusive access to the processor.

This primitive is implemented by disabling maskable interrupts and is
not associated with a particular resource. Access is released by
EXIT\_SCR.

This primitive is recommended for mutually exclusive access of short
duration because kernel time-related function is affected.

The processor condition code register is preserved upon return from
the
EXIT\_SCR primitive.

This is a non-suspending primitive.

### PARAMETERS

none

### RETURN

none

## ENTER\_SSTATE

Enter Supervisory State

### SYNTAX

ENTER\_SSTATE ();

```c
#include <gxos.h>
#include <gxconfig.h>

int int0(void)
{
    ENTER_SSTATE ();    // enter supervisory state
    handle_interrupt0() // interrupt processing
    EXIT_SSTATE ();     // return to previous state
}
```

### DESCRIPTION

This primitive is the mechanism for an interrupt service routine to invoke
supervisory state processing.

This primitive allows the calling interrupt service routine and nested
interrupt service routines to switch to the common system stack.

The last interrupt service routine to exit the supervisory state returns
control to the interrupted task and switches to the task stack.

For every ENTER\_SSTATE call, there must be a matching EXIT\_SSTATE
call.

### PARAMETERS

none

### RETURN

none

## EXIT\_CR

Release Access to Resource

### SYNTAX

status = EXIT\_CR (cr\_id);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    BYTE app_cr;

    // get critical region ID defined in config file
    if (GET_CRID (RESOURCE_A, &app_cr) == SUCCESS)
    {
        status = ENTER_CR (&app_cr);
        
        switch (status):
        {
            case SUCCESS:                   // region acquired
                process_resource();
                status = EXIT_CR (&app_cr); // release region
                break;
            default:
            case FAILURE:                   // invalid cr ID
                LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
        }
    }

}
```

### DESCRIPTION

Release control of a resource that was acquired with the ENTER\_CR
primitive.

The resource identifier must first be gotten using the CR\_GETID
primitive and must be the same identifier used to acquire the
resource
with ENTER\_CR. Only the task that currently has access to the
resource
may unlock the resource.

When the resource is released, if a higher-priority task is waiting on
the
resource, the current task is suspended, and the higher-priority task
becomes the active task. The suspended task becomes active according
to
its assigned priority.

An error status is returned if the resource wasn't previously locked,
an
invalid resource id was given, or corrupted data structures were
detected.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *cr\_id* | BYTE\* | Resource id |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Resource released. |
| FAILURE | Invalid resource id, resource not acquired with previous ENTER\_CR by this task, or corrupted kernel data detected. |

## EXIT\_SCR

Release Exclusive Access to Processor

### SYNTAX

status = EXIT\_SCR ();

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    // get exclusive access to the processor
    if (status = ENTER_SCR () == SUCCESS)
    {
        exclusive_action();
        status = EXIT_SCR (); // release exclusive access
    }
    else
    {
        alternate_action();   // wait or alternate processing
    }

}
```

### DESCRIPTION

This primitive is the complement of ENTER\_SCR and releases mutually
exclusive control of the processor.

The processor condition code register at the time ENTER\_SCR was
called is restored.

This primitive must not be called without a previous call to
ENTER\_SCR.

### PARAMETERS

none

### RETURN

none

## EXIT\_SSTATE

Exit Supervisory State

### SYNTAX

EXIT\_SSTATE ();

```c
#include <gxos.h>
#include <gxconfig.h>

int int0(void)
{
    ENTER_SSTATE ();    // enter supervisory state
    handle_interrupt0() // interrupt processing
    EXIT_SSTATE ();     // return to previous state
}
```

### DESCRIPTION

This primitive complements the ENTER\_SSTATE primitive and returns to
the task state from the supervisory state.

For every ENTER\_SSTATE call, there must be a matching EXIT\_SSTATE
call.

Interrupts may be nested and ENTER\_SSTATE primitive calls may be
nested. If EXIT\_SSTATE is called by an interrupt service routine that
isn't the last interrupt service routine pending, processing continues
in the supervisory state.

The last interrupt service routine to exit the supervisory state returns
control to the interrupted task and switches to the task stack.

### PARAMETERS

none

### RETURN

none

## GET\_CRID

Get Resource Identifier

### SYNTAX

status = GET\_CRID (cr\_name, crid\_p);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    BYTE app_cr;

    // get critical region ID defined in config file
    if (GET_CRID (RESOURCE_A, &app_cr) == SUCCESS)
    {
        status = ENTER_CR (&app_cr);
        
        switch (status):
        {
            case SUCCESS:                   // region acquired
                process_resource();
                status = EXIT_CR (&app_cr); // release region
                break;
            default:
            case FAILURE:                   // invalid cr ID
                LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, "Region doesn't exist");
        }
    }

}
```

### DESCRIPTION

This primitive gives a resource\'s identifier given the resource\'s
name.

The identifier is used for the ENTER\_CR and EXIT\_CR primitives,
which allow mutually exclusive access to the resource.

Resource names are assigned in the system configuration file. The name
is a consecutive number, beginning with zero and ending with one less
than the number of resources allocated. Designers may define literals
to associate more meaningful names with a resource.

The name is used by this primitive to map to the semaphore data
structures associated with the resource, however, these data
structures
never need to be referenced by an application.

Because a resource identifier doesn't change once it is created by
the
kernel, this primitive only needs to be called once.

This is a non-suspending primitive.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *cr\_name* | INT | Resource name |
| *crid\_p* | BYTE\* | Resource id location |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Resource id available. |
| FAILURE | Invalid resource name. |

## GETMYTID

Get Own Task Identifier

### SYNTAX

status = GETMYTID (tid\_pp);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    TID *tid_p;

    status = GETMYTID (&tid_p); // get id of this task
    
    switch (status):
    {
        case SUCCESS:           // valid task id
            this_task_handler();
            break;
        default:                // system fault
        case FAILURE:
            LOG_FATAL (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
    }
}
```

### DESCRIPTION

This returns the identity of the currently active task. It operates
the same as GETTID, except the task name isn't needed.

The task that invokes this primitive is the currently active task.

This primitive also verifies the integrity of the task\'s data
structures.

Because the task\'s identity doesn't change after it is created, this
primitive only needs to be called once.

This is a non-suspending primitive.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *tid\_pp* | BYTE\*\* | Task id location |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Task id available. |
| FAILURE | Task data structure fault detected. |

## GETTID

Get Task Identifier

### SYNTAX

status = GETTID (tname, tid\_pp);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    TID *tid_p;
    CHAR test_msg[];

    status = GETTID (TASK_0, &tid_p); // get id of given task
    
    switch (status):
    {
        case SUCCESS:           // valid task id
            test_msg = "This is a test."
            status = SEND (*tid_p, TEST_MSG_ID, &test_msg, sizeof(test_msg));
            break;
        default:
        case FAILURE:           // invalid task name
            LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
    }
}

```

### DESCRIPTION

This primitive is used to get a task\'s identifier, which is used with
other primitives that reference a task.

Given a logical task name, defined at compile time, this primitive
returns the run-time identity of the task.

The task name is a number between zero and the maximum allowed number
of tasks, minus one. The maximum number of tasks in the GX/kernel is
64. The task name is assigned in the configuration file.

The task identifier is a pointer variable to the task\'s control data
structure, however, the application never needs to reference the data
structure in normal operation.

The task must exist in the kernel\'s task data area. The task may not
exist if this primitive is called from a procedure before tasks are
created or if the task was terminated as a result of an error detected
by the kernel. In these cases, INV\_ADDR is returned as the task
identifier.

Because a task\'s identity never changes once it is created, this
primitive only needs to be called once, usually during initialization.

This primitive also checks the task\'s data structures for
consistency.

This is a non-suspending primitive.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *tname* | CHAR | Task logical name. |
| *tid\_pp* | BYTE\*\* | Task id location. |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Task id available. |
| FAILURE | Invalid task name, task doesn't exist, or invalid data. |

## GETTIK

Get Current System Timer Value

### SYNTAX

status = GETTIK (tikval\_p)

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    WORD tikval_p;

    if (GETTIK (&tikval_p) == SUCCESS)   // get timer tik
    {
        process_tik()
    }
    else
    {
        LOG_FATAL (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
    }
}
```

### DESCRIPTION

Get the current system time counter value.

The value is a continuous, 16-bit, 100-millisecond counter.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *tikval\_p* | WORD\* | Current timer value location. |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Timer value available. |
| FAILURE | Kernel fault. |

## LOCATE\_MEM

Locate Task Dynamic Memory

### SYNTAX

status = LOCATE\_MEM (mem\_pp);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    CHAR *mem_pp;

    status = LOCATE_MEM (mem_pp); // get task dynmem address
    
    switch (status):
    {
        case SUCCESS:             // memory available
            memory_action();
            break;
        default:
        case FAILURE:             // mem not allocated for task
            LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
    }
}
```

### DESCRIPTION

Get the location of a dynamic memory partition allocated to the current
task.

This memory partition is never used by the kernel, unlike the tasks\'
stack area, and is only made known to the task assigned to the
partition. Once the task locates its memory, there are no restrictions
on its use and it may be made available to other tasks.

This primitive only needs to be called once, preferably at task
initialization time.

Higher-level memory management functions are the responsibility of the
task; memory is never released through the kernel.

The memory is defined in the configuration file, by specifying the
memory size needed by the task. The requested size is guaranteed to the
task at run-time, although the location is determined by the kernel from
available memory.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *mem\_pp* | CHAR\*\* | Address for task\'s dynamic memory location. INV\_ADDR is returned if memory hasn't been allocated for the task. |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Memory location available. |
| FAILURE | Memory not allocated for task. |

## LOG\_FATAL

Indicate Fatal Fault Occurrence

### SYNTAX

status = LOG\_FATAL (loc, qual);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    WORD tikval_p;

    if (GETTIK (&tikval_p) == SUCCESS)   // get timer tik
    {
        process_tik()
    }
    else
    {   // report location of fatal fault with info
        LOG_FATAL (LY_0 + SS_0 + LV_L + P0 + 0, "System error");
    }
}
```

### DESCRIPTION

Log fatal-type fault and initiate recovery.

If a task has been defined in the configuration file as a fault handler,
the task is signaled with EVT\_0, to indicate a fatal fault occurred,
provided the fault didn't occur in the fault handler task.

If a task detects and reports a fatal-type fault or the kernel detects a
fatal-type fault in the task domain, the task is removed from the list
of available tasks and is never scheduled to run again. In this case,
this primitive returns to the scheduler and the system continues to run,
as much as possible without the affected task.

If a fatal-type fault is detected in the kernel domain or in hardware on
which the kernel is dependent, system integrity can't be guaranteed and
a STOP instruction is executed. Upon reset, the fault is signaled to the
fault handler task, if one was defined.

In all cases, fault-related information is logged to the fault analysis
area for future reference. This includes fault location, fault-specific
data, task and kernel stack areas, and kernel and task states. These
data are preserved until the next fault, even through a system reset.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |arn
| *loc* | WORD | location code |
| *qual* | WORD | Fault qualifier |

### RETURN

Not applicable

## LOG\_WARN

Indicate Non-fatal Fault Occurrence

### SYNTAX

status = LOG\_WARN (loc, qual);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    status = ALERT (SEC_10);   // set timeout and wait
    
    switch (status):
    {
        case TIMEOUT:          // timeout occurred
            handle_timeout();
            break;
        default:
        case FAILURE:
            // report location of non-fatal fault with info
            LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, "Unable to set timeout");
    }
}
```

### DESCRIPTION

Report a warning-type fault.

This primitive is used if detected faults aren'table but may not
compromise the system.

For warning-type faults, only the fault location and fault-specific
data, if any, are logged to the Fault Analysis Area. If a task has been
defined in the configuration file as a fault handler, the task signaled
with EVT\_1, to indicate a warning-type fault occurred, and the task may
query the fault analysis area.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *loc* | WORD | location code |
| *qual* | WORD | Fault qualifier |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Fault is logged. |

## Q\_CLEAR

Initialize Queue

### SYNTAX

status = Q\_CLEAR (qid);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    CHAR qid;
    CHAR q_entry[];

    status = Q_GETID (TASK0_Q, &qid); // get queue id
    
    switch (status):
    {
        case SUCCESS:  // queue exists; clear queue
            status = Q_CLEAR (&qid);
            break;
        default:
        case FAILURE:  // invalid queue name
            LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
    }
}
```

### DESCRIPTION

Initialize the specified queue

The queue control block pointers are reset to indicate there are no
queued items.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *qid* | CHAR\* | Physical queue id |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Queue initialized. |
| FAILURE | Invalid queue id. |

## Q\_GET

Get Item from Linked-list Queue

### SYNTAX

status = Q\_GET (qid, item\_pp);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    CHAR qid;
    CHAR q_entry[];

    status = Q_GETID (TASK0_Q, &qid); // get queue id
    
    switch (status):
    {
        case SUCCESS:  // queue exists; get queue item
            if ( Q_GET (&qid, &q_entry) == SUCCESS)
            {
                process_queue_item(q_entry)
            }
            break;
        default:
        case FAILURE:  // invalid queue name
            LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
    }
}

```

### DESCRIPTION

Get the next item from a linked list queue.

Items returned from the queue have a link pointer of type BYTE\* as the
first two bytes of the returned item.

Queue discipline is first-in-first-out.

Exclusive access is guaranteed to the calling task, and the task isn't
suspended if there are no queued items.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *qid* | CHAR\* | Queue id |
| *item\_pp* | CHAR\*\* | Address for queue item |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Item dequeued. |
| FAILURE | Invalid queue id. |
| LIMIT | Queue is empty. |

## Q\_GETID

Get Queue Identifier

### SYNTAX

status = Q\_GETID (q\_name, qid);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    CHAR qid;
    CHAR q_entry[];

    status = Q_GETID (TASK0_Q, &qid); // get queue id
    
    switch (status):
    {
        case SUCCESS:  // queue exists; add queue entry
            q_entry = "This is a queue entry"
            status = Q_PUT (&qid, q_entry);
            break;
        default:
        case FAILURE:  // invalid queue name
            LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
    }
}
```

### DESCRIPTION

Get the linked-list queue identifier, given a queue name. The identifier
is used for queue access primitives.

The queue is a linked list, with a first-in-first-out discipline.

The queue name is a value between zero and 255. These are defined in the
configuration file, beginning with zero and progressing sequentially for
the number of queues defined.

The queue control blocks are allocated in the kernel address space. The
only data structure requirement imposed on the application is that the
first two bytes of the queued item be reserved for linked list
management. However, this pointer should never be written to by the
application.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *q\_name* | INT | Queue name |
| *qid* | CHAR\* | Queue id |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Queue id available. |
| FAILURE | Invalid queue name. |

## Q\_PUT

Add Item to Linked-list Queue

### SYNTAX

status = Q\_PUT (qid, item\_p);

```c
#include <gxos.h>
#include <gxconfig.h>

int main(void)
{
    CHAR qid;
    CHAR q_entry[];

    status = Q_GETID (TASK0_Q, &qid); // get queue id
    
    switch (status):
    {
        case SUCCESS:  // queue exists; add queue item
            q_entry = "This is a queue entry"
            if (Q_PUT (&qid, q_entry) == SUCCESS)
            {
                continue_processing();
            }
            else
            {
                LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, "Queue PUT failed");
            }
            break;
        default:
        case FAILURE:  // invalid queue name
            LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, "Invalid queue name");
    }
}
```

### DESCRIPTION

Add an item as the last entry of a linked list queue.

The queue is a linked list. Items to be queued must reserve a link
pointer of type BYTE\* as the first two bytes of the item to be queued.

Queue discipline is first-in-first-out.

The calling task is guaranteed exclusive access to the queue and isn't
suspended.

There is no limit on the number of items that may be queued, other than
the availability of memory for items to queue.

An error status is returned if an attempt is made to queue an item more
than once to any queue.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *qid* | CHAR\* | Queue id |
| *item\_p* | CHAR\* | Queue item location |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Item queued. |
| FAILURE | Invalid queue id or item is already queued. |

## RECV

Wait for Message from Task

### SYNTAX

status = RECV(msgid\_p, msg\_pp, msgsiz\_p, tout\_val);

```c
#include <gxos.h>
#include <gxconfig.h>

int TASK_0(void)
{
    INT msgid_p;
    CHAR *msg_pp;
    WORD msgsiz_p;

    DO_FOREVER
        {
        // wait for message with timeout
        status = RECV(&msgid_p, &msg_pp, &msgsiz_p, SEC_10);
        
        switch (status):
        {
            case SUCCESS:           // message received
                process_msg(msgid_p, msg_pp, msgsiz_p);
                break;
            case TIMEOUT:           // timeout occurred
                handle_timeout();
                break;
            default:                // request failed
            case FAILURE:
                LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
                break;
        }
    }
}
```

### DESCRIPTION

This primitive complements the SEND synchronization primitive to receive
a message from a task.

The receiving task continues to receive any queued messages as long as
they are available, without suspension, until a task of higher priority
becomes ready to run. Care should be taken in system design to ensure
control is released to the scheduler within the watchdog timer period.

When the task resumes after the RECV call, the received message is
removed from the task\'s message queue and any timeout request is
canceled.

If a timeout occurs before a message is received, the task becomes
active according to its assigned priority.

The message address returned is the location of the message provided by
the task that sent the message. The message area is managed by the
sending and receiving tasks.

Message id and message content agreement between origination and
destination tasks is defined at compile time.

This primitive may not be called from an interrupt service routine.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *msgid\_p* | INT\* | Address for message id tag |
| *msg\_pp* | CHAR\*\* | Address for message location |
| *msgsiz\_p* | WORD\* | Address for message length |
| *tout\_val* | INT | Suspension timeout |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Message available. |
| FAILURE | Parameter error. |
| TIMEOUT | Timeout occurred. |

## SEND

Send message to task

### SYNTAX

status = SEND (tid, msgid, msg\_p, msgsiz);

```c
#include <gxos.h>
#include <gxconfig.h>

int TASK_1(void)
{
    TID tid_p;
    INT msgid_p;
    CHAR *msg_pp;
    WORD msgsiz_p;
    CHAR task1_msg[]

    DO_FOREVER
        {
        // wait for message with timeout
        status = RECV(&msgid_p, &msg_pp, &msgsiz_p, SEC_10);
        
        switch (status):
        {
            case SUCCESS:           // message received
                // send a message to TASK_0
                GETTID (TASK_0, &tid_p);
                task1_msg = "I have a message for you"
                status = SEND (tid_p, TASK_1_MSG, task1_msg, sizeof(task1_msg));
                break;
            case TIMEOUT:           // timeout occurred
                handle_timeout();
                break;
            default:                // request failed
            case FAILURE:
                LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
                break;
        }
    }
}
```

### DESCRIPTION

This primitive provides intertask communication, using messages, as a
task synchronization mechanism.

The destination task must exist and receives the message with the RECV
primitive. The destination task may be the current task.

If a higher-priority task is waiting for a message, the sending task is
suspended. The sending task resumes as the active task according to its
assigned priority.

Multiple messages may be queued to a task, and are serviced with a
first-in-first-out discipline. Also, messages are posted to a task
regardless of whether or not a RECV is currently pending.

This is an asynchronous operation because the sending task doesn't wait
for a response from the destination task. End-to-end confirmation of
message delivery is done by the application.

Message identifiers and message content are defined between message
origination and destination tasks at design time.

No message data is copied during the message transfer, only the pointer
to the message is passed to the destination task; it is the
responsibility of both tasks to coordinate allocation and freeing of the
message data area. The message location and size parameters refer only
to application message data. Memory doesn't need to be reserved for
message management because this is handled by the kernel.

The total number of messages that may be active at any given time is
defined in the system configuration file.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *tid* | BYTE\* | Destination task id |
| *msgid* | INT | Message id tag |
| *msg\_p* | CHAR\* | Message location |
| *msgsiz* | WORD | Message length |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Message sent to task. |
| FAILURE | Unable to send message; invalid destination or resources not available. |

## SIGNAL

Signal Event Occurrence

### SYNTAX

status = SIGNAL (tid, event\_id);

```c
#include <gxos.h>
#include <gxconfig.h>

int TASK_1(void)
{
    TID tid_p;

    DO_FOREVER
        {
        // wait for message with timeout
        status = WAIT (EVT_0 & EVT_2, EVT_AND, SEC_1);
        
        switch (status):
        {
            case SUCCESS:        // all events received
                // send EVT_1 signal to TASK_0
                GETTID (TASK_0, &tid_p);
                status = SIGNAL (tid_p, EVT_1);
                break;
            case TIMEOUT:        // timeout occurred
                handle_timeout();
                break;
            default:             // request failed
            case FAILURE:
                LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
                break;
        }
    }
}
```

### DESCRIPTION

This primitive provides a synchronization mechanism by signaling a task
that one or more events occurred.

The task to be signaled must exist. The task can be the current task.

The *event\_id* is a user-defined bit map with each bit corresponding
to an
Event. Events may be defined between tasks for a total of 16 events
per
task, or at the system level for a total of 16 system events. Event
agreement between tasks is a design issue.

If a higher priority task is waiting for the event(s), and all wait
criteria are
satisfied, the signaling task is suspended. The task resumes as the
active
task, according to its assigned priority.

If all wait criteria of the signaled task aren't satisfied, the event
is posted
to the signaled task, whether or not the task has a WAIT request
pending.

The task identifier of the task to be signaled must be gotten using
the GETTID or GETMYTID primitives.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *tid* | BYTE\* | Task identifier |
| *event\_id* | WORD | Event identifier |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Task signaled. |
| FAILURE | Task doesn't exist or corrupted data detected. |

## WAIT

Wait for event occurrence

### SYNTAX

status = WAIT (evt\_desc, e\_logic, tout\_val);

```c
#include <gxos.h>
#include <gxconfig.h>

int TASK_1(void)
{
    DO_FOREVER
        {
        // wait for message with timeout
        status = WAIT (EVT_0 & EVT_2, EVT_AND, SEC_1);
        
        switch (status):
        {
            case SUCCESS:        // all events received
                process_event();
                break;
            case TIMEOUT:        // timeout occurred
                handle_timeout();
                break;
            default:             // request failed
            case FAILURE:
                LOG_WARN (LY_0 + SS_0 + LV_L + P0 + 0, NOT_USED);
                break;
        }
    }
}
```

### DESCRIPTION

This primitive complements SIGNAL and allows a task to wait for one or
more events to occur.

The *event\_id* is a user-defined bit map with each bit corresponding
to an
event; events may be defined between tasks or at the system level, and
agreement between tasks is a design issue.

The calling task may request a wait for all specified events to occur
(AND-
conditional), or for any one of the events to occur (OR-conditional).

A WAIT condition is considered satisfied when, either a logical AND
condition was specified and all requested events were
received, or a
logical OR condition was specified and any of the
requested events were
received.

All events are cleared, including those that weren't used to complete
the wait, when the task is resumed.

This primitive may not be called from an interrupt service routine.

### PARAMETERS

| Name | Type | Description |
| --- | --- | --- |
| *evt\_desc* | WORD | Event(s) specification bit map |
| *e\_logic* | INT | EVT\_OR OR condition EVT\_AND AND condition |
| *tout\_val* | INT | Suspension timeout |

### RETURN

| Value | Description |
| --- | --- |
| SUCCESS | Requested event(s) occurred. |
| FAILURE | Invalid parameter. |
| TIMEOUT | Timeout occurred before requested event(s). |
