# IPC & Virtualization

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · [Deadlocks](os_03_deadlocks.md) · [Memory Management](os_04_memory_management.md) · [File Systems/IO](os_05_file_systems_and_io.md) · **IPC/Virtualization** · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · [OS Practice Problems](os_08_os_interview_practice_problems.md) · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

## Inter-Process Communication — the mechanisms, compared

| Mechanism | Speed | Use case |
|---|---|---|
| **Pipe** (`pipe()`) | fast | unidirectional, byte-stream, between related processes (parent/child) — the mechanism behind shell `|` |
| **Named pipe / FIFO** | fast | same as pipe, but has a filesystem path, so unrelated processes can use it |
| **Shared memory** (`shm_open`/`mmap`, or `System V shmget`) | fastest | direct shared address-space region — but you must build your own synchronization (semaphores, typically) since the OS provides no automatic coordination, only the shared memory itself |
| **Message queue** | moderate | structured, discrete messages (not a raw byte stream like a pipe), kernel manages queueing/ordering |
| **Sockets** (Unix domain, or TCP/IP loopback) | moderate-fast | works across machines too (TCP/IP), not just same-machine — the most general-purpose option, at some cost vs. shared memory |
| **Signals** | fast, but very limited payload | async notification only (a signal number, essentially) — not for data transfer, for "something happened" events |

- **The precise trade-off worth stating:** shared memory is fastest because there's no copying and no kernel mediation on each access, but it pushes all the synchronization burden onto you; everything else (pipes, message queues, sockets) has the kernel do some work per message (copying data across the user/kernel boundary at minimum) in exchange for handling ordering/buffering/synchronization for you. This is the same fundamental trade-off as file 05's page-cache/direct-IO discussion and file 02's lock-free-vs-locked discussion: more kernel/library-managed convenience vs. more raw performance with more of your own responsibility.

## Virtualization — VMs vs. Containers, precisely

- **Virtual Machine:** a **hypervisor** (Type 1/"bare-metal" like ESXi/KVM, or Type 2/hosted like VirtualBox) virtualizes actual hardware — each VM runs its own full OS kernel, believing it has its own real hardware (CPU, memory, disk, NIC — often with hardware-assisted virtualization, Intel VT-x/AMD-V, trapping and emulating privileged instructions). Strong isolation (a compromised VM's kernel can't directly touch the host kernel), but heavier — full OS boot, more memory overhead per instance.
- **Container:** shares the **host's kernel** — isolation is achieved via kernel features, not a separate kernel:
  - **Namespaces** — isolate what a process can *see* (its own view of process IDs, network interfaces, mount points, hostname, users) — a containerized process has PID 1 inside its own PID namespace even though it's some arbitrary PID on the host.
  - **cgroups (control groups)** — limit and account for what a process can *use* (CPU shares, memory limits, I/O bandwidth) — this is the mechanism that actually enforces a container's resource limits, distinct from namespaces which only control visibility, not resource allocation.
  - Much lighter than a VM (no second kernel to boot, near-native performance since there's no hardware emulation layer), but weaker isolation boundary (a kernel vulnerability can potentially be exploited to escape a container's namespace/cgroup confinement, since all containers share literally the same kernel).
- **The precise interview-ready summary:** "a VM virtualizes hardware and runs a full separate kernel per instance; a container virtualizes the OS interface (via namespaces for isolation and cgroups for resource limits) while sharing one kernel across all containers on the host — that's the entire source of the performance-vs-isolation trade-off between the two."

## Practice signal

"Why would you choose shared memory over sockets for two processes on the same machine" → raw throughput/latency, at the cost of hand-rolling your own synchronization — a good answer names the specific synchronization primitive you'd add (a semaphore pair, or a lock-free ring buffer if throughput is critical, tying back to file 02). "What actually isolates one container from another" → namespaces (visibility) + cgroups (resource limits), and naming that they share one kernel is the key fact that explains container escape vulnerabilities as a category.

---

## Related Modules

- [OS 02 — Synchronization (shared memory needs its own locking)](os_02_synchronization_and_concurrency.md)
- [Networking 05 — Sockets (sockets as IPC)](../networking-course/networking_05_sockets_programming.md)
