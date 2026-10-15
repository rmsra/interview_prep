# Memory Model & the GIL

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** [Index](python_00_README_start_here.md) · [Core Language Features](python_01_core_language_features.md) · [OOP in Python](python_02_oop_in_python.md) · [Decorators/Generators](python_03_functional_features_and_decorators.md) · **Memory Model/GIL** · [Concurrency in Python](python_05_concurrency_in_python.md) · [Typing/Modern Python](python_06_typing_and_modern_python.md) · [Interview Trivia](python_07_interview_common_questions.md) · [Testing/Tooling](python_08_testing_and_tooling.md)

---

## Reference Counting — Python's primary memory management mechanism

Every Python object has a **reference count** — the number of names/containers currently referring to it. `sys.getrefcount(obj)` inspects it directly. When the count hits zero, **CPython frees the object immediately** — deterministically, not on some later garbage-collection sweep. This is genuinely different from Java/C#'s purely tracing-GC model, and worth stating precisely: `del x` doesn't necessarily free the object (only decrements the refcount and removes that one binding) — the object is only actually freed once *every* reference to it is gone.

```python
a = SomeObject()   # refcount = 1
b = a              # refcount = 2
del a              # refcount = 1 -- object still alive, b still references it
del b              # refcount = 0 -- object freed NOW, deterministically
```

## The Cycle-Detecting Garbage Collector — the part refcounting alone can't handle

```python
a = {}
b = {}
a['b'] = b
b['a'] = a   # a and b reference each other -- refcount of each is >= 1 FOREVER via the cycle
del a, b     # neither refcount ever reaches 0 through this cycle alone
```
Pure reference counting can't collect a **reference cycle** — two or more objects that reference each other keep each other's count above zero forever, even when nothing outside the cycle references either of them. CPython's `gc` module runs periodically, specifically looking for and collecting these unreachable cycles (using a mark-and-sweep-style algorithm, restricted to container types capable of participating in cycles — dicts, lists, class instances, etc.). This is directly the Python-level version of the `shared_ptr` cycle problem from the C++ course's smart-pointers file — same underlying issue (mutual ownership prevents refcount-based collection), different language's fix (Python's is automatic via the cycle collector; C++'s is manual, via `weak_ptr`).

## The GIL — Global Interpreter Lock

**What it actually is:** a single mutex inside the CPython interpreter (not a language requirement — this is a CPython implementation detail; PyPy and Jython have different models) that ensures **only one thread executes Python bytecode at a time**, even on a multi-core machine, even with multiple `threading.Thread` objects created.

**Why it exists, precisely — not just "Python is slow":** CPython's reference counting (above) is not thread-safe on its own — two threads concurrently incrementing/decrementing the same object's refcount is a genuine data race that could corrupt memory. The GIL is the (blunt, but simple and historically effective) solution: serialize all bytecode execution so refcount updates can never race. This is the real, specific reason the GIL exists, and it's a much stronger answer than "Python just has a GIL."

**What the GIL does and doesn't block, precisely:**
- **Blocks:** genuine CPU-bound parallelism across threads — two threads both doing pure Python computation cannot run simultaneously on separate cores, no matter how many cores are available.
- **Doesn't block:** I/O-bound concurrency — the GIL is **released** during blocking I/O calls (file reads, network calls, `time.sleep`) specifically so other threads can run while one thread is waiting on something external. This is why `threading` genuinely helps for I/O-bound work despite the GIL, and doesn't help (and can even hurt, from lock-contention/context-switch overhead) for CPU-bound work.
- **Doesn't block:** C extensions that explicitly release the GIL during long-running native computation (NumPy's array operations do this) — this is precisely why NumPy-heavy code can achieve real parallelism across threads even in vanilla CPython, worth naming given your own NumPy/Pandas background.

Full treatment of what to actually *do* about this (threading vs. multiprocessing vs. asyncio) is file 05 — this file is specifically the "why," which is what's usually actually being tested when someone asks "explain the GIL."

## Shallow Copy vs. Deep Copy

```python
import copy
original = [[1, 2], [3, 4]]

shallow = copy.copy(original)      # or original[:], or list(original)
shallow[0].append(99)
print(original)                     # [[1, 2, 99], [3, 4]] -- inner list is SHARED, mutation visible

deep = copy.deepcopy(original)
deep[0].append(100)
print(original)                     # unaffected -- deepcopy recursively copies every nested object
```
A shallow copy creates a new **outer** container but populates it with the *same* inner object references — mutating a nested mutable object through the copy is visible through the original too. `copy.deepcopy` recursively copies every level. **The gotcha to know:** for a container of only immutable elements (`[1, 2, 3]`), shallow and deep copies behave identically in practice, which can mask this distinction until you're working with nested mutable structures (lists of lists, objects containing lists) — exactly the case above.

## Practice signal

"Explain the GIL" is close to a guaranteed question given Python is an explicit qualification here — the strong answer leads with *why* (refcount thread-safety), not just *what* (one thread at a time), and immediately follows with the I/O-bound vs. CPU-bound distinction, since that's what makes the answer actionable rather than trivia. "What happens if two objects reference each other — do they leak?" → no, the cycle-detecting GC handles it (contrast with the C++ course's `shared_ptr` cycle, which *does* leak without manual intervention) — a good, specific cross-language comparison if both courses come up in the same conversation.

---

## Related Modules

- [C++ 03 — RAII/Smart Pointers (contrast: manual weak_ptr vs. automatic cycle GC)](../cpp-course/cpp_03_raii_and_smart_pointers.md)
- [Python 05 — Concurrency in Python](python_05_concurrency_in_python.md)
