# OOP in Python

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** [Index](python_00_README_start_here.md) · [Core Language Features](python_01_core_language_features.md) · **OOP in Python** · [Decorators/Generators](python_03_functional_features_and_decorators.md) · [Memory Model/GIL](python_04_memory_model_and_gil.md) · [Concurrency in Python](python_05_concurrency_in_python.md) · [Typing/Modern Python](python_06_typing_and_modern_python.md) · [Interview Trivia](python_07_interview_common_questions.md) · [Testing/Tooling](python_08_testing_and_tooling.md)

---

## Dunder (Magic) Methods — the mechanism behind Python's syntax sugar

Python's built-in operators and functions (`+`, `len()`, `str()`, `for`, `[]`) work on custom objects by looking for specifically-named methods — "dunder" (double-underscore) methods. This is *how* operator overloading and protocol conformance work in Python — there's no separate `operator+` syntax like C++; you just implement `__add__`.

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __repr__(self):                       # unambiguous, dev-facing: repr(obj), or fallback for print()
        return f"Vector({self.x}, {self.y})"
    def __str__(self):                        # readable, user-facing: str(obj), print(obj) if defined
        return f"({self.x}, {self.y})"
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    def __len__(self):
        return 2
```
- **`__repr__` vs. `__str__`, precisely:** `__repr__` should be unambiguous — ideally something that could recreate the object (`Vector(1, 2)`), meant for developers/debugging (this is what shows up in a REPL or inside a `list` of these objects when printed). `__str__` is for end-user-facing display, and can be looser/prettier. **If only `__repr__` is defined, `str()`/`print()` fall back to it** — this is why defining `__repr__` first, even if `__str__` never gets added, is the higher-value default.
- **Defining `__eq__` without `__hash__` makes the object unhashable** (Python sets `__hash__` to `None` automatically when you define `__eq__` without also defining `__hash__`) — meaning it can no longer be used as a `dict` key or `set` member. This is a real, common surprise: if you need both value equality *and* hashability, you must define both explicitly (and they must be consistent — equal objects must have equal hashes).

## Class vs. Instance Attributes — a genuine, common bug source

```python
class Dog:
    tricks = []                 # CLASS attribute — shared across ALL instances
    def __init__(self, name):
        self.name = name         # INSTANCE attribute — unique per object

d1 = Dog("Rex")
d2 = Dog("Fido")
d1.tricks.append("sit")
print(d2.tricks)   # ['sit'] -- same list, shared! Same root cause as the mutable-default-argument trap.
```
A **mutable** class attribute is shared by every instance unless an instance explicitly shadows it with its own instance attribute of the same name. This is the class-level version of the mutable-default-argument bug from file 01 — same underlying cause (one object, shared where a fresh one per instance was intended), different syntax. The fix is the same shape: initialize mutable per-instance state inside `__init__`, not as a class-level default.

## `@classmethod`, `@staticmethod`, `@property`

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @classmethod
    def from_diameter(cls, diameter):   # alternate constructor — receives the CLASS, not an instance
        return cls(diameter / 2)

    @staticmethod
    def is_valid_radius(r):             # no implicit first argument at all — just namespaced under the class
        return r > 0

    @property
    def radius(self):                   # accessed like an attribute: circle.radius, no parens
        return self._radius

    @radius.setter
    def radius(self, value):
        if value <= 0:
            raise ValueError("radius must be positive")
        self._radius = value
```
- **`classmethod`** receives `cls` (the class itself) instead of `self` — the standard pattern for alternate constructors (`from_diameter`, `from_json`, etc.), since it can return `cls(...)` and works correctly for subclasses too (calling `cls(...)` on a subclass constructs the subclass, not hardcoded to the base class).
- **`staticmethod`** receives neither `self` nor `cls` — just a regular function namespaced inside the class for organizational purposes; genuinely just sugar, no special binding behavior at all.
- **`@property`** lets you expose a method as if it were a plain attribute (no `()` at the call site) — the real value is being able to **add validation or computed-on-access logic later without breaking every caller's existing `obj.radius` syntax** — you can start with a plain public attribute, and convert it to a property later if you need validation, with zero changes required at any call site. Worth stating this "you can add it later transparently" property specifically if asked why Python favors properties over Java/C++-style explicit getters/setters from the start.

## Duck Typing & the "Protocol" Mindset

**"If it walks like a duck and quacks like a duck, it's a duck"** — Python generally doesn't check an object's declared type before using it; it just tries to call the method/use the operator, and lets it fail at runtime (`AttributeError`/`TypeError`) if the object doesn't actually support it. This is the philosophical opposite of C++'s compile-time-checked interfaces (abstract base classes with pure virtual functions) — Python *has* `abc.ABC`/`abstractmethod` for when you genuinely want to enforce an interface, but idiomatic Python code more often relies on duck typing and only reaches for formal abstract base classes when the enforcement is actually worth the rigidity.

## MRO — Method Resolution Order (multiple inheritance, Python's version of the diamond problem)

```python
class A:
    def greet(self): return "A"
class B(A):
    def greet(self): return "B"
class C(A):
    def greet(self): return "C"
class D(B, C):
    pass

D().greet()          # "B" — MRO is D, B, C, A, object
print(D.__mro__)     # inspect the actual resolution order directly
```
Python resolves multiple inheritance using the **C3 linearization algorithm** — deterministic, and (unlike C++'s ambiguity error for the equivalent diamond) always produces a single, well-defined order: roughly, "depth-first, but a class never appears before its own subclasses, and the explicit left-to-right base-class order in the class definition is preserved." You don't need to derive C3 by hand in an interview — knowing it's called C3 linearization, that `ClassName.__mro__` lets you inspect it directly, and that it's *always* well-defined (no ambiguity error the way C++ has) is the useful, precise level of depth.

## Practice signal

"Implement `__eq__` and `__hash__` for this class" or "why doesn't `str()` show what I expect" (missing `__repr__`/`__str__`) are realistic live-coding prompts. The class-vs-instance-attribute mutable-default bug is a strong "spot the bug" candidate, parallel to file 01's function-default version — worth recognizing both as the same root cause if either comes up.

---

## Related Modules

- [C++ 02 — OOP/Class Design (contrast: virtual inheritance vs. MRO/C3)](../cpp-course/cpp_02_oop_and_class_design.md)
