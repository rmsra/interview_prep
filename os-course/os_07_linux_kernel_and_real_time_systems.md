# Linux Kernel Internals & Real-Time Systems (your home turf)

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · [Deadlocks](os_03_deadlocks.md) · [Memory Management](os_04_memory_management.md) · [File Systems/IO](os_05_file_systems_and_io.md) · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · **Linux Kernel/Real-Time** · [OS Practice Problems](os_08_os_interview_practice_problems.md) · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

This file is less "learn new material" and more "here's the standard OS-course vocabulary for things you already do." Use it to translate your real-time/DPDK experience into terms a generalist OS interviewer will recognize precisely.

## Interrupts vs. Polling — the general OS framing of a decision you've already made

- **Interrupt-driven:** hardware signals the CPU asynchronously when something needs attention (packet arrived, disk I/O done); CPU stops what it's doing, runs an **interrupt handler** (kept short — often splits into a fast "top half" that does minimal urgent work and defers the rest to a "bottom half"/softirq/tasklet run later, so the interrupt handler itself doesn't block other interrupts too long).
- **Polling:** software repeatedly checks a status register/flag itself, no async signal at all.
- **The trade-off, stated generally (this is the framing to use even outside a DPDK-specific question):** interrupts are efficient when events are infrequent relative to CPU speed — the CPU does other useful work between events instead of wasting cycles checking. Polling wins when events are frequent enough that the interrupt overhead (context switch into the handler, cache pollution from switching contexts) exceeds the cost of just checking directly — which is exactly the regime high-packet-rate networking lives in, and precisely why DPDK's poll-mode drivers outperform interrupt-driven kernel networking at your throughput levels. This is the OS-theory-level version of the argument you'd make packet-processing-specific in the networking course's file 07.

## NUMA (Non-Uniform Memory Access)

- On multi-socket systems, each CPU socket has its own **local** memory bank; accessing memory attached to a *different* socket ("remote" memory) is slower — hence "non-uniform."
- **NUMA-aware design, precisely:** allocate memory on the same NUMA node as the CPU core that will primarily access it (`numactl`, or DPDK's own NUMA-aware `mempool` allocation), and pin threads to cores on that same node, so hot-path memory access stays local. Getting this wrong (a thread on node 0 constantly touching memory allocated on node 1) is a real, measurable performance cliff that's easy to miss without specifically profiling for it (`numastat` is the tool to name).
- **Why this belongs in an "OS" bucket and not just "hardware":** the OS scheduler and memory allocator are NUMA-aware by default on Linux (it tries to keep threads and their memory co-located automatically) — but default heuristics can be wrong for specialized high-throughput workloads, which is exactly why systems like yours override the default with explicit pinning/allocation rather than trusting the general-purpose scheduler.

## Lock-Free / RCU — beyond basic atomics (file 02 covers the atomics primitives; this is how the kernel actually uses them at scale)

- **RCU (Read-Copy-Update):** a synchronization technique, heavily used inside the Linux kernel, optimized for **read-mostly, write-rare** data structures — readers proceed with **zero locking overhead** (not even an atomic operation in the common case), while writers create a new copy of the data, atomically swap the pointer to it, and wait for all pre-existing readers to finish (a "grace period") before freeing the old version.
- **Why it beats a reader-writer lock for this access pattern:** an RWLock still makes readers pay some synchronization cost (even shared-mode locking involves atomic operations and potential cache-line bouncing on the lock itself); RCU readers pay essentially nothing, at the cost of writers being more expensive and readers potentially seeing slightly stale data during the grace period (RCU provides eventual, not immediate, consistency for writers vs. readers — worth being precise about this, since claiming RCU gives the same consistency guarantees as a lock is a common but wrong simplification).
- **Where this connects to your own work:** conceptually similar territory to your dual-priority-ring design — both are cases where a general-purpose synchronization primitive (a plain mutex/lock, or a single unified queue) was traded for a more specialized structure that exploits the actual access pattern (read-heavy for RCU; priority-tiered for your ring design) to avoid unnecessary synchronization cost. Naming RCU by name, even briefly, signals kernel-level fluency beyond application-level concurrency.

## `isolcpus`, IRQ affinity, and building a real-time-ish system on general-purpose Linux

- **`isolcpus`** (kernel boot parameter): removes specified cores from the general scheduler's load-balancing pool entirely — the OS won't schedule normal tasks onto them unless a process is explicitly pinned there (`taskset`/`sched_setaffinity`). This is what makes a core available for a DPDK worker to have essentially undisturbed, dedicated access to.
- **IRQ affinity** (`/proc/irq/*/smp_affinity`): by default, hardware interrupts can land on any core, disturbing whatever's running there — for a latency-sensitive pinned worker, you also need to steer interrupts *away* from its dedicated core(s), or the isolation is incomplete (a NIC interrupt landing on your "isolated" core still preempts your worker).
- **The precise, honest framing if asked "is this real real-time":** standard Linux (even heavily tuned this way) gives you *soft* real-time — very good, very consistent low-latency behavior in practice, but no formally guaranteed worst-case bound the way a true RTOS (or `PREEMPT_RT` patched kernel, worth naming as the closer-to-hard-real-time option) provides. Being precise about this distinction, rather than overclaiming "real-time" loosely, is itself a signal of seniority.

## Practice signal

If an interviewer asks you to explain your own DDDS/CU-UP architecture, this file is the vocabulary to reach for to make it legible to someone grading against a standard OS curriculum: name the interrupt-vs-poll trade-off explicitly, name NUMA if your workers were pinned with memory locality in mind, and be precise about "soft real-time" rather than overclaiming — precision here reads as more senior than a bigger claim would.

---

## Related Modules

- [OS 02 — Synchronization (atomics underneath RCU)](os_02_synchronization_and_concurrency.md)
- [OS 01 — Scheduling (SCHED_FIFO/SCHED_RR)](os_01_processes_threads_and_scheduling.md)
- [OS 09 — Debugging Tools (eBPF/XDP vs. kernel bypass)](os_09_debugging_and_profiling_tools.md)
- [Networking 07 — DPDK/Your Story](../networking-course/networking_07_dpdk_kernel_bypass_and_your_story.md)
