# Synchronization & Concurrency (spend the most time here)

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · **Synchronization/Concurrency** · [Deadlocks](os_03_deadlocks.md) · [Memory Management](os_04_memory_management.md) · [File Systems/IO](os_05_file_systems_and_io.md) · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · [OS Practice Problems](os_08_os_interview_practice_problems.md) · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

This is the OS-course equivalent of the DSA course's DP file — the topic most likely to be both explained *and* live-coded, and the one where "I've used mutexes" and "I understand the memory model" genuinely diverge.

## The core primitives, precisely differentiated

| Primitive | What it protects | Key property |
|---|---|---|
| **Mutex** | mutual exclusion — one thread in a critical section at a time | has ownership — only the thread that locked it can unlock it; blocks the calling thread |
| **Semaphore** | a resource count | no ownership concept — any thread can `signal`/`post`; a **binary semaphore** looks like a mutex but isn't one (no ownership, so thread A can lock and thread B can unlock — legal but a design smell if unintentional) |
| **Spinlock** | mutual exclusion, but busy-waits instead of sleeping | avoids the cost of a context switch/syscall — good only when the critical section is very short and you expect the wait to be shorter than a context switch would cost; bad on a single-core system (the lock holder can't run to release it while the spinner burns the only core) |
| **Condition Variable** | waiting for a *state change*, not just exclusion | always paired with a mutex — used to sleep a thread until some condition becomes true, avoiding busy-polling |

```cpp
#include <mutex>
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> q;

// producer
{
    std::lock_guard<std::mutex> lock(mtx);
    q.push(item);
}
cv.notify_one();

// consumer
std::unique_lock<std::mutex> lock(mtx);
cv.wait(lock, [] { return !q.empty(); });   // atomically releases the lock while waiting, reacquires before returning
int item = q.front();
q.pop();
```
- **Why `cv.wait` takes a predicate/lambda, not just a bare wait:** protects against **spurious wakeups** (the OS is allowed to wake a waiting thread even without a real `notify` — POSIX explicitly permits this for implementation reasons) and against the **lost wakeup** problem (if you check the condition, then call `wait()`, a `notify` that happens in that gap is lost forever) — the predicate form re-checks the condition atomically with re-acquiring the lock, closing both holes. Writing `wait()` without the predicate and manually looping (`while(!condition) cv.wait(lock);`) is equivalent and worth knowing as the "what's actually happening under the hood" version.
- **`std::lock_guard` vs `std::unique_lock`:** `lock_guard` is a strict RAII lock, non-movable, minimal overhead — use when you just need scope-based lock/unlock. `unique_lock` is more flexible (can unlock/relock manually, required by `condition_variable::wait` because it needs to unlock while waiting) at slightly higher overhead — the rule of thumb is default to `lock_guard`, reach for `unique_lock` only when you need the extra flexibility (condition variables, deferred locking, transferring ownership).

## Atomics & Memory Ordering — where real fluency separates candidates

```cpp
#include <atomic>
std::atomic<int> counter{0};
counter.fetch_add(1, std::memory_order_relaxed);   // atomic, but no ordering guarantee with other memory ops
```

- **Why atomics exist beyond "avoid a data race":** even a single `int++` is not atomic at the machine level — it's read-modify-write, three separate steps, so two threads can interleave and lose an increment. `std::atomic` guarantees the RMW is indivisible.
- **Memory ordering is the part people gloss over.** Modern CPUs and compilers **reorder instructions** for performance as long as single-threaded behavior looks unchanged — but that reordering can be visible to *other threads* in a multi-threaded program. `std::memory_order` controls what ordering guarantees an atomic operation provides to other threads:
  - `memory_order_relaxed` — atomicity only, no ordering guarantee with surrounding operations. Fastest, use when you only care about the operation itself (e.g., a pure counter no one's using to synchronize anything else).
  - `memory_order_acquire` (on loads) / `memory_order_release` (on stores) — the pairing that implements the classic "publish a value, then a flag; another thread checks the flag, then reads the value safely" pattern. A `release` store guarantees all writes *before* it in program order are visible to another thread that does an `acquire` load of the same variable and sees that value — this is what actually makes a lock-free flag-based handoff safe, not just "it's atomic so it's fine."
  - `memory_order_seq_cst` (sequentially consistent, the **default** if you don't specify) — the strongest and simplest to reason about: all threads agree on a single global order of all seq_cst operations. Safest default, but has real performance cost on weakly-ordered architectures (ARM, in particular) compared to acquire/release, because it may require extra memory fence instructions.
- **The interview-precise way to say this:** "acquire/release is a *synchronizes-with* relationship between one thread's store and another thread's load of the same atomic — it's not a global ordering guarantee, it's a pairwise one, which is exactly why it's cheaper than seq_cst and why using it correctly requires you to be precise about which operations are actually paired."

## False Sharing — your own story, generalized

- Two variables that are logically unrelated but happen to land on the **same cache line** (typically 64 bytes) — writes by different cores to their own variable invalidate the *entire line* in the other core's cache, forcing constant cross-core cache-coherence traffic (via MESI or similar protocols) even though there's no actual data race.
- **Fix:** pad/align structures so independent per-thread/per-core data lands on separate cache lines — `alignas(64)` in standard C++, or `__rte_cache_aligned` in DPDK specifically (your own fix). This is a pure performance issue, not a correctness one — the program is *correct* without the fix, just slow, which is exactly what makes it a good "debugging" story: nothing looks wrong until you profile it.
- **How to frame this if asked "how did you find it":** cache-coherence traffic shows up as unexplained scaling limits (throughput doesn't grow linearly with added cores/threads despite no obvious lock contention) — worth naming `perf c2c` (Linux's cache-to-cache sharing analysis tool) as the concrete diagnostic if pushed on methodology, since "I profiled it" alone is a weaker answer than naming the actual tool.

## Deadlock-adjacent concurrency bugs, quickly (full deadlock treatment is file 03)

- **Race condition:** outcome depends on timing/interleaving of concurrent operations — the general category; a lost update from an unprotected `counter++` is a race condition.
- **Priority Inversion:** a low-priority task holds a lock a high-priority task needs, and a *medium*-priority task (which needs no lock) preempts the low-priority holder, indirectly blocking the high-priority task for longer than the lock hold time alone would explain. Famous real-world case: Mars Pathfinder. **Priority inheritance** (the low-priority holder temporarily inherits the waiting high-priority task's priority until it releases the lock) is the standard fix, worth naming.
- **ABA problem** (specific to lock-free code using compare-and-swap): a value changes from A to B and back to A between a thread's read and its CAS — the CAS succeeds because the value matches, but the underlying state actually changed in between and that change might matter (e.g., in a lock-free stack, the memory could have been freed and reallocated). Fixed with tagged pointers/version counters (pair the value with a monotonically increasing counter, so "A" at version 5 is distinguishable from "A" at version 7) or hazard pointers. Worth knowing this exists even if you'd never hand-roll lock-free code live in an interview — it signals real depth if the conversation goes there.

## Practice signal

Live-coding synchronization primitives (implement a simple thread-safe queue, a bounded buffer, a rate limiter) is common — file 08 has the classic named versions worked through. If asked to explain a bug in someone else's threaded code, the diagnostic order is: is there a shared mutable variable touched without synchronization (race) → is there a lock-ordering cycle (deadlock, file 03) → is there a performance cliff with no correctness bug (false sharing, or lock contention/priority inversion).

---

## Related Modules

- [OS 03 — Deadlocks](os_03_deadlocks.md)
- [OS 08 — Practice Problems (producer-consumer, thread pool)](os_08_os_interview_practice_problems.md)
- [OS 09 — Debugging Tools (perf c2c, TSan/Helgrind for races)](os_09_debugging_and_profiling_tools.md)
- [Networking 07 — DPDK (your own false-sharing story)](../networking-course/networking_07_dpdk_kernel_bypass_and_your_story.md)
