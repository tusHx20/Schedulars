[CPU_Scheduling_Theory (1).md](https://github.com/user-attachments/files/33397561/CPU_Scheduling_Theory.1.md)
# CPU Scheduling Algorithms — Theory

## 1. Why CPU Scheduling?

In a multiprogramming operating system, several processes reside in memory
at once, but a single CPU (or core) can execute only one process at a time.
The **CPU scheduler** decides which process in the *ready queue* gets the
CPU next. A good scheduling algorithm tries to:

- Maximize CPU utilization and throughput (processes completed per unit time)
- Minimize waiting time, turnaround time, and response time
- Be fair to all processes

### Key Terms

| Term | Meaning |
|---|---|
| **Arrival Time (AT)** | The time at which a process enters the ready queue |
| **Burst Time (BT)** | The total CPU time a process needs to finish |
| **Completion Time (CT)** | The time at which a process finishes execution |
| **Turnaround Time (TAT)** | `TAT = CT − AT` — total time spent in the system |
| **Waiting Time (WT)** | `WT = TAT − BT` — time spent waiting in the ready queue |
| **Priority** | A number indicating how urgent a process is (used in Priority Scheduling) |

Scheduling algorithms are also classified as:
- **Non-preemptive**: once a process starts running, it keeps the CPU until it
  finishes or voluntarily gives it up.
- **Preemptive**: the OS can forcibly take the CPU away from a running
  process (e.g., when its time slice ends) and give it to another process.

---

## 2. First Come First Served (FCFS)

**Type:** Non-preemptive

**Idea:** Processes are executed strictly in the order they arrive — like a
queue at a ticket counter. The process that arrives first is scheduled
first.

**Algorithm:**
1. Sort all processes by arrival time.
2. Run them one after another with no interruption.

**Advantages**
- Extremely simple to understand and implement.
- No starvation — every process eventually runs.

**Disadvantages**
- **Convoy effect**: if a long process arrives first, all shorter processes
  behind it must wait a long time, even though they need very little CPU
  time. This drags down average waiting time.
- Not ideal for time-sharing/interactive systems.

---

## 3. Shortest Job First (SJF) — Non-Preemptive

**Type:** Non-preemptive

**Idea:** Among all the processes that have already arrived, always pick
the one with the **smallest burst time** to run next.

**Algorithm:**
1. At each scheduling point, look at all processes that have arrived and
   are not yet finished.
2. Pick the one with the smallest burst time (break ties by earliest
   arrival).
3. Run it to completion, then repeat.

**Advantages**
- Provably gives the **minimum possible average waiting time** among
  non-preemptive algorithms, for a fixed set of processes.

**Disadvantages**
- Requires knowing the burst time in advance, which is usually not
  possible in a real system (it's often *estimated*).
- **Starvation**: a long process may wait forever if shorter processes
  keep arriving.

---

## 4. Priority Scheduling — Non-Preemptive

**Type:** Non-preemptive (a preemptive version also exists)

**Idea:** Each process is assigned a priority number. The CPU is given to
the process with the **highest priority** among those that have arrived.
(Convention used in this project: a **smaller number = higher priority**,
e.g., priority 1 runs before priority 4.)

**Algorithm:**
1. Among arrived, unfinished processes, pick the one with the best
   (numerically smallest) priority value.
2. Run it to completion, then repeat.

**Advantages**
- Important/urgent processes (e.g., system processes) can be given
  precedence over less important ones.

**Disadvantages**
- **Starvation**: a low-priority process may never run if high-priority
  processes keep arriving.
- **Solution**: *Aging* — gradually increase the priority of processes that
  have been waiting a long time.

---

## 5. Round Robin (RR)

**Type:** Preemptive

**Idea:** Designed for time-sharing systems. Each process is given a small,
fixed unit of CPU time called a **time quantum** (e.g., 2 ms). If the
process doesn't finish within that quantum, it is preempted and sent to the
back of the ready queue, and the next process in line gets its turn.

**Algorithm:**
1. Maintain a circular queue of ready processes.
2. Take the process at the front, run it for `min(quantum, remaining time)`.
3. If it still has remaining burst time, put it back at the end of the
   queue. If it finishes, remove it.
4. Add any newly arrived processes to the queue before re-queuing the
   current one (standard rule, used in this project's code).
5. Repeat until all processes are done.

**Advantages**
- Fair: every process gets a regular turn on the CPU.
- Good response time for interactive/time-sharing systems.
- No starvation.

**Disadvantages**
- Performance heavily depends on the size of the time quantum:
  - **Too large** → behaves like FCFS (poor response time).
  - **Too small** → too much time wasted in context switching (overhead).
- Average waiting time can be **higher** than SJF/Priority for the same
  workload, as seen in this project's comparison output.

---

## 6. Comparison at a Glance

| Algorithm | Preemptive? | Starvation? | Best For | Weakness |
|---|---|---|---|---|
| FCFS | No | No | Batch systems, simplicity | Convoy effect |
| SJF | No | Yes | Minimizing avg. waiting time | Needs future knowledge of burst time |
| Priority | No | Yes | Task urgency-based systems | Low-priority processes may starve |
| Round Robin | Yes | No | Time-sharing / interactive systems | Choice of quantum is critical |

---

## 7. How This Maps to the Code

The accompanying `cpu_scheduler.py` implements each algorithm as a separate
function (`fcfs`, `sjf_non_preemptive`, `priority_scheduling`,
`round_robin`) that all take the same list of `Process` objects, so results
are directly comparable. Each function returns:

1. The list of processes with their `completion_time`, `turnaround_time`,
   and `waiting_time` filled in.
2. A **Gantt chart** list showing the exact order and time slices in which
   processes ran.

The `main()` function runs all four algorithms on one sample process set
and prints a final summary table comparing their average waiting and
turnaround times — this is the core "comparison" your project asks for.
