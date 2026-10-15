# File Systems & I/O

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · [Deadlocks](os_03_deadlocks.md) · [Memory Management](os_04_memory_management.md) · **File Systems/IO** · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · [OS Practice Problems](os_08_os_interview_practice_problems.md) · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

## The Inode Model (Unix/Linux)

- A file's **name** and its actual data/metadata are separate: a directory entry maps a filename to an **inode number**; the inode holds metadata (size, permissions, timestamps, pointers to the actual data blocks on disk) — but *not* the filename itself.
- **Why this matters, concretely:** multiple directory entries (hard links) can point to the same inode — the "file" has no single canonical name, just a reference count of how many names point to it. `rm` doesn't necessarily delete data — it removes one directory entry and decrements the inode's link count; data is only actually freed when the link count *and* open-file-descriptor count both hit zero. This is the precise explanation for the classic gotcha: deleting a file that a running process still has open doesn't free disk space until that process closes the file (or exits) — `df` and `du` disagreeing is the visible symptom, worth naming if asked "why is my disk full but I can't find the big files."

## Buffered I/O vs. Direct I/O

- **Buffered (the default):** reads/writes go through the **page cache** — the kernel caches recently-read/written file data in otherwise-unused RAM, so repeated access to the same data is served from memory, and writes can be batched/reordered before actually hitting disk (`write()` returning doesn't mean data is durably on disk yet — `fsync()` forces that).
- **Direct I/O (`O_DIRECT`):** bypasses the page cache entirely, going straight to the device. Used when the application manages its own caching more intelligently than the generic kernel cache would (databases are the classic example — they know their own access patterns and don't want a second, redundant cache layer competing for the same RAM), or when consistent, predictable latency matters more than average-case throughput (the kernel cache's behavior can be less predictable under memory pressure).
- **`mmap()`:** maps a file directly into a process's virtual address space — reads/writes become regular memory accesses (page faults pull data in lazily), avoiding explicit `read()`/`write()` syscalls entirely for file access. Efficient for large files accessed non-sequentially/randomly, but has real gotchas: a `SIGBUS` if the file is truncated by another process while mapped, and it interacts with the same page cache as buffered I/O underneath — this is a common trivia distinction (mmap isn't "faster than the page cache," it *is* the page cache, just exposed as memory rather than syscalls).

## IO Scheduling (relevant if disk/storage performance comes up)

- The kernel's I/O scheduler decides the order pending disk requests are actually issued in — matters more for spinning disks (where request *reordering* to minimize physical seek distance is a huge win) than SSDs (no seek penalty, though request *merging* and queue depth still matter).
- Worth knowing by name: **CFQ** (Completely Fair Queuing, older default, fairness-oriented), **deadline** (bounds worst-case latency per request), **noop** (minimal reordering, appropriate for SSDs/hardware that does its own scheduling), and **mq-deadline/BFQ** (modern multi-queue-aware schedulers on current Linux, since I/O scheduling moved to the `blk-mq` multi-queue framework to actually scale with multi-core, high-IOPS NVMe devices — the older single-queue schedulers were themselves a bottleneck at NVMe speeds, which is why this changed).

## Practice signal

If a question drifts toward "why is this write-heavy service slow," a strong OS-literate answer distinguishes whether it's actually disk-bound (check I/O wait, `iostat`) versus page-cache/memory-pressure-bound (cache thrashing under RAM pressure, visible via reclaim activity) versus genuinely CPU-bound — naming the specific diagnostic (`iostat`, `vmstat`) rather than a vague "check the disk" is what separates a strong answer here.

---

## Related Modules

- [OS 04 — Memory Management (page cache)](os_04_memory_management.md)
- [Networking 08 — Troubleshooting (iostat)](../networking-course/networking_08_troubleshooting_and_design_questions.md)
