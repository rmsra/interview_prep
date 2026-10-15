# Cisco Onsite Operating Systems Course — Start Here

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** **Index** · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · [Deadlocks](os_03_deadlocks.md) · [Memory Management](os_04_memory_management.md) · [File Systems/IO](os_05_file_systems_and_io.md) · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · [OS Practice Problems](os_08_os_interview_practice_problems.md) · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

Built for the "Development" and "Debugging" interview focus areas Tanique flagged — OS concepts (concurrency, memory, scheduling) are where those two buckets overlap most, and it's also where interviewers most often ask you to write real code live (synchronization primitives, producer-consumer, thread pools).

This course covers the concepts; `cpp-course/cpp_07_concurrency_in_cpp.md` covers the C++ standard-library API on top of them (`std::thread`, `std::async`, `std::atomic`) — read both together for the concurrency questions specifically.

## Priority given your background

Like the networking course: you're not learning this for the first time. You've built multi-threaded, real-time, high-throughput systems (DDDS worker pools, false-sharing fixes, priority ring buffers). The risk isn't understanding — it's **fluency in the classic vocabulary and classic problems** interviewers use as a shared language, even when your day-to-day reasoning has moved past needing to consciously name them.

- **Spend real time:** synchronization/concurrency (file 02) and the classic interview problems (file 08) — producer-consumer, readers-writers, dining philosophers come up constantly as live-coding prompts, and knowing the *named* solution pattern is faster than re-deriving it live.
- **Calibration pass, you know this:** processes/threads/scheduling (file 01), memory management (file 04) — refresh vocabulary and the 2-3 non-obvious details, don't relearn from scratch.
- **Your home turf:** file 07 (kernel internals, real-time systems, NUMA, lock-free) is where your actual experience is strongest — written to help you connect it to standard OS-course vocabulary.
- **Useful but lower-frequency:** file 05 (file systems/IO), file 06 (IPC/virtualization) — solid to know, less likely to be the centerpiece of a question.
- **Added after reviewing the SAI job posting:** file 09 (debugging/profiling tools — GDB, PDB, Valgrind, perf, eBPF) — the posting names GDB/PDB explicitly as a preferred qualification, and it's real, resume-backed tooling you already use.

## Files

1. [`os_01_processes_threads_and_scheduling.md`](os_01_processes_threads_and_scheduling.md) — process vs thread, context switching, scheduling algorithms
2. [`os_02_synchronization_and_concurrency.md`](os_02_synchronization_and_concurrency.md) — mutexes, semaphores, condition variables, atomics, memory ordering — **spend the most time here**
3. [`os_03_deadlocks.md`](os_03_deadlocks.md) — conditions, prevention/avoidance/detection, livelock
4. [`os_04_memory_management.md`](os_04_memory_management.md) — virtual memory, paging, TLB, page faults, allocators
5. [`os_05_file_systems_and_io.md`](os_05_file_systems_and_io.md) — inodes, buffered vs direct IO, IO scheduling
6. [`os_06_ipc_and_virtualization.md`](os_06_ipc_and_virtualization.md) — pipes, shared memory, message queues, containers vs VMs
7. [`os_07_linux_kernel_and_real_time_systems.md`](os_07_linux_kernel_and_real_time_systems.md) — interrupts, NUMA, lock-free structures, RCU — your home turf
8. [`os_08_os_interview_practice_problems.md`](os_08_os_interview_practice_problems.md) — producer-consumer, readers-writers, dining philosophers, thread pool, worked in C++
9. [`os_09_debugging_and_profiling_tools.md`](os_09_debugging_and_profiling_tools.md) — GDB, Valgrind, ASan/TSan, perf, flame graphs, eBPF, Wireshark, PDB

## Format

Same as the networking course: **the mechanism**, **C/C++ where it's codeable**, **the gotcha/misconception interviewers probe**, **how to frame it if asked directly**.
