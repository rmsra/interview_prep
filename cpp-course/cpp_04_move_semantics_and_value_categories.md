# Move Semantics & Value Categories (spend the most time here)

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · [OOP/Class Design](cpp_02_oop_and_class_design.md) · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · **Move Semantics** · [Templates](cpp_05_templates_and_generic_programming.md) · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · [Exceptions](cpp_08_exceptions_and_error_handling.md) · [Interview Trivia](cpp_09_interview_common_questions.md)

---

This is the C++ course's equivalent of the OS course's synchronization file — the single topic most likely to separate "uses `std::move`" from "understands what's actually happening." Get the vocabulary exactly right; this is a section where imprecise language reads as a knowledge gap even when the intuition is correct.

## Value Categories — the taxonomy, precisely

Every C++ expression has a **value category**. The two you need cold:

- **lvalue:** has identity, refers to a specific memory location you could take the address of. A named variable is always an lvalue: `x`, `*ptr`, `container[i]`.
- **rvalue** (specifically, **prvalue** — pure rvalue): a temporary, no persistent identity, about to be destroyed. `5`, `x + y`, the return value of a function returning by value, a temporary object `Widget()`.
- **xvalue** ("eXpiring value," the one people forget): an lvalue-like expression that's *about to be destroyed* — the result of `std::move(x)` is the canonical example. It has identity (it's a real object, `x`), but it's marked as safe to pillage because you've explicitly said you're done with it.

**Why this taxonomy matters practically:** overload resolution for move-vs-copy constructors is decided by value category, not by type. `T&&` (rvalue reference) binds to prvalues and xvalues; `const T&` (lvalue reference) binds to lvalues (and can also bind to rvalues, which is *why* copy always works as a fallback even when move doesn't exist). This is the entire mechanism that makes `std::move(x)` cause the *move* constructor to be selected instead of the copy constructor — `std::move` doesn't move anything itself, it just **casts its argument to an xvalue**, which changes which overload gets picked.

```cpp
Widget w1;
Widget w2 = w1;              // w1 is an lvalue -> copy constructor
Widget w3 = std::move(w1);   // std::move(w1) is an xvalue -> move constructor
Widget w4 = Widget();        // Widget() is a prvalue -> move constructor (or elided entirely, see below)
```

## `std::move` — what it actually does (and doesn't)

**`std::move` performs zero moves.** It's purely a cast: `static_cast<T&&>(x)`. Nothing happens at the call site itself — it just makes `x` eligible to be selected by an rvalue-reference overload (a move constructor/assignment) at the next place it's used. **The actual "move" is whatever the selected constructor/assignment operator's body does** — typically stealing a pointer and nulling out the source, but that's implementation, not something `std::move` itself performs. This exact distinction ("`std::move` doesn't move; it enables the *next* function called to move") is worth stating precisely if asked to define it.

**Real gotcha — moving from an object you then use again:**
```cpp
std::vector<int> v = {1,2,3};
auto v2 = std::move(v);
std::cout << v.size();   // legal, but v is in a "valid but unspecified" state — DON'T rely on its contents
```
After being moved from, an object is guaranteed to still be in a valid state (you can destroy it, reassign it, call any function with no preconditions on its value) — but its *contents* are unspecified. Using a moved-from object's value (not just destroying/reassigning it) is a real, common bug.

## `std::forward` & Perfect Forwarding

**The problem `std::forward` solves:** inside a function template, a parameter declared as `T&&` (a **forwarding reference**, sometimes loosely called a "universal reference" — only when `T` is a deduced template parameter, *not* the same thing as an rvalue reference on a concrete type) is itself an **lvalue** once inside the function body, even if an rvalue was passed in — because it now has a name and identity. Forwarding it to another function with plain `std::move` would always move, even when the original caller passed an lvalue that shouldn't be moved from.

```cpp
template<typename T>
void wrapper(T&& arg) {
    inner(std::forward<T>(arg));   // preserves whether the ORIGINAL caller passed an lvalue or rvalue
}
```
`std::forward<T>` conditionally casts back to rvalue **only if `T` was deduced as a non-reference type** (meaning an rvalue was originally passed — template deduction rules collapse `T` to `U&` for an lvalue argument and to `U` for an rvalue argument, which is the actual mechanism `std::forward` inspects). This is what makes a single wrapper function correctly preserve the calling convention (copy vs. move) all the way through to `inner`, regardless of what was originally passed — the textbook use case is a generic factory function like `make_unique`, which needs to forward constructor arguments without forcing an unnecessary copy or an incorrect move.

**The precise one-liner if asked "difference between `std::move` and `std::forward`":** `std::move` unconditionally casts to rvalue; `std::forward` conditionally casts to rvalue, only when the original argument actually was one — `move` is for "I'm done with this, take it"; `forward` is for "pass this exactly as it was given to me."

## Copy Elision & RVO/NRVO

```cpp
Widget makeWidget() {
    return Widget(args);   // prvalue -- C++17: GUARANTEED no copy or move at all
}
Widget makeWidgetNamed() {
    Widget w(args);
    return w;               // NRVO -- typically elided, but only as a PERMITTED optimization, not guaranteed
}
```
- **RVO (Return Value Optimization), unnamed:** constructing a temporary directly in the caller's storage, skipping any copy/move entirely. As of **C++17, this is a language guarantee** for the unnamed case (returning a prvalue directly) — not just an allowed optimization compilers happened to do. Real consequence: a type with **no** accessible copy or move constructor at all can still be returned by value this way in C++17, which was ill-formed pre-17.
- **NRVO (Named RVO):** the same optimization for a *named* local variable being returned — still only a **permitted**, not guaranteed, optimization even in C++17 (compilers do it in the vast majority of real cases, but the standard doesn't require it). Worth knowing this distinction precisely: unnamed = guaranteed since C++17, named = still just usually-happens.
- **Why this matters for interview code:** don't reflexively write `return std::move(w);` for a local variable — this actually *defeats* NRVO in many cases (it turns a potential elision into a guaranteed move, which is strictly worse than elision, since elision skips the operation entirely rather than just making it cheap) and is a genuinely well-known anti-pattern. Plain `return w;` lets the compiler choose the best option (elision, or move as the fallback).

## Move Constructor / Move Assignment — when each actually gets called

- Move constructor: initializing a **new** object from an rvalue (`Widget w2 = std::move(w1);`, or a return-by-value not eligible for elision).
- Move assignment: assigning to an **existing** object from an rvalue (`w2 = std::move(w1);` where `w2` already exists).
- **The implicit-generation rule, worth knowing precisely:** the compiler only auto-generates a move constructor/assignment if the class declares **none** of: destructor, copy constructor, copy assignment, move constructor, move assignment. Declaring *any one* of the Rule-of-5 members suppresses the implicit move operations (they silently fall back to copy instead, which still compiles — no error, just quietly worse performance) — this is a real, common, silent-performance-regression gotcha, not just a rule to recite.

## Practice signal

The three questions that come up constantly, worth being able to answer in one precise sentence each: **"What does `std::move` do?"** (casts to rvalue, moves nothing itself). **"When would you mark a move constructor `noexcept`?"** (see file 03 — `vector` reallocation reason). **"Why not always `return std::move(localVar)`?"** (defeats NRVO, moves are cheap but elision is free). If you can answer these three precisely and explain *why* rather than just *what*, that's usually enough to fully clear this topic in an interview.

---

## Related Modules

- [C++ 03 — RAII/Smart Pointers (noexcept move + vector reallocation)](cpp_03_raii_and_smart_pointers.md)
- [DSA 01 — C++ Setup (interview-speed idioms)](../dsa-course/dsa_01_cpp_setup_and_best_practices.md)
