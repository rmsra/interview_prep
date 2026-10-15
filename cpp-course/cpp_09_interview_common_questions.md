# C++ Interview Common Questions — the Trivia Cluster

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · [OOP/Class Design](cpp_02_oop_and_class_design.md) · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · [Templates](cpp_05_templates_and_generic_programming.md) · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · [Exceptions](cpp_08_exceptions_and_error_handling.md) · **Interview Trivia**

---

These come up as fast, standalone calibration questions — often early in an interview, before any coding starts, to get a quick read on your fluency. Each answer here is deliberately short; the goal is precision under time pressure, not exhaustive explanation. Most of these are covered in more depth elsewhere in this course — this file is the fast-recall version, with pointers to where to go deeper.

**Why is a virtual destructor necessary in a base class?**
Deleting a derived object through a base pointer only runs the base destructor if it's non-virtual — the derived destructor (and any resources it owns) never runs. Any class meant to be used polymorphically through a base pointer needs a virtual destructor. → `cpp_02`

**What's the difference between static and dynamic binding?**
Static (compile-time) binding: which function is called is decided by the *static* type at compile time — the default for non-virtual functions. Dynamic (runtime) binding: decided by the *actual* object type at runtime, via the vtable — what `virtual` enables. → `cpp_02`

**Pass by value, reference, or pointer — how do you choose?**
By value: when you want an independent copy, or the type is small/trivial (an `int`, a `bool`) where copying is cheaper than indirection. By `const&`: the default for anything non-trivial you're only reading — avoids a copy, communicates "won't modify." By `&` (non-const): when the function needs to modify the caller's object. By pointer: when the argument might legitimately be absent (`nullptr` is a valid "no object" signal a reference can't express) or when you need to reseat/reassign what's pointed to.

**What is the Static Initialization Order Fiasco?**
The relative initialization order of **static/global objects defined in different translation units (`.cpp` files) is unspecified** — if one such object's constructor depends on another already being initialized, and they happen to initialize in the wrong order, you get undefined behavior with no compiler warning. The standard fix: wrap the object in a function returning a `static` local (a "Meyers singleton") — function-local statics are guaranteed to initialize on **first use**, sidestepping cross-translation-unit ordering entirely.

**What does `explicit` do, and why does it matter?**
Prevents a single-argument constructor from being used for **implicit conversions**. Without it, `void f(Widget w); f(5);` silently converts `5` to a `Widget` via a single-argument `Widget(int)` constructor, even though that's rarely what was intended. `explicit` forces `Widget(5)` to be written out at the call site — a real, common source of surprising bugs in constructors that weren't marked `explicit` when they should have been. Default assumption for any single-argument constructor: mark it `explicit` unless you have a specific reason for implicit conversion to be desirable.

**What is the ODR (One Definition Rule)?**
A non-inline function/object must have **exactly one definition** across the entire program (across all translation units combined) — declarations can appear many times (e.g., in a header included everywhere), but the actual definition can only exist once, or you get a linker error (multiply-defined symbol) or, worse, silent undefined behavior if the violation isn't caught. This is *why* function/class definitions typically live in `.cpp` files while headers hold only declarations — and *why* `inline` exists as a keyword for the exception (a function defined identically in a header, included in multiple translation units, is permitted precisely because `inline` tells the linker to treat all those identical definitions as one).

**Undefined vs. unspecified vs. implementation-defined behavior — what's the difference?**
**Undefined:** the standard makes zero guarantees at all — anything can happen, including working "correctly" by pure accident (signed integer overflow, use-after-free, out-of-bounds access). **Unspecified:** the standard allows several valid behaviors and doesn't require documenting which one an implementation chose (order of function argument evaluation pre-C++17; comparing unrelated pointers with `<`, from the deadlock worked example in the OS course). **Implementation-defined:** like unspecified, but the implementation *must* document and consistently apply its choice (e.g., the size of `int`). Precision on this three-way distinction is a genuine, specific signal of depth — most candidates only know "undefined behavior is bad" without the other two categories.

**Why does multiple inheritance get a bad reputation?**
The diamond problem (duplicate base sub-objects, resolved via `virtual` inheritance — file 02) and the general complexity of reasoning about which base's members/methods apply. In practice, multiple inheritance of pure-interface (abstract, no-data) base classes is common and fine; multiple inheritance of classes carrying real state is the genuinely risky pattern most style guides steer away from.

**What is the PIMPL idiom, and why use it?**
"Pointer to IMPLementation" — a class's private implementation details are moved into a separate, forward-declared class, accessed only via a pointer held by the public class. Two real benefits: **compile-time decoupling** (changing the implementation's private members doesn't force every file that includes the public header to recompile, since the header no longer exposes those details) and a cleaner ABI boundary. The real cost: an extra indirection (pointer dereference) and a heap allocation for the impl object — a genuine trade-off, not a free win, worth naming both sides if asked.

**What's the difference between `struct` and `class` in C++?**
Purely default member/inheritance access: `struct` defaults to `public`, `class` defaults to `private`. No other language difference. Convention (not enforced by the compiler) typically uses `struct` for plain data aggregates with no invariants to protect, `class` for types with behavior and encapsulated state.

**Why is `#include` guarding (`#pragma once` or include guards) necessary?**
Without it, including the same header twice in one translation unit (common via transitive includes) redefines every declaration in it — an ODR-style violation caught at compile time for most declarations, producing a "redefinition" error. `#pragma once` (non-standard but universally supported) or traditional `#ifndef`/`#define` guards make a header a no-op on any include past the first, per translation unit.

## Practice signal

These are meant to be answered in 15-30 seconds each, precisely — if you find yourself needing a full paragraph to explain any one of them, that's a signal to go back to the fuller treatment in the file referenced next to it and rebuild the one-sentence version deliberately, rather than trying to compress it on the fly in the room.

---

## Related Modules

- [OS 03 — Deadlocks (unspecified vs. undefined, pointer comparison)](../os-course/os_03_deadlocks.md)
- [Python 07 — Interview Trivia (same format, other language)](../python-course/python_07_interview_common_questions.md)
