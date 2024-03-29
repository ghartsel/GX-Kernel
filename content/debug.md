---
weight: 50
title: Debugging
---


# Debugging

The data structures described here are useful for debugging to get
visibility into kernel and task states.

These data are used exclusively by the kernel and never need to be
referenced by application software.

Refer to the section on memory layout for the location of these data,
which are shown below as C- language structures.

## Kernel Control Data

```c
typedef struct
{
    FAA faa;               // fault analysis area
    CHAR monstk[SSTK_SIZ]; // initialization stack
    QCB sk_q_pr0;          // task scheduler queues
    QCB sk_q_pr1;
    QCB sk_q_pr2;
    QCB msg_free_q;        // free message queue
    CHAR istate;           // kernel state
    TCB *curtsk;           // currently active task
    TCB *flttsk;           // fault management task
    INT task_cnt;          // total number of tasks
    WORD clktik_cnt;       // continuous timer counter
    CHAR clktik_skip;      // timer rate adjustment
    CHAR curbnk;           // currently active bank
    TCB *tcbblk_p;         // locations: task control blocks
    QCB *queblk_p;         // queue resource blocks
    CHAR *semblk_p;        // semaphore block
    MSG_D *msgblk_p;       // message resource block
    CHAR *clkblk_p;        // interval timer block
    CHAR *dmemblk_p;       // memory allocation ctrl.
    CHAR *sstktop_p;       // top of supervisor stack
} KDD;
```

(data<br>
structure<br>
element<br>
descriptions<br>
to be<br>
provided)

## Task Control Block

These data describe a task instance.

```c
typedef struct
{
    BYTE *tcb_link;   // linked list pointer
    CHAR tname;       // task name
    CHAR tattr;       // task-type data marker
    INT (*ent_pt)();  // task entry point
    QCB *priority_qp; // task priority
    CHAR *stktop;     // top of task stack
    CHAR *stkend;     // end of task stack
    CHAR *stk_p;      // current stack pointer
    QCB w_que;        // task wait queue
    CHAR state;       // current task state
    EVT_D evt;        // task event management
    QCB msg_q;        // task message list
} TCB;
```

(data<br>
structure<br>
element<br>
descriptions<br>
to be<br>
provided)

## Task Queue Control Block

These data describe the state of a task message queue.

```c
typedef struct
{
    CHAR q_mark; // queue-type data marker
    CHAR q_sem;  // queue semaphore
    BYTE **head; // head item pointer
    BYTE **tail; // tail item pointer
} QCB;
```

(data<br>
structure<br>
element<br>
descriptions<br>
to be<br>
provided)

## Task Event Descriptor

These data describe the state of events associated with a task.

```c
typedef struct
{
    CHAR logic;   // event request condition
    WORD evt_wt;  // pending event map
    WORD evt_sig; // posted event map
} EVT_D;
```

(data<br>
structure<br>
element<br>
descriptions<br>
to be<br>
provided)

## Task Message Descriptor

These data describe the task message state.

```c
typedef struct
{
    BYTE *msglnk; // message link
    CHAR msgmrk;  // message-type data marker
    INT msgid;    // message identification
    INT msgsiz;   // message size
    CHAR *msg_p;  // message location
} MSG_D;
```

(data<br>
structure<br>
element<br>
descriptions<br>
to be<br>
provided)

## Fault Analysis Area

This data area is used to log fault information reported by the
[LOG\_FATAL](#log_fatal) and [LOG\_WARN](#log_warn) primitives.

Its content reflects the system state when the fault was reported.

For the [LOG\_WARN](#log_warn) primitive, only *fault\_loc* and *fault\_qual* data
are logged.

Task-related data isn't logged if no task is active when the fault is
reported.

> Fault Analysis Area

```c
typedef struct
{
    CHAR r_A;        // processor registers: A
    CHAR r_B;        //                      B
    WORD r_IX;       //                      IX
    WORD r_IY;       //                      IY
    WORD sp;         //                      SP
    WORD pc;         //                      PC
    CHAR cc;         //                      CC
    CHAR istate;     // kernel state
    CHAR *curtcb_p;  // active task
    CHAR curtcb[32]; // active task control block
    CHAR ustk[16];   // task stack
    CHAR sstk[16];   // supervisor stack
    WORD fault_loc;  // fault location
    WORD fault_qual; // fault qualifier data
} FAA;
```

(data<br>
structure<br>
element<br>
descriptions<br>
to be<br>
provided)
