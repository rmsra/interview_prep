# Python Interview Common Questions — the Trivia Cluster

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** [Index](python_00_README_start_here.md) · [Core Language Features](python_01_core_language_features.md) · [OOP in Python](python_02_oop_in_python.md) · [Decorators/Generators](python_03_functional_features_and_decorators.md) · [Memory Model/GIL](python_04_memory_model_and_gil.md) · [Concurrency in Python](python_05_concurrency_in_python.md) · [Typing/Modern Python](python_06_typing_and_modern_python.md) · **Interview Trivia** · [Testing/Tooling](python_08_testing_and_tooling.md)

---

Same shape as the C++ course's trivia file: fast, precise, standalone-answerable questions used for a quick fluency read. Short answers on purpose.

**What's the difference between a list and a tuple?**
Lists are mutable, tuples are immutable. Practical consequences: tuples are hashable (when their contents are) and can be used as `dict` keys/`set` members; lists cannot. Tuples are also conventionally used for fixed-structure heterogeneous data (a coordinate pair, a database row), lists for homogeneous, variable-length collections — a convention, not an enforced rule.

**Why is a `list` a bad default for a function argument, and what should you use instead?**
Covered fully in file 01 — mutable defaults are evaluated once at definition time and shared/accumulated across calls. Use `None` as the default, initialize the mutable object inside the function body.

**How does Python's `sorted()`/`.sort()` stability work?**
Both are **stable** — equal elements retain their relative order — using Timsort (a hybrid merge/insertion sort designed to exploit existing runs of ordered data in real-world input). Directly parallels the DSA course's `std::stable_sort` discussion; worth naming that Python's default sort is *always* stable, unlike C++'s `std::sort`, which is explicitly not.

**What's the difference between `__init__` and `__new__`?**
`__new__` actually **creates** the instance (called first, returns the new object) — rarely overridden except for immutable types (subclassing `str`/`tuple`/`int`) or metaclass/singleton patterns, since you need to intercept object *creation* itself, before `__init__` even has an instance to operate on. `__init__` **initializes** an already-created instance (receives `self`, sets attributes, returns `None`) — what you override for essentially all normal class definitions. Most Python developers never need to touch `__new__` at all; knowing the distinction exists (and the rare cases that need it) is the useful depth here.

**What does `if __name__ == "__main__":` actually do?**
Every module has a `__name__` variable — set to `"__main__"` if the file is being run directly (`python script.py`), or to the module's actual name if it's being **imported** by another file. This guard lets a file define reusable functions/classes *and* have script-only behavior (like a demo or CLI entry point), without that script-only code running on import — a real, common pattern, not just boilerplate.

**What's a list comprehension vs. a generator expression, and when does the difference matter?**
Syntax: `[x for x in y]` (list, eager, builds the whole thing in memory) vs. `(x for x in y)` (generator, lazy, one value at a time — file 03). Matters concretely when: the sequence is large/unbounded (memory), you only need to iterate once and don't need random access or `len()`, or you're feeding a function (like `sum()` or `any()`) that only needs one pass and can short-circuit — a generator lets `any()` stop at the first `True` without ever materializing the rest.

**How do you handle exceptions in Python — `try`/`except`/`else`/`finally`, precisely?**
```python
try:
    risky()
except ValueError as e:
    handle(e)
else:
    only_runs_if_no_exception()   # a real, often-forgotten clause
finally:
    always_runs()                 # cleanup — runs whether or not an exception occurred, even if the except block re-raises
```
The `else` clause specifically (not just `try`/`except`/`finally`) is worth knowing — it separates "code that might raise" (in `try`) from "code that should only run if nothing went wrong" (in `else`), which is more precise than just appending that code to the end of the `try` block (where it would ambiguously be inside the risky/monitored region).

**What's the difference between `@staticmethod` and a plain module-level function?**
Almost none, functionally — `@staticmethod` is purely for **namespacing/organization**, grouping a function under a class it's conceptually related to, when it doesn't need `self` or `cls`. If it doesn't need to live inside the class's namespace, a plain module-level function is equally valid and arguably simpler — worth saying this rather than treating `@staticmethod` as doing something functionally special.

**What is monkey-patching, and is it a good idea?**
Modifying/replacing a class or module's attributes/methods at runtime, from outside its original definition (`SomeClass.method = new_implementation`). Genuinely useful in tests (patching out a real network call for a fake one — `unittest.mock.patch`, file 08) and occasionally for working around a bug in a third-party library you can't modify directly. Generally a code smell in application logic itself — it makes behavior hard to trace (a method's actual behavior depends on import order and what's patched where), so the honest answer distinguishes "great, standard tool for testing" from "risky if used in production logic."

**Why might `except:` (bare, no exception type) be considered bad practice?**
Catches **everything**, including `KeyboardInterrupt` and `SystemExit` (both inherit from `BaseException`, not `Exception`) — swallowing a user's Ctrl+C or a legitimate program-exit signal is a real, specific problem, not just "too broad" in the abstract. `except Exception:` is the more correct "catch basically anything expected to be an application-level error" — it excludes those two, letting them propagate as intended.

## Practice signal

Same guidance as the C++ trivia file: each of these should be answerable in 15-30 seconds. If one takes longer to explain, that's the signal to revisit the fuller treatment elsewhere in this course and compress deliberately, rather than trying to think through the compression live in the room.

---

## Related Modules

- [C++ 09 — Interview Trivia (same format, other language)](../cpp-course/cpp_09_interview_common_questions.md)
