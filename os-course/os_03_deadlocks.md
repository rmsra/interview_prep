# Deadlocks

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · **Deadlocks** · [Memory Management](os_04_memory_management.md) · [File Systems/IO](os_05_file_systems_and_io.md) · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · [OS Practice Problems](os_08_os_interview_practice_problems.md) · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

## The Four Necessary Conditions (Coffman conditions) — all four must hold simultaneously

1. **Mutual Exclusion** — at least one resource is held in a non-shareable mode.
2. **Hold and Wait** — a thread holds at least one resource while waiting to acquire another.
3. **No Preemption** — a resource can only be released voluntarily by the thread holding it.
4. **Circular Wait** — a cycle exists in the "waiting-for" graph (thread A waits on a resource held by B, which waits on one held by A, ...).
- **The interview-precise point:** breaking *any one* of these four is sufficient to prevent deadlock — that's the entire menu of prevention strategies, and stating it this way (rather than listing ad-hoc fixes) signals you understand the structure of the problem, not just memorized folk remedies.

## Worked Example — AB-BA Deadlock in a Linked-List Node Swap

A concrete illustration of circular wait, worth more than the abstract A-waits-B-waits-A description: this is a bug that looks completely reasonable until you trace two threads through it.

### The setup

```c
struct ListNode {
    int value;
    struct ListNode *next;
    pthread_mutex_t *lock;
};

void swap(struct ListNode *n1, struct ListNode *n2) {
    pthread_mutex_lock(n1->lock);
    pthread_mutex_lock(n2->lock);

    int temp = n1->value;
    n1->value = n2->value;
    n2->value = temp;

    pthread_mutex_unlock(n1->lock);
    pthread_mutex_unlock(n2->lock);
}
```

Single-threaded, this is fine: `swap(n2, n5)` on the list `1->2->3->4->5` produces `1->5->3->4->2` — only the two values exchange, list structure untouched.

### The bug: lock order follows argument order, not a fixed order

`swap(n1, n2)` always locks its first argument, then its second. So the *same two nodes*, called with arguments reversed, acquire the locks in **opposite order**:

```
Thread1: swap(n1, n2)
Thread2: swap(n2, n1)

t1: Thread1 locks n1
t2: Thread2 locks n2
t3: Thread1 tries to lock n2 — held by Thread2, blocks
t4: Thread2 tries to lock n1 — held by Thread1, blocks
```

Both threads now hold one lock and wait forever on the other. All four Coffman conditions present simultaneously: mutual exclusion (the mutexes), hold-and-wait (each holds one, waits on the other), no preemption, circular wait (Thread1→n2→Thread2→n1→Thread1). Not a race, not a partial result — a permanent hang, zero output.

### Tempting but wrong fix #1 — order locks by comparing `->value`

```c
if (n1->value < n2->value) { lock(n1); lock(n2); }
else                       { lock(n2); lock(n1); }
```

Looks like a fixed ordering scheme. It isn't, for two independent reasons:

**1. Equal values reproduce the exact same deadlock.** `swap(X, Y)` and `swap(Y, X)` with `X->value == Y->value` both fall into the `else` branch — but with `n1`/`n2` swapped between the two calls, that means *opposite* lock order. Same AB-BA deadlock, and `swap()` itself is what creates equal values, so the precondition for the bug gets more likely to occur the more the buggy function runs.

**2. Reading `->value` before acquiring any lock is a data race — undefined behavior, not just unspecified.** You're reading a field another thread may be concurrently writing inside its own critical section, with no synchronization between the read and the write. This is a strictly worse problem than the tie case: it's UB regardless of whether the values happen to be equal.

### Tempting but wrong fix #2 — skip the swap when values are equal

```c
if (n1->value == n2->value) return;  // "no need to swap"
```

Fixes the tie case narrowly. Does not fix the data race (the equality check itself still reads unsynchronized). And there's a sharper counterexample that needs no tie at all — a **TOCTOU (time-of-check to time-of-use) race on the ordering key**:

```
Initial: X=5, Y=3, Z=3

Thread1: swap(X, Y) — reads X=5, Y=3. 5<3? false → decides: lock Y, then lock X.
Thread3: swap(Y, Z) — runs to completion before Thread1 acquires anything.
                       Y's value changes 3 → 9.
Thread2: swap(Y, X) — reads Y=9, X=5. 9<5? false → decides: lock X, then lock Y.

Thread1 now locks: Y, then X
Thread2 now locks: X, then Y
```

Opposite order again — same deadlock — no tie anywhere in the trace. Thread1's decision was correct *at the instant it read the values*; a third thread changed the value before Thread1 acted on its decision. The tie case and the TOCTOU case are the same underlying flaw: **`value` is mutable, and it is specifically the field this operation mutates.** No amount of special-casing removes that; it can never be a safe ordering key.

### The correct fix — order by something invariant, never by the mutated field

```c
void swap(struct ListNode *n1, struct ListNode *n2) {
    struct ListNode *first  = (n1 < n2) ? n1 : n2;
    struct ListNode *second = (n1 < n2) ? n2 : n1;

    pthread_mutex_lock(first->lock);
    pthread_mutex_lock(second->lock);

    int temp = n1->value;
    n1->value = n2->value;
    n2->value = temp;

    pthread_mutex_unlock(second->lock);
    pthread_mutex_unlock(first->lock);
}
```

`swap(n1,n2)` and `swap(n2,n1)` now lock the same node first, every time, regardless of argument order — no cycle possible, and there's no TOCTOU window because pointer identity never changes for the life of the node.

**Interview-precise point:** comparing two unrelated heap pointers with `<` is technically *unspecified behavior* per the C standard (not undefined — universally reliable in practice on flat-address-space hardware, and exactly what real lock-ordering code, including the Linux kernel, does). Worth knowing the distinction between "works everywhere in practice" (pointer comparison) and "guaranteed-broken" (comparing a field the operation itself mutates) if pushed on it. Fully standard-compliant alternative: a stable unique ID assigned once at node creation, never touched again, instead of the raw address.

**Other valid fixes worth naming if asked "what else could you do":** `pthread_mutex_trylock` on the second lock, releasing and retrying on failure (no ordering scheme needed, risks livelock under contention); in C++11, `std::lock(m1, m2)` acquires both simultaneously using an internal deadlock-avoidance algorithm.

**The one-sentence version for an interview:** breaking any one of the four Coffman conditions prevents deadlock — this fix breaks circular wait by imposing a total order on lock acquisition that's independent of both argument order and any value the operation mutates.

## Prevention (break one of the four conditions structurally)

- **Break circular wait — the most common real fix:** impose a global total ordering on lock acquisition (e.g., "always lock the lower memory address / lower ID first"), and require every thread to acquire locks in that order. Guarantees no cycle can ever form.
```cpp
// classic example: transferring between two accounts, locking both
void transfer(Account& a, Account& b, int amount) {
    Account& first  = (&a < &b) ? a : b;   // order by address — arbitrary but consistent
    Account& second = (&a < &b) ? b : a;
    std::lock_guard<std::mutex> lock1(first.mtx);
    std::lock_guard<std::mutex> lock2(second.mtx);
    // ... perform transfer ...
}
```
- **`std::lock` / `std::scoped_lock` (C++17):** locks multiple mutexes together using a deadlock-avoidance algorithm internally (effectively try-lock-and-backoff), so you don't have to hand-roll the ordering yourself:
```cpp
std::scoped_lock lock(a.mtx, b.mtx);   // acquires both, deadlock-free, regardless of call order elsewhere
```
- **Break hold-and-wait:** require a thread to request all resources it will ever need up front, atomically — rarely practical (you often don't know what you'll need until you're partway through), but worth naming as one of the four theoretical options.
- **Break no-preemption:** allow the OS/runtime to forcibly reclaim a resource from a holder — rare in application-level code, more common in OS-level resource management.

## Avoidance — Banker's Algorithm (worth knowing the name and idea, not the full derivation)

- Rather than statically preventing deadlock structurally, dynamically checks before granting each resource request: "if I grant this, does the system remain in a **safe state**" (a state from which there's still some ordering of remaining requests that lets every thread eventually complete)? Only grants requests that preserve safety.
- Requires knowing the maximum resource needs of each process in advance — a strong assumption that limits real-world applicability, which is exactly why prevention (lock ordering) is what's actually used in practice far more often than avoidance. Worth stating this contrast if asked "why don't we just always use Banker's algorithm."

## Detection & Recovery

- Build a **resource-allocation graph** (or wait-for graph); periodically check for cycles. If found, deadlock is confirmed.
- **Recovery options:** kill one or more processes in the cycle (simplest, but loses work), or preempt resources from one of them and roll back.
- **Where this is actually used:** databases commonly use detection + recovery (aborting/rolling back one transaction in the cycle) rather than prevention, because in a database, lock ordering is often impossible to guarantee in advance across arbitrary application queries — worth this concrete example if asked where each strategy actually shows up in real systems.

## Deadlock vs. Livelock vs. Starvation — precisely distinguished (a common trivia cluster)

- **Deadlock:** threads are blocked, waiting on each other, permanently — nothing progresses, nothing changes state.
- **Livelock:** threads are *not* blocked — they're actively running, continuously changing state in response to each other, but never making actual progress (classic image: two people repeatedly stepping the same direction trying to avoid each other in a hallway). Often caused by overly-polite retry/backoff logic (both sides yield simultaneously, forever).
- **Starvation:** a thread is perpetually denied a resource it needs, even though the system as a whole is making progress — other threads are running fine, just not this one. Doesn't require a cycle at all, just an unfair scheduling/priority policy.

## Practice signal

"Debug this deadlocked code" almost always reduces to spotting inconsistent lock-acquisition order across two or more code paths — read the code specifically hunting for two locks acquired in different orders in different functions, that's the single highest-yield thing to check first. "How would you prevent deadlock in this design" → lead with lock ordering (`std::scoped_lock` in modern C++) as the practical default, mention Banker's/detection only if asked for the fuller theoretical picture.

---

## Related Modules

- [OS 02 — Synchronization/Concurrency](os_02_synchronization_and_concurrency.md)
- [OS 08 — Practice Problems (Dining Philosophers)](os_08_os_interview_practice_problems.md)
- [OS 09 — Debugging Tools (GDB's thread apply all bt)](os_09_debugging_and_profiling_tools.md)
