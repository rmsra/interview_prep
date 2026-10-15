# Core Language Features

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** [Index](python_00_README_start_here.md) · **Core Language Features** · [OOP in Python](python_02_oop_in_python.md) · [Decorators/Generators](python_03_functional_features_and_decorators.md) · [Memory Model/GIL](python_04_memory_model_and_gil.md) · [Concurrency in Python](python_05_concurrency_in_python.md) · [Typing/Modern Python](python_06_typing_and_modern_python.md) · [Interview Trivia](python_07_interview_common_questions.md) · [Testing/Tooling](python_08_testing_and_tooling.md)

---

## Everything Is an Object, and Names Are Just Bindings

In Python, a variable is not a box holding a value — it's a **name bound to an object**. `x = 5` doesn't put `5` "into" `x`; it makes `x` point at an `int` object `5` that exists independently. `x = 6` doesn't mutate that object — it rebinds `x` to point at a *different* `int` object. This single mental model explains most of the mutability/aliasing behavior below.

## Mutable vs. Immutable Built-in Types — the distinction that causes real bugs

| Immutable | Mutable |
|---|---|
| `int`, `float`, `bool`, `str`, `tuple`, `frozenset` | `list`, `dict`, `set`, most user-defined classes by default |

```python
a = [1, 2, 3]
b = a          # b is another NAME for the SAME list object, not a copy
b.append(4)
print(a)       # [1, 2, 3, 4] — a changed too, because a and b are the same object
```

## The Mutable Default Argument Trap — the single most common Python gotcha

```python
def append_item(item, target=[]):   # DANGER: default evaluated ONCE, at function definition time
    target.append(item)
    return target

append_item(1)   # [1]
append_item(2)   # [1, 2] -- NOT [2]! Same list object reused across every call.
```
**Why this happens, precisely:** default argument values are evaluated **once**, when the `def` statement executes (at import/definition time) — not once per call. A mutable default is therefore the *same object*, shared and accumulated across every call that doesn't explicitly pass that argument. **The fix, idiomatic and worth having memorized:**
```python
def append_item(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target
```
This is a very common live-coding "spot the bug" prompt — recognizing it instantly, and being able to explain *why* (not just "don't do that"), is a strong signal.

## Argument Passing — "pass-by-object-reference," precisely

Python is neither strictly pass-by-value nor pass-by-reference — the precise term is **pass-by-object-reference** (sometimes "pass-by-assignment"): the function parameter is a new name bound to the *same object* the caller's argument referred to.
- **Reassigning the parameter inside the function** doesn't affect the caller's binding (you're just rebinding the local name to a different object).
- **Mutating the object the parameter refers to** (if it's mutable) *is* visible to the caller, since both names refer to the same object.
```python
def f(x):
    x = x + [4]   # rebinds local x to a NEW list — caller's list untouched

def g(x):
    x.append(4)   # mutates the SAME list object — caller sees the change

lst = [1, 2, 3]
f(lst); print(lst)   # [1, 2, 3] — unchanged
g(lst); print(lst)   # [1, 2, 3, 4] — changed
```

## `is` vs. `==`

- `==` checks **value equality** (calls `__eq__`) — "do these represent the same value."
- `is` checks **identity** — "are these literally the same object in memory" (equivalent to comparing `id(a) == id(b)`).
```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b   # True  — same value
a is b   # False — different objects
a is a   # True
```
**The `is None` convention:** always use `is None` / `is not None`, never `== None` — `None` is a singleton (exactly one `None` object exists in the entire running process), so identity comparison is both correct and (very marginally) faster than a value-equality call, and it's the unambiguous idiomatic convention regardless.

## List/Dict/Set Comprehensions

```python
squares = [x**2 for x in range(10)]                     # list comprehension
evens = {x for x in range(10) if x % 2 == 0}             # set comprehension
lookup = {x: x**2 for x in range(10)}                    # dict comprehension
gen = (x**2 for x in range(10))                          # generator expression — lazy, see file 03
```
- Comprehensions are generally faster than the equivalent explicit `for` loop with `.append()` calls, because the loop is executed in C internally rather than via repeated bytecode-level method-call overhead — worth knowing as the *why*, not just "comprehensions are more Pythonic."
- **Readability limit, worth stating if asked to review one:** nested comprehensions beyond 2 levels, or comprehensions with complex conditional logic, are a real readability regression — a plain `for` loop is the better choice past a certain complexity, and knowing where that line is (not just "comprehensions are always better") is a good signal of judgment.

## Unpacking, `*args`, `**kwargs`

```python
a, *rest = [1, 2, 3, 4]     # a=1, rest=[2,3,4] — star-unpacking, not just tuple assignment
first, *middle, last = range(5)   # first=0, middle=[1,2,3], last=4

def f(*args, **kwargs):     # args: tuple of positional args; kwargs: dict of keyword args
    print(args, kwargs)
f(1, 2, x=3, y=4)            # (1, 2) {'x': 3, 'y': 4}

def g(a, b, *, c):           # * forces c to be KEYWORD-ONLY — g(1, 2, 3) is a TypeError
    ...
```
The bare `*` in a function signature (not followed by a name) is a real, specific syntax worth knowing: it doesn't collect anything itself, it just marks everything after it as keyword-only — a common way to force callers to be explicit about less-obvious arguments rather than relying on positional order.

## The Walrus Operator `:=` (3.8+)

```python
if (n := len(data)) > 10:
    print(f"data has {n} items")   # n is usable both inside the if AND after it
```
Assignment as an *expression*, not just a statement — lets you assign and test in one line, avoiding a redundant call/lookup. Genuinely useful in `while` loops reading from a stream (`while (chunk := f.read(8192)):`) — worth knowing the syntax exists and one clean use case, moderate interview priority.

## Practice signal

The mutable-default-argument trap is close to a guaranteed question in some form — either directly ("what's wrong with this function") or as a bug planted in a larger code-review prompt. `is` vs. `==`, and precisely explaining pass-by-object-reference with a mutate-vs-reassign example, are the other two most likely direct-recall questions in this file.
