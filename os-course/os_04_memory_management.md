# Memory Management

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · [Deadlocks](os_03_deadlocks.md) · **Memory Management** · [File Systems/IO](os_05_file_systems_and_io.md) · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · [OS Practice Problems](os_08_os_interview_practice_problems.md) · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

## Virtual Memory — why it exists, precisely

Every process sees its own private, contiguous **virtual address space**, translated to physical addresses by the MMU (Memory Management Unit) using page tables maintained by the OS. Three distinct benefits worth naming separately (not just "isolation"):
1. **Isolation** — one process can't read/corrupt another's memory (or the kernel's) by accident or maliciously.
2. **Simplifies programming/linking** — every process can assume it owns the same address layout (e.g., code starting at a fixed base) regardless of what else is running, and regardless of how fragmented physical memory actually is.
3. **Enables overcommit and paging to disk** — a process can use more virtual memory than physically exists, with the OS transparently moving cold pages to disk (swap).

## Paging — the actual mechanism

- Physical and virtual memory are divided into fixed-size **pages** (commonly 4KB; **huge pages** at 2MB/1GB reduce TLB pressure for large working sets — the same mechanism your DPDK background already relies on, worth connecting explicitly).
- A **page table** maps virtual page numbers to physical frame numbers, per-process.
- **Multi-level page tables:** a single flat page table for a 64-bit address space would itself be enormous — practically, page tables are hierarchical (e.g., x86-64 uses a 4-level radix tree: PML4 → PDPT → PD → PT), so only the branches actually in use consume real memory, at the cost of multiple memory accesses to walk the tree for each translation.
- **TLB (Translation Lookaside Buffer):** a small, fast cache of recent virtual→physical translations inside the CPU, avoiding the multi-level page-table walk on every single memory access. A **TLB miss** means the CPU (or OS, depending on architecture) has to walk the page table — meaningfully slower. This is exactly why huge pages help: fewer, larger pages means far more of your working set's translations fit in the same fixed number of TLB entries.

## Process Address Space Layout — the picture underneath "stack vs heap"

A single process's virtual address space is conventionally laid out (exact addresses vary by architecture/ASLR, but the relative layout is standard and worth being able to sketch):

```
High addresses
+-------------------+
|       Stack        |  grows downward — local variables, return addresses, saved registers
|         |           |
|         v           |
|                     |
|         ^           |
|         |           |
|        Heap         |  grows upward — malloc/new allocations
+-------------------+
|   BSS segment       |  uninitialized global/static variables (zero-initialized by the OS at load time)
+-------------------+
|   Data segment      |  initialized global/static variables
+-------------------+
|   Text/Code segment |  the compiled instructions — typically read-only + executable
+-------------------+
Low addresses
```
- **Stack and heap grow toward each other** from opposite ends of the available space — historically the classic "they'll collide" failure mode; modern systems with ASLR and typically much larger virtual address spaces than physical make an actual collision rare, but the mental model (bidirectional growth into a shared gap) is still the right one and a common whiteboard-sketch ask.
- **Why BSS is separate from the initialized Data segment:** BSS doesn't need to store the actual zero values on disk in the executable file — the loader just zeroes that memory range at load time, which is why a program with a huge uninitialized global array doesn't bloat the binary's file size the way a huge *initialized* array would (that one has to actually store its initial values in the Data segment, on disk, in the executable).
- **Text segment is shared** across multiple instances of the same running program (and marked read-only, executable) — this is part of why running the same binary many times is cheaper than the naive "each process needs its own full copy" intuition suggests.

## Page Faults — not all "faults," some are normal and expected

| Type | What happened | Handling |
|---|---|---|
| Minor fault | page exists in memory, just not yet mapped into this process's page table (e.g., a shared library page another process already loaded, or a copy-on-write page not yet touched) | cheap — just update the page table, no disk I/O |
| Major fault | page genuinely isn't in physical memory — must be read from disk (swap, or a memory-mapped file) | expensive — actual disk I/O in the critical path |
| Segmentation fault | access to memory the process has no valid mapping for at all (invalid address, or valid address but wrong permissions — e.g., writing to a read-only page) | not resolvable — signal (`SIGSEGV`) delivered, process typically terminates |
- **Copy-on-write, precisely tied to page faults:** after `fork()`, parent and child share physical pages marked read-only; a write by either triggers a **minor fault**, the kernel copies that one page, remaps it read-write for the writer, and execution continues — the process itself never "sees" this happen except as a small latency blip on first write.

## Common Allocators — what to know beyond "malloc/free"

- **`malloc`/`free`** manage a **heap** — typically via a free-list of available chunks; real implementations (glibc's `ptmalloc`, `jemalloc`, `tcmalloc`) use size-class binning (grouping similarly-sized allocations to reduce fragmentation and speed up allocation) and per-thread arenas (to reduce lock contention between threads allocating concurrently — directly a concurrency/false-sharing-adjacent concern from file 02).
- **Fragmentation, two kinds worth distinguishing:**
  - **External fragmentation:** enough total free memory exists, but not as one contiguous block big enough for a given request.
  - **Internal fragmentation:** memory is allocated in fixed-size chunks/pages larger than what was actually requested, wasting the difference inside an allocated block.
- **Why systems like DPDK use custom memory pools instead of `malloc`:** general-purpose allocators aren't optimized for the "allocate/free a huge number of same-sized objects at very high rate" pattern typical of packet buffers — a pre-allocated, fixed-size-object memory pool (like DPDK's `mempool`) avoids allocator overhead and fragmentation entirely for that specific access pattern, trading generality for speed. Good bridge from a generic memory-management question into your own systems background.

## Stack vs. Heap — the precise distinction, not just "one's fast, one's not"

| | Stack | Heap |
|---|---|---|
| Allocation | implicit, on function call/return — just moving a stack pointer | explicit (`malloc`/`new`), managed |
| Lifetime | tied to scope, automatic cleanup | manual (or RAII-managed in C++) — lives until explicitly freed |
| Speed | very fast (pointer bump) | slower (allocator bookkeeping, possible lock contention) |
| Size | fixed, limited (a few MB typically), overflow = crash | large, limited mainly by available (virtual) memory |
| Thread-safety | inherently per-thread, no sharing | shared across threads by default, needs synchronization |

## Practice signal

"Why is a stack overflow different from a segfault, or is it one" → a stack overflow (exceeding the stack's fixed size, often from unbounded/runaway recursion) *is* typically reported as a segfault, because the guard page below the stack is unmapped specifically to turn stack overflow into a detectable fault rather than silently corrupting adjacent memory — a good, precise answer connects these two concepts instead of treating them as unrelated. "Why would huge pages help our workload" is a very plausible question given your background — TLB pressure reduction is the answer, stated above.

---

## Related Modules

- [OS 02 — Synchronization (per-thread arenas)](os_02_synchronization_and_concurrency.md)
- [OS 09 — Debugging Tools (Valgrind/ASan for memory bugs)](os_09_debugging_and_profiling_tools.md)
- [Networking 07 — DPDK (huge pages, memory pools)](../networking-course/networking_07_dpdk_kernel_bypass_and_your_story.md)
