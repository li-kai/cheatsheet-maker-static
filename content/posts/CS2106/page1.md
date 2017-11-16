---
title: CS2106
description: Introduction to Operating Systems
---

## Hardware utilization

## Process Management

OS be able to switch from program A to B

Process: Currently executing program (running), also Task/JobProcess: Currently executing program (running), also Task/JobProcess: Currently executing program (running), also Task/JobProcess: Currently executing program (running), also Task/JobProcess: Currently executing program (running), also Task/JobProcess: Currently executing program (running), also Task/Job

### Recap: Sample program and assembly code

C program -> memory assembly code + ram

Therefore, we always need processor, memory and i/o for a program.

Memory for code looks like this:

[ assembly code | global data variables | free ]


### Low level function calls

-> Can we simply store function as data?

No, because scope is different. Imagine merge sort with same i,j, wow O(log n).

2 problems

1. Control flow, passing parameters to other funcs

1. Data storage, need to capture return result

Hence, we introduce - stack memory!

#### Stack memory

Used only to support function invocations.

It is:

- Dynamically allocated
- Require a stack pointer to keep track of size
- As stack size grows, stack pointer value goes down (reverse direction of numeration)

Given, an example

```js
function f() {
    g();
}
```

When we first invoke f(), a stack frame for f() is allocated.
Each new invocation in f() adds a stack frame for g() on top of the stack frame.
When g() invokation finish, f().

When memory usage exceeds due to infinite function call, program will crash.

##### Stack frame

Includes:

- parameters
- local variables (e.g. intermediate calc results)
- return PC
- other info

There is no single function call convention.

###### Setup

Caller: pass parameters with registers and/or stack
Caller: save return PC on stack

Callee: save old Stack Pointer (to call back function), save registers used by callee
Callee: allocate space for function
Callee: execute code

###### Teardown

Callee: place return result on stack (if present), restore saved register
Callee: restore saved SP, effectively deallocating itself

Caller: utilize return result
Caller: continues

###### Frame pointer

We can't use stack pointer because it keeps moving due to execution of the program (hence increasing/decreasing stack size).

Frame pointer is static.

#### Heap Memory

Dynamic variable allocation

### Process Id & State

PID is unique among processes

#### 5-state process model

= Create (allocate memory) => New state = admit => Ready state <= switch: scheduled / release => Running state = event wait => Terminated state

Need a Ready Queue and Blocked Queue for multiple processes. These are more like sets than actual queues.

#### Process table

A list of necessary information for processes.
These are called Process Blocks, which then allocate them into memory accordingly.

# Processess

## Concurrent Execution
- Logical concept to cover multitasked process
- There are
    1) virtual parallelism
    1) physical parallelism

## Simplistic Concurrency Model
Using timeslicing to achieve "fake parallelism"

## Interleaved execution (context switch)

Process itself enforces interleaving. The OS will step in.

OS will do context switching.

OS has a scheduler that makes the decision. Has an algorithm.

## Scheduling

Scheduling is not clear cut. Different requirements, also different ways to allocate by environment.

Exists certain criteria to evaluate the scheduler.

### Process behavior

CPU Activity
Compution, Compute Bound Process spend most time here

IO Activity
Interupt to handle user I/O input

#### Types of processing env

1) Batch processing
    No user, no interaction
2) Interactive (or Multiprogramming)
    With activer user interacting, should be responsive
3) Real time
    Dead line to meet
    Usually periodic process

#### Criterias

- Fairness
    Should get a fair share of CPU time, based on per process or per user basis. Meaning, no starvation.
- Balance
    All parts of the computing system should be utilized
- Turnaround time
    Total time taken from start to finish, related to waiting time waiting for CPU
- Throughput
    Number of tasks finished per unit time
- CPU utilization
    Percentage of time when CPU is working on a task
- Response time
    Time between request and response
- Predictability
    Variation in response time, lesser variation == more predictable

#### When to perform scheduling
Two types
- Non-preemptive (Cooperative)
    A process stayed scheduled until it blocks OR gives up CPU voluntarily
- Preemptive
    A process is given a fixed time quota to run, can volunteer to give up

### Scheduling a process

Process Control Board (PCB)
Keeps context of
- ID
- Memory
- Hardware (pointers, etc)

Copied over for every PID

#### Batch processing

E.g. Supercomputers

Non-preemptive scheduling is predominant

- First Come First Served (FCFS)
    + Guaranteed to have no starvation, always be served
    - Convoy effect
    Waiting time is number of process unit time in front of it.
- Shortest Job First (SJF)
    Allow shortest job to run first
    Guess future CPU time bassed on previous CPU-Bound phases (Exponential Average)
- Shortest Remaining Time (SRT)
    Variation of SJF, using remaining time, but is preemptive
    Allow the task with the shortest remaining time (meaning expected - already ran time)
    Stop task if a shortest remaining time, starvation is possible

#### Interactive System

Ensures good response time

Timer interrupt = Interrupt that goes off periodically (based on hardware clock)
Ensures that OS cannot be intercepted

Timer interval (typically 1ms to 10ms)
Different from below because this is the one that OS follows

Time quantum (basically a tick)
- Execution duration given to a process
- Could be constant or variable among the processes
- Large rang eof values (commonly 5ms to 100ms)

- Round Robin
    Tasks are stored in a FIFO queue
    Big quantum: Better CPU utilization but longer waiting
    Small quantum: Bigger overhead
    Time before a task get CPU is bounded by (n - 1)q
    Response time guarantee
- Priority Scheduling
    Low priority process can starve, hack: decrease priority after every quantum
    Hard to guarantee or control the exact amount of CPU time given to a process using priority
    Priority inversion: Low priority holds key resources.
    E.g. Cleaner holds key to office for CEO, CTO
- Multive-level Feedback Queue (MLFQ)
    Adapative, learns the process behavior
    Minimizes both responsive time for IO and turnaround time for CPU
    Basic rules
    - Priority(A) > Priority(B) -> A runs
    - Priority(A) == Priority(B) -> A and B runs in RR
    - New job -> Highest Priority
    - Job fully utilizes time slice -> priority reduced
    - Job gives up / blocks -> retain priority
- Lottery scheduling
    Give out "lottery tickets" for processes.
