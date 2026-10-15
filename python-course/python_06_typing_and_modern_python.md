# Typing & Modern Python

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** [Index](python_00_README_start_here.md) · [Core Language Features](python_01_core_language_features.md) · [OOP in Python](python_02_oop_in_python.md) · [Decorators/Generators](python_03_functional_features_and_decorators.md) · [Memory Model/GIL](python_04_memory_model_and_gil.md) · [Concurrency in Python](python_05_concurrency_in_python.md) · **Typing/Modern Python** · [Interview Trivia](python_07_interview_common_questions.md) · [Testing/Tooling](python_08_testing_and_tooling.md)

---

## Type Hints — what they are, and precisely what they aren't

```python
def add(a: int, b: int) -> int:
    return a + b

x: list[int] = []
```
**Type hints are not enforced at runtime by the interpreter at all.** `add("a", "b")` runs without error and returns `"ab"` — Python's runtime remains fully dynamically typed regardless of hints. Hints exist purely for: **static analysis tools** (`mypy`, `pyright`) that check consistency *before* running anything, IDE autocomplete/navigation, and documentation for humans reading the signature. This precise distinction — "hints are checked by external tools, not the language itself" — is worth stating exactly this way if asked, since assuming Python enforces them at runtime is a common misconception.

**Common typing constructs worth recognizing:**
```python
from typing import Optional, Union

def find(key: str) -> Optional[str]: ...          # Optional[str] == Union[str, None]
def process(x: Union[int, str]) -> None: ...       # accepts either type
def process(x: int | str) -> None: ...             # same as Union, modern 3.10+ syntax
```

## `dataclasses` — reducing class-definition boilerplate

```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: int
    y: int
    tags: list[str] = field(default_factory=list)   # mutable default -- SAFE here, unlike a plain function default
```
`@dataclass` auto-generates `__init__`, `__repr__`, and `__eq__` (comparing all fields) from the type-annotated class body — eliminating the boilerplate of hand-writing all three for a class that's mostly just structured data. **`field(default_factory=list)` is the dataclass-specific fix for the mutable-default trap from file 01** — instead of a shared default object evaluated once, `default_factory` is a **callable** invoked fresh for every new instance, giving each instance its own independent list. Knowing this exists (and why it's needed, tying directly back to the mutable-default gotcha) is a good, specific signal.

## `enum`

```python
from enum import Enum, auto

class Status(Enum):
    PENDING = auto()
    ACTIVE = auto()
    DONE = auto()

s = Status.ACTIVE
s.name    # "ACTIVE"
s.value   # 2
```
A type-safe, named-constant alternative to using bare strings or integers as status/mode flags — comparisons (`s == Status.ACTIVE`) are checked against a real, closed set of members rather than an arbitrary string that could be misspelled with no error. `auto()` assigns increasing integer values automatically when the specific numeric value doesn't matter, only distinctness does.

## `pathlib` vs. `os.path`

```python
from pathlib import Path
p = Path("data") / "logs" / "output.txt"    # / operator overloaded for path joining
p.exists(), p.suffix, p.stem, p.parent
p.read_text()

# vs. the older os.path style:
import os
p = os.path.join("data", "logs", "output.txt")
os.path.exists(p)
```
`pathlib` represents paths as **objects** with methods, rather than plain strings passed to free functions — genuinely more ergonomic (the overloaded `/` for joining is a real, well-liked piece of API design) and is the modern idiomatic default for new code; `os.path` remains extremely common in existing/older codebases and is still fully supported, not deprecated. Worth knowing both, since you'll read plenty of code using either.

## `f-strings` (3.6+) — the modern default for string formatting

```python
name, count = "widgets", 5
f"{name}: {count} items"           # "widgets: 5 items"
f"{count:>5}"                      # right-aligned in a 5-char field
f"{value:.2f}"                     # 2 decimal places
f"{count=}"                        # 3.8+: "count=5" -- prints both the expression text and its value, great for quick debugging
```
Faster than both `%`-formatting and `.format()` (evaluated more directly by the interpreter rather than via a separate method-call/parsing step) and the current idiomatic default. The `{expr=}` debug specifier (3.8+) is a small, genuinely handy detail worth knowing for quick print-debugging.

## Structural Pattern Matching (3.10+) — worth knowing exists, lower priority

```python
match command.split():
    case ["go", direction]:
        move(direction)
    case ["look"]:
        describe_room()
    case _:
        print("unknown command")
```
Python's `switch`-equivalent, but structurally match-capable (destructuring, not just value equality) — genuinely newer than most production codebases have fully adopted, so lower interview priority than everything else in this file; worth recognizing the syntax if you see it, not worth deep fluency unless directly asked.

## Practice signal

"Are type hints enforced?" is a fair, common, precise-answer-rewarded question — "no, purely for tooling/documentation" plus naming `mypy` is the complete correct answer. If asked to refactor a class with a lot of `__init__` boilerplate live, reaching for `@dataclass` unprompted (when appropriate — plain structured data, not something needing custom validation logic in `__init__`) is a good, concrete signal of current Python fluency.
