# Classic OS Interview Practice Problems (worked in C++)

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · **Operating Systems** · [C++](../cpp-course/cpp_00_README_start_here.md) · [Python](../python-course/python_00_README_start_here.md)
**Operating Systems modules:** [Index](os_00_README_start_here.md) · [Processes/Threads/Scheduling](os_01_processes_threads_and_scheduling.md) · [Synchronization/Concurrency](os_02_synchronization_and_concurrency.md) · [Deadlocks](os_03_deadlocks.md) · [Memory Management](os_04_memory_management.md) · [File Systems/IO](os_05_file_systems_and_io.md) · [IPC/Virtualization](os_06_ipc_and_virtualization.md) · [Linux Kernel/Real-Time](os_07_linux_kernel_and_real_time_systems.md) · **OS Practice Problems** · [Debugging & Profiling Tools](os_09_debugging_and_profiling_tools.md)

---

These come up as live-coding prompts constantly — knowing the *named* pattern means you're transcribing a known solution instead of deriving synchronization logic under pressure. Same instinct as the DSA course: recognize the pattern, then execute.

## Producer-Consumer (Bounded Buffer)

**Trigger:** any "one or more threads produce items, one or more threads consume them, buffer has limited capacity" framing.

```cpp
#include <mutex>
#include <condition_variable>
#include <queue>

template<typename T>
class BoundedQueue {
    std::mutex mtx;
    std::condition_variable not_full, not_empty;
    std::queue<T> q;
    size_t capacity;
public:
    BoundedQueue(size_t cap) : capacity(cap) {}

    void push(T item) {
        std::unique_lock<std::mutex> lock(mtx);
        not_full.wait(lock, [this] { return q.size() < capacity; });
        q.push(std::move(item));
        lock.unlock();
        not_empty.notify_one();
    }

    T pop() {
        std::unique_lock<std::mutex> lock(mtx);
        not_empty.wait(lock, [this] { return !q.empty(); });
        T item = std::move(q.front());
        q.pop();
        lock.unlock();
        not_full.notify_one();
        return item;
    }
};
```
- **Two condition variables, not one** — `not_full` for producers waiting on space, `not_empty` for consumers waiting on data. Using a single CV works too (just notify_all and let everyone re-check their own predicate) but wakes threads unnecessarily; two CVs is the more precise, commonly-expected answer.
- **Gotcha:** notify *after* unlocking (as above) when possible — avoids the woken thread immediately blocking again on the mutex the notifier still holds (a "hurry up and wait" inefficiency, not a correctness bug, but worth doing right).

## Readers-Writers

**Trigger:** many readers can access data concurrently, but a writer needs exclusive access.

```cpp
#include <shared_mutex>

class RWData {
    mutable std::shared_mutex mtx;
    int data = 0;
public:
    int read() const {
        std::shared_lock lock(mtx);      // multiple readers can hold this simultaneously
        return data;
    }
    void write(int value) {
        std::unique_lock lock(mtx);      // exclusive — blocks all readers and other writers
        data = value;
    }
};
```
- `std::shared_mutex` (C++17) is the standard-library answer — know it exists and use it directly rather than hand-rolling reader/writer counters, unless explicitly asked to build the primitive from scratch.
- **If asked to hand-roll it** (a real follow-up): the classic version uses a reader count + a mutex protecting that count + a second mutex for writer exclusion, and there are two well-known variants — **reader-preference** (readers never wait if other readers are active, which can starve writers indefinitely) and **writer-preference** (once a writer is waiting, new readers block behind it, preventing writer starvation at some cost to reader throughput). Naming this trade-off directly is a strong answer even without writing the hand-rolled version.

## Dining Philosophers

**Trigger:** the canonical deadlock-avoidance problem — N philosophers, N forks, each needs both adjacent forks to eat.

- **The naive version deadlocks:** if every philosopher picks up their left fork simultaneously, all forks are held and everyone waits forever for their right fork — classic circular wait (file 03's four conditions, all present).
- **Standard fix — break circular wait via resource ordering** (directly file 03's general prevention strategy, applied here): number the forks; every philosopher picks up the **lower-numbered fork first**, regardless of which side it's physically on. This breaks the symmetry that created the cycle.
```cpp
void philosopher(int id, std::mutex forks[], int n) {
    int left = id, right = (id + 1) % n;
    int first = std::min(left, right);
    int second = std::max(left, right);
    while (true) {
        std::lock_guard<std::mutex> l1(forks[first]);
        std::lock_guard<std::mutex> l2(forks[second]);
        // eat
    }
}
```
- **Alternative fix worth naming:** limit at most `N-1` philosophers to sit down at once (an "arbitrator" semaphore initialized to `N-1`) — guarantees at least one philosopher can always get both forks. Both fixes are valid; the fork-ordering version is generally considered the cleaner answer and directly demonstrates you understand *why* it works (breaks circular wait), not just that it does.

## Thread Pool (a very common "build this from scratch" prompt)

```cpp
#include <vector>
#include <thread>
#include <queue>
#include <functional>
#include <mutex>
#include <condition_variable>
#include <atomic>

class ThreadPool {
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cv;
    std::atomic<bool> stop{false};
public:
    ThreadPool(size_t n) {
        for (size_t i = 0; i < n; i++) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(mtx);
                        cv.wait(lock, [this] { return stop || !tasks.empty(); });
                        if (stop && tasks.empty()) return;
                        task = std::move(tasks.front());
                        tasks.pop();
                    }
                    task();   // run outside the lock — critical, don't hold the lock during task execution
                }
            });
        }
    }

    void enqueue(std::function<void()> task) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            tasks.push(std::move(task));
        }
        cv.notify_one();
    }

    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(mtx);
            stop = true;
        }
        cv.notify_all();
        for (auto& t : workers) t.join();
    }
};
```
- **The detail that separates a working answer from a good one:** running `task()` *outside* the lock. Holding the queue's mutex while a task executes serializes all task execution behind that single lock, defeating the purpose of having multiple workers — worth stating this out loud as you write it, since it's exactly the kind of subtlety interviewers watch for.
- **Shutdown correctness:** setting `stop` under the lock, notifying `notify_all` (not `notify_one` — every worker needs to wake up and re-check), and each worker's wait-predicate checking `stop && tasks.empty()` (not just `stop`) so pending tasks still get drained before exit rather than being dropped.

## Practice signal

If given a bare "implement X" prompt without more detail, state your assumptions out loud first (bounded or unbounded? single or multiple producers/consumers? does shutdown need to drain pending work or can it discard it?) — the classic problems above are templates, and interviewers often want to see you adapt one to stated constraints rather than reciting a memorized solution verbatim.

---

## Related Modules

- [OS 02 — Synchronization/Concurrency](os_02_synchronization_and_concurrency.md)
- [OS 03 — Deadlocks](os_03_deadlocks.md)
- [OS 09 — Debugging Tools (GDB for a hung thread pool)](os_09_debugging_and_profiling_tools.md)
