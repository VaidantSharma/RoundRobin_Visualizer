## How the Round Robin algorithm works

Round Robin (RR) is a preemptive scheduling method that allocates a fixed time slice (the time quantum) to each ready process in a cyclic order. Processes execute for up to one quantum; if a process does not complete within its allotted quantum, it is preempted and appended to the tail of the ready queue. The cycle repeats until all processes have finished execution.

Operational summary:

1. Simulation time advances in discrete ticks.
2. Processes that arrive at the current tick are enqueued to the ready queue.
3. The scheduler dispatches the process at the front of the ready queue to the CPU and grants it up to one time quantum.
4. While executing, the process's remaining CPU time and the quantum remainder decrement on each tick.
   - If remaining CPU time reaches zero, the process completes and is recorded as finished.
   - If the quantum expires before completion, the process is preempted and re-queued at the tail.
5. The scheduler terminates when every process has completed.

This approach enforces bounded CPU share per process and provides round-robin fairness across runnable processes.

## Why it is necessary

Round Robin is appropriate for systems that require predictable responsiveness and equitable CPU allocation among concurrent tasks. By bounding how long any process may hold the CPU, RR prevents indefinite monopolization and reduces worst-case response latency for individual processes.

Primary motivations:

- Fairness: every runnable process receives periodic CPU access in turn.
- Responsiveness: interactive or short-lived tasks obtain CPU service within a bounded delay proportional to the queue length and quantum.
- Predictability: system behavior under load is easier to reason about because the maximum contiguous CPU time per process is limited.

## Why it can be preferable to other schedulers

Relative advantages compared with common alternatives:

- FCFS (First-Come, First-Served): RR avoids starvation of short or interactive tasks and reduces perceived latency by preempting long-running jobs.
- SJF (Shortest Job First): SJF optimizes average turnaround time but requires accurate knowledge or estimation of remaining CPU time and may cause starvation of long jobs; RR operates without such predictions and enforces fairness.
- Priority scheduling: priority-based approaches can starve lower-priority tasks unless additional mechanisms (e.g., aging) are applied; RR provides equitable sharing by design.

Considerations and trade-offs:

- Context-switch overhead: shorter quanta increase preemption frequency and thus context-switch cost.
- Quantum selection: the time quantum should balance responsiveness and overhead — a very small quantum increases overhead, whereas a very large quantum reduces preemptive benefits and approaches FCFS behavior.

## How it is implemented in the code

Reference: `components/round-robin-scheduler.tsx`.

Implementation summary:

- Framework: React client component using functional hooks (`useState`, `useEffect`). The component includes a visualization of the ready queue, CPU execution area, and a Gantt chart for schedule inspection.

- Principal state values and data structures:

  - `processes`: Array of process objects { id, name, burstTime, arrivalTime, remainingTime, color }.
  - `timeQuantum`: Configurable integer specifying the quantum length.
  - `currentTime`: Integer simulation clock.
  - `readyQueue`: FIFO array representing processes waiting for CPU service.
  - `currentProcess`: Process object currently executing, or null when CPU is idle.
  - `quantumLeft`: Remaining quantum for the currently executing process.
  - `ganttChart`: Array capturing dispatch events (process name and start time) for post-hoc visualization.
  - `completedProcesses`: Array of finished processes with recorded completion times.
  - `contextSwitches`: Counter incremented on CPU switches.

- Control and utility functions:

  - `addProcess()`, `removeProcess(id)`, `updateProcess(id, field, value)`: manage the process list.
  - `reset()`: returns the simulation to the initial state and resets remaining times.
  - `calculateMetrics()`: computes completion, turnaround, waiting times and averages based on `completedProcesses`.

- Scheduling loop (driven by a `useEffect` while `isRunning` is true):

  1. The loop advances the simulation clock at an interval controlled by the `speed` parameter.
  2. Newly arriving processes (arrivalTime === currentTime) are enqueued onto `readyQueue`.
  3. If the CPU is idle and `readyQueue` is non-empty, the scheduler dequeues the head and dispatches it to the CPU, setting `quantumLeft = timeQuantum` and logging the dispatch into `ganttChart`.
  4. If a process is executing, each tick decrements its `remainingTime` and `quantumLeft` by one. Two cases follow:
     - Completion: when `remainingTime` reaches zero, the process is recorded in `completedProcesses` with its completion timestamp and the CPU is freed.
     - Preemption: when `quantumLeft` reaches zero but `remainingTime` > 0, the process is appended to the tail of `readyQueue` and the CPU is freed for the next dispatch.
  5. The simulation stops when all processes are accounted for in `completedProcesses`.

- Metrics and visualization: the component derives per-process turnaround and waiting times and displays average metrics and a Gantt chart to aid analysis.

Notes for modification and analysis:

- Adjust `timeQuantum` to study the impact of quantum size on average waiting/turnaround and context switches.
- Use `speed` to alter tick duration for faster or slower visualization during demonstrations.

(This document contains only the requested sections: algorithm description, rationale, comparative advantages, and implementation notes.)
