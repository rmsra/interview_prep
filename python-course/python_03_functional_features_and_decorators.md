# Functional Features, Decorators, Generators, Context Managers

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** [Index](python_00_README_start_here.md) · [Core Language Features](python_01_core_language_features.md) · [OOP in Python](python_02_oop_in_python.md) · **Decorators/Generators** · [Memory Model/GIL](python_04_memory_model_and_gil.md) · [Concurrency in Python](python_05_concurrency_in_python.md) · [Typing/Modern Python](python_06_typing_and_modern_python.md) · [Interview Trivia](python_07_interview_common_questions.md) · [Testing/Tooling](python_08_testing_and_tooling.md)

---

## Closures

```python
def make_counter():
    count = 0
    def increment():
        nonlocal count      # without this, `count += 1` would raise UnboundLocalError
        count += 1
        return count
    return increment

c = make_counter()
c(), c(), c()   # 1, 2, 3 — each call to make_counter() creates an INDEPENDENT closure
```
A closure is an inner function that **remembers the enclosing scope's variables**, even after the outer function has returned. **`nonlocal` is required to *rebind* an enclosing-scope variable from inside the closure** (without it, `count += 1` is treated as creating a new *local* variable named `count`, and Python raises `UnboundLocalError` because that local `count` is read before it's ever assigned) — reading an enclosing variable needs no special keyword, only rebinding it does. This asymmetry (read freely, must declare to write) is a genuinely common point of confusion worth having precise.

## Decorators — the mechanism, precisely

```python
def timer(func):
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.perf_counter() - start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    ...

# @timer above is EXACTLY equivalent to:
slow_function = timer(slow_function)
```
A decorator is just a function that **takes a function and returns a (usually different) function** — `@decorator` above a `def` is pure syntax sugar for reassigning the name to the decorator's return value. Understanding this equivalence is what makes decorators-with-arguments and stacked decorators comprehensible rather than magic.

**Decorators that take their own arguments** need one more layer of nesting:
```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(times=3)
def greet(): print("hi")
```
`repeat(times=3)` is called first, immediately, and *its return value* (`decorator`) is what actually gets applied to `greet` — three levels: the argument-taking outer function, the actual decorator, the wrapper.

**`functools.wraps` — the detail people forget, worth naming:**
```python
from functools import wraps
def timer(func):
    @wraps(func)          # without this, wrapper.__name__ == "wrapper", not the original function's name
    def wrapper(*args, **kwargs):
        ...
    return wrapper
```
Without `@wraps(func)`, the decorated function's `__name__`, `__doc__`, and other metadata get silently replaced by the wrapper's — breaking introspection, debuggers, and anything relying on `func.__name__` (including error messages and some test frameworks). A small, easy-to-forget detail that's a genuinely good thing to include unprompted if asked to write a decorator live.

## Generators & `yield`

```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i
        i += 1

for x in count_up_to(5):   # 1, 2, 3, 4, 5 -- computed LAZILY, one at a time
    print(x)
```
- A function containing `yield` becomes a **generator function** — calling it doesn't run the body at all; it returns a generator object. Execution happens incrementally: each `next()` call (which a `for` loop does implicitly) runs the body until the next `yield`, pauses there with **all local state preserved**, and resumes exactly there on the next `next()` call.
- **Why this matters, concretely — memory:** `[x**2 for x in range(10_000_000)]` builds the entire 10-million-element list in memory immediately; `(x**2 for x in range(10_000_000))` (a generator expression) produces values one at a time, using O(1) memory regardless of how many values are eventually consumed. This is the real, practical reason to reach for a generator over a list comprehension — when you're going to iterate once and don't need random access or the full collection in memory at once.
- **The iterator protocol underneath:** a generator automatically implements `__iter__` and `__next__` — this is *the* mechanism behind Python's `for` loop, which is itself just repeatedly calling `next()` until `StopIteration` is raised. Any custom class implementing both methods manually is a hand-rolled iterator; `yield` is the ergonomic shortcut that writes this boilerplate for you.

## Context Managers — the `with` statement

```python
class ManagedFile:
    def __init__(self, name):
        self.name = name
    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file
    def __exit__(self, exc_type, exc_value, traceback):
        self.file.close()
        return False   # False (or None) = don't suppress the exception; True = swallow it

with ManagedFile('log.txt') as f:
    f.write('hello')
# __exit__ runs here UNCONDITIONALLY -- even if the with block raised
```
- **`__exit__`'s return value controls exception suppression** — a real, specific, commonly-missed detail: returning `True` swallows any exception that occurred inside the `with` block (the exception simply doesn't propagate further); returning `False`/`None` lets it propagate normally after cleanup still runs. Worth stating this precisely rather than "it just cleans up" — it's the direct Python analog to RAII's guaranteed-destructor-on-unwind from the C++ course, applied at the statement level instead of the object-lifetime level.
- **`contextlib.contextmanager`** — the decorator-based shortcut, avoiding the full class:
```python
from contextlib import contextmanager

@contextmanager
def managed_file(name):
    f = open(name, 'w')
    try:
        yield f              # code inside the `with` block runs here, at the yield point
    finally:
        f.close()            # guaranteed to run on the way out, exception or not — same role as __exit__
```
The single `yield` splits the function into "setup" (before `yield`) and "teardown" (after, in `finally`) — the `try`/`finally` is what guarantees teardown runs even if the `with` block raises, functionally identical to a hand-written `__exit__`, in far less code. Knowing this shortcut exists (and using it) is itself a mild positive interview signal — it reads as idiomatic, not just correct.

## Practice signal

"Write a decorator that [times/retries/caches] a function" is a very common live-coding prompt — know the three-level nesting pattern for a parameterized decorator cold, and remember `functools.wraps` unprompted. "Implement a context manager for [a resource]" is equally common — the `@contextmanager` generator-based version is usually faster to write correctly live than the full `__enter__`/`__exit__` class.

---

## Related Modules

- [Python 08 — Testing/Tooling (yield-based fixtures use the same shape)](python_08_testing_and_tooling.md)
