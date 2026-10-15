# Concurrency in C++ (the API surface — the conceptual depth lives in the OS course)

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · [OOP/Class Design](cpp_02_oop_and_class_design.md) · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · [Templates](cpp_05_templates_and_generic_programming.md) · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · **Concurrency in C++** · [Exceptions](cpp_08_exceptions_and_error_handling.md) · [Interview Trivia](cpp_09_interview_common_questions.md)

---

**Read `os-course/os_02_synchronization_and_concurrency.md` for the concepts** (mutex vs. semaphore vs. condition variable, atomics and memory ordering, false sharing). This file is deliberately narrower: the C++-specific standard-library API on top of those concepts, since that's what you'll actually be asked to write live in a C++ round.

## `std::thread`

```cpp
#include <thread>

void worker(int id) { /* ... */ }

std::thread t(worker, 42);   // starts immediately
t.join();                    // block until it finishes
// or: t.detach();           // let it run independently — rarely the right choice, see below
```
- **Must `join()` or `detach()` before a `std::thread`'s destructor runs**, or `std::terminate` is called — this is a hard, commonly-tripped rule. RAII wrapping (a small class that joins in its destructor) is the standard fix for exception-safety here, since a thrown exception between thread creation and an explicit `join()` would otherwise skip the join entirely.
- **`detach()` is rarely correct in production code** — a detached thread's lifetime is fully decoupled from the object that launched it, which usually means it can outlive resources it references. Worth being able to say this rather than presenting `detach` and `join` as equally reasonable options.

## `std::async`, `std::future`, `std::promise`

```cpp
std::future<int> fut = std::async(std::launch::async, [] { return compute(); });
int result = fut.get();   // blocks until ready, retrieves the result (or rethrows an exception thrown inside the task)
```
- **`std::async` with `std::launch::async`** guarantees the task runs on a new thread. **Without an explicit policy** (`std::launch::async | std::launch::deferred`, the default), the implementation is free to choose — including running it **lazily on the calling thread only when `.get()` is called**, which surprises people expecting concurrent execution by default. **Always pass `std::launch::async` explicitly if you actually need it to run concurrently** — a real, specific gotcha worth having ready.
- **`future::get()` propagates exceptions** — an exception thrown inside the async task is captured and rethrown at the point `.get()` is called, not where it originally occurred. This is genuinely convenient (no manual error-code plumbing across threads) and worth knowing as the answer to "how do you get an error out of a background task."
- **`std::promise`/`std::future` pair** (the lower-level building block `async` is built on): a `promise` is the *write* side (`set_value()`/`set_exception()`), its paired `future` is the *read* side (`get()`) — useful when you need to hand a result-channel to a thread you're managing manually (e.g., inside a thread pool), rather than letting `std::async` manage the thread for you.

## `std::atomic<T>`

```cpp
std::atomic<int> counter{0};
counter.fetch_add(1, std::memory_order_relaxed);
int snapshot = counter.load(std::memory_order_acquire);
```
- The C++-standard-library expression of the atomics/memory-ordering model covered in depth in the OS course — `.load()`, `.store()`, `.fetch_add()`, `.compare_exchange_weak()`/`.compare_exchange_strong()`, each taking an optional `std::memory_order` (defaulting to `memory_order_seq_cst`, the strongest/safest/slowest option).
- **`compare_exchange_weak` vs. `_strong`:** `_weak` may fail *spuriously* (return false even though the comparison would have succeeded) on some platforms — this is intentional, since it allows a more efficient underlying instruction sequence on architectures without a single atomic compare-and-swap instruction. **Always use `_weak` inside a retry loop** (where a spurious failure just means one extra harmless iteration); use `_strong` only for a genuine one-shot check where a retry loop isn't already present.

## `std::mutex` family, briefly (concepts in the OS course)

```cpp
std::mutex m;
std::lock_guard<std::mutex> lock(m);       // RAII, scope-based, non-movable, minimal overhead
std::unique_lock<std::mutex> lock2(m);     // flexible: can unlock/relock, required by condition_variable::wait
std::scoped_lock lock3(m1, m2);            // C++17: locks multiple mutexes deadlock-free in one call
```
`std::scoped_lock` (C++17) is the modern default for acquiring more than one mutex at once — internally uses a deadlock-avoidance algorithm so lock order across different call sites doesn't matter, directly the fix named in the OS course's deadlocks file for the classic AB-BA problem.

## Practice signal

A C++ concurrency question is much more likely to be "write a thread-safe X using `std::mutex`/`condition_variable`" (see `os-course/os_08_os_interview_practice_problems.md` for the fully worked producer-consumer/thread-pool templates — those are C++ already) than to be "explain memory ordering from scratch." Know the API well enough to write it fluently; the *why* behind memory ordering lives in the OS course, don't duplicate re-deriving it here.

---

## Related Modules

- [OS 02 — Synchronization/Concurrency (the concepts this file's API sits on)](../os-course/os_02_synchronization_and_concurrency.md)
- [OS 08 — Practice Problems (fully worked C++ examples)](../os-course/os_08_os_interview_practice_problems.md)
