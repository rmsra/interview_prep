# Processes, Threads, and Scheduling

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · **Processes/Threads/Scheduling** · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · [Deadlocks](os_03_deadlocks.md) · [Memory Management](os_04_memory_management.md) · [File Systems/IO](os_05_file_systems_and_io.md) · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · [OS Practice Problems](os_08_os_interview_practice_problems.md) · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

## Process vs Thread — the precise version

| | Process | Thread |
|---|---|---|
| Memory | own address space | shares address space with sibling threads |
| Creation cost | expensive (`fork()`: copy page tables, though copy-on-write makes this cheaper than it sounds) | cheap (`pthread_create()`: new stack + register set, shares everything else) |
| Communication | needs IPC (pipes, shared memory, sockets — file 06) | direct, via shared memory — but needs synchronization (file 02) |
| Crash isolation | one process crashing doesn't take down another | one thread crashing (segfault) takes down the whole process |
| Context switch cost | higher (full address space switch — TLB flush territory) | lower (same address space, just registers/stack pointer) |

- **What a thread actually owns, precisely** (a common trivia-style question): its own stack, its own register set/program counter, its own thread-local storage. It shares: heap, global/static variables, open file descriptors, signal handlers (though each thread can have its own signal mask).
- **`fork()` and copy-on-write:** `fork()` doesn't actually copy the parent's entire memory immediately — it duplicates the page table with pages marked read-only and copy-on-write; an actual physical copy of a page only happens when either process writes to it. This is why `fork()` is cheap in practice despite conceptually "copying everything," and it's a common follow-up if you claim to know why `fork()` isn't as expensive as it sounds.

## Context Switching — what actually happens

1. Save current process/thread's CPU state (registers, program counter, stack pointer) into its **PCB (Process Control Block)** / TCB.
2. Scheduler picks the next thread/process to run.
3. Restore the new thread's saved state.
4. If switching between **processes** (not just threads within one process): also switch the **page table base register** (e.g., `CR3` on x86), which invalidates the **TLB** (translation lookaside buffer — the cache of virtual-to-physical address translations) unless the CPU supports tagged TLBs (PCID on x86) to avoid the full flush.
- **Why this is expensive, precisely:** it's not the register save/restore itself (fast) — it's the **cache and TLB cold-start** afterward. The new process's working set isn't in cache, so you eat a burst of cache misses right after switching in, on top of the TLB flush. This is the real reason context-switch-heavy designs (like thread-per-connection at high connection counts) degrade — not the switch mechanics themselves, but everything that follows a switch running slow until caches warm back up. Directly connects to your own cache/false-sharing story.

## CPU Scheduling Algorithms

| Algorithm | Idea | Trade-off |
|---|---|---|
| FCFS (First-Come-First-Served) | run in arrival order | simple, but a long job blocks everything behind it ("convoy effect") |
| SJF (Shortest Job First) | run the shortest job next | optimal average wait time — but requires knowing job length in advance, which is often unknowable |
| Round Robin | fixed time quantum per process, cycle through | fair, bounded response time — but quantum too small = context-switch overhead dominates; too large = degenerates toward FCFS |
| Priority Scheduling | highest priority runs first | can starve low-priority tasks indefinitely without aging |
| Multilevel Feedback Queue | multiple queues by priority, processes move between queues based on observed behavior (CPU-bound vs I/O-bound) | what most real general-purpose OS schedulers approximate — adapts without needing to know job length in advance |
| CFS (Completely Fair Scheduler, actual Linux default) | models "fairness" via a virtual runtime per task, always picks the task with the least accumulated virtual runtime (implemented as a red-black tree — the exact structure from the DSA course's trees file) | good general-purpose fairness, not tuned for hard real-time guarantees out of the box |

- **Starvation vs. Aging:** starvation is a low-priority task never running because higher-priority tasks keep arriving; **aging** is the standard fix — gradually increase a waiting task's priority the longer it waits, guaranteeing it eventually becomes highest-priority.
- **Real-time scheduling, relevant to your background:** standard CFS optimizes average-case fairness, not worst-case latency guarantees. Real-time systems (like yours) typically need `SCHED_FIFO`/`SCHED_RR` (Linux real-time scheduling classes — these preempt any normal `SCHED_OTHER`/CFS task unconditionally) combined with `isolcpus`/core pinning to get predictable, bounded latency instead of best-effort fairness. Worth stating this contrast directly if asked about scheduling in the context of your own systems.

## Practice signal

"Why would you choose a thread pool over spawning a thread per request" → context-switch/memory overhead at scale, plus resource exhaustion — directly the same C10K-style reasoning as the networking course's epoll discussion; these two are the same underlying problem viewed from different layers, worth explicitly connecting if both come up in the same interview.

---

## Related Modules

- [OS 02 — Synchronization/Concurrency](os_02_synchronization_and_concurrency.md)
- [Networking 05 — Sockets (C10K, thread-per-connection)](../networking-course/networking_05_sockets_programming.md)
- [DSA 05 — Trees (CFS's red-black tree)](../dsa-course/dsa_05_trees_tries_union_find.md)
