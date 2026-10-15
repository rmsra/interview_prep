# Concurrency in Python (spend real time here)

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** [Index](python_00_README_start_here.md) · [Core Language Features](python_01_core_language_features.md) · [OOP in Python](python_02_oop_in_python.md) · [Decorators/Generators](python_03_functional_features_and_decorators.md) · [Memory Model/GIL](python_04_memory_model_and_gil.md) · **Concurrency in Python** · [Typing/Modern Python](python_06_typing_and_modern_python.md) · [Interview Trivia](python_07_interview_common_questions.md) · [Testing/Tooling](python_08_testing_and_tooling.md)

---

Three genuinely different concurrency models exist in Python, and the single most common interview question in this area is **"when would you use which"** — a decision-tree question, not a syntax-recall one. Get the decision tree right; the syntax is secondary.

## The Decision Tree

| Workload | Reach for | Why |
|---|---|---|
| I/O-bound, moderate concurrency (dozens of connections/files/requests) | `threading` | GIL releases during I/O waits (file 04) — threads genuinely overlap while waiting, cheap to create |
| I/O-bound, **high** concurrency (thousands of connections) | `asyncio` | avoids per-thread OS overhead entirely — single thread, cooperative scheduling, scales to far more concurrent "waiters" than OS threads reasonably can |
| CPU-bound (real computation, not waiting) | `multiprocessing` | the **only** one of the three that achieves genuine parallel execution across cores — each process has its own interpreter, its own GIL, no shared lock to serialize on |

**The one-sentence version, worth having ready verbatim:** *"Threading and asyncio both help with waiting — I/O-bound work — because the GIL isn't the bottleneck there; only multiprocessing helps with computing, because it's the only one that sidesteps the GIL by using separate processes instead of threads within one interpreter."*

## `threading`

```python
import threading

def worker(n):
    print(f"processing {n}")

threads = [threading.Thread(target=worker, args=(i,)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()
```
- Real OS threads, but GIL-serialized for actual Python bytecode execution (file 04) — genuinely concurrent only during GIL-released I/O waits.
- **`threading.Lock`** — the same mutual-exclusion concept as the OS course's mutex, needed whenever multiple threads touch shared mutable state, *despite* the GIL — the GIL prevents low-level memory corruption from concurrent refcount updates, but does **not** prevent higher-level race conditions in your own code (e.g., a non-atomic `check-then-act` sequence like `if not cache.has(k): cache.set(k, compute())` can still race between threads even with a GIL present). This is a genuinely common misconception worth correcting precisely: **the GIL is not a substitute for your own locking.**

## `multiprocessing`

```python
from multiprocessing import Pool

def cpu_heavy(n):
    return sum(i*i for i in range(n))

with Pool(processes=4) as pool:
    results = pool.map(cpu_heavy, [10_000_000] * 4)   # genuinely runs on 4 separate cores
```
- Each worker is a **separate OS process** with its own Python interpreter and its own memory space — no shared state by default, which is precisely what sidesteps the GIL (there's no single shared refcount-protecting lock to contend over, because there's no shared interpreter at all).
- **The real cost, worth naming precisely:** process creation overhead (much heavier than a thread) and **inter-process communication cost** — since memory isn't shared, passing data between processes means serializing (via `pickle`, by default) and copying it across a pipe/queue, which is real, measurable overhead for large data. This is *why* multiprocessing helps CPU-bound work but can actually be slower than a single-threaded approach for a task that's mostly moving large amounts of data with only a little computation per chunk — the serialization cost dominates.
- **`Pool.map`** is the common entry point — splits an iterable across worker processes, collects results, correctly ordered.

## `asyncio`

```python
import asyncio

async def fetch(session, url):
    async with session.get(url) as resp:
        return await resp.text()

async def main():
    async with aiohttp.ClientSession() as session:
        results = await asyncio.gather(*(fetch(session, u) for u in urls))

asyncio.run(main())
```
- **Single-threaded, cooperative concurrency** — there is exactly one thread, running one event loop. Concurrency comes from a coroutine voluntarily yielding control (at every `await`) back to the event loop, which then runs whatever other ready coroutine is waiting — not from OS-level preemption the way threads work.
- **`async def`** defines a coroutine function — calling it doesn't run the body, it returns a coroutine *object* (directly parallel to how calling a generator function returns a generator object without running the body, file 03) — you must `await` it (or schedule it via `asyncio.gather`/`create_task`) to actually run it.
- **The precise mental model, worth stating exactly this way if asked "how is asyncio different from threading":** with threading, the OS scheduler can preempt a thread at any bytecode instruction (though the GIL still serializes actual execution); with asyncio, control only ever transfers at an explicit `await` point — your own code decides exactly where it can be interrupted, nowhere else. This makes asyncio code easier to reason about for shared-state correctness (no `Lock` needed for state that's never touched across an `await` boundary) but means **a single coroutine that never awaits (e.g., doing heavy CPU computation inline) blocks the entire event loop** — every other coroutine waits, since there's no preemption to save you the way there is with threads.
- **Why it scales past threading for extreme I/O concurrency:** an OS thread has real overhead (a stack allocation, kernel scheduling structures) — thousands of threads is expensive and eventually hits OS limits (the same C10K problem from the networking course's sockets file). A coroutine is just a lightweight Python object the event loop tracks — tens of thousands of concurrent coroutines is routine, which is precisely the same "many-connections-on-one-thread" idea as `epoll`, just implemented at the language level instead of the OS-syscall level.

## Practice signal

The decision-tree question (I/O-bound → threading/asyncio, CPU-bound → multiprocessing) is close to guaranteed in some phrasing. A strong follow-up answer, if pushed further on threading vs. asyncio specifically: threading for *moderate* I/O concurrency where you don't want to restructure existing synchronous code around `async`/`await`; asyncio for *high* I/O concurrency or when the ecosystem you're working in (many modern HTTP clients, web frameworks) is already async-native. If asked to write concurrent code live, `asyncio.gather` for "run these I/O operations concurrently" and `multiprocessing.Pool` for "parallelize this computation" are the two highest-probability correct answers depending on which kind of workload the prompt describes.

---

## Related Modules

- [Python 04 — Memory Model/GIL (why the GIL exists)](python_04_memory_model_and_gil.md)
- [Networking 05 — Sockets (epoll = same many-on-one-thread idea as asyncio)](../networking-course/networking_05_sockets_programming.md)
- [OS 01 — Processes/Threads/Scheduling](../os-course/os_01_processes_threads_and_scheduling.md)
