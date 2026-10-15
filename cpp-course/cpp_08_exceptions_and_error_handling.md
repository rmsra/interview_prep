# Exceptions & Error Handling

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · [OOP/Class Design](cpp_02_oop_and_class_design.md) · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · [Templates](cpp_05_templates_and_generic_programming.md) · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · **Exceptions** · [Interview Trivia](cpp_09_interview_common_questions.md)

---

## The Mechanism — throw, unwind, catch

```cpp
try {
    risky();
} catch (const std::invalid_argument& e) {
    // handle
} catch (const std::exception& e) {
    // catch-all for anything deriving std::exception
} catch (...) {
    // catch absolutely anything — last resort, can't inspect what was thrown
}
```
- **Stack unwinding:** when an exception is thrown and no matching `catch` exists in the current scope, the stack unwinds — every fully-constructed local object between the `throw` and the matching `catch` has its **destructor called**, in reverse order of construction. This is the mechanism that makes RAII (file 03) work during error handling: cleanup happens automatically, without a single explicit cleanup call in the code path that actually failed.
- **Catch by `const&`, not by value:** catching by value copies the exception object (usually harmless but wasteful) and, for a polymorphic exception hierarchy, **slices** it the same way any by-value polymorphic assignment does (file 02) — you'd lose the derived-type information a `catch` block further down might need. Catch by reference (`const std::exception&`) avoids both problems.
- **Order matters:** `catch` clauses are checked top-to-bottom, first match wins — a base-class `catch` before a derived-class `catch` means the derived handler is unreachable (and most compilers warn about this).

## Custom Exception Hierarchies

```cpp
class ConfigError : public std::runtime_error {
public:
    explicit ConfigError(const std::string& msg) : std::runtime_error(msg) {}
};
```
Deriving from `std::exception` (or one of its standard subclasses — `std::runtime_error`, `std::logic_error`, etc.) rather than an unrelated type means calling code that only knows about `std::exception` can still catch and log/handle your custom type generically via `catch (const std::exception& e) { e.what(); }` — worth doing even for project-specific exception types, since it keeps a fallback catch-all meaningful.

## `noexcept` — the contract, and why violating it is severe

```cpp
void f() noexcept {
    throw std::runtime_error("oops");   // std::terminate() is called IMMEDIATELY — no unwinding, no destructors run
}
```
Already covered from the `vector`-reallocation performance angle in file 03 — the correctness angle here: a `noexcept` function that throws anyway doesn't propagate the exception at all. `std::terminate` is called on the spot, generally aborting the whole program. This makes `noexcept` a genuine contract you can rely on for reasoning about a function's behavior — not just an optimization hint.

## Exceptions vs. Error Codes vs. `std::optional` — the real trade-off, not just style

| Approach | When it fits |
|---|---|
| **Exceptions** | genuinely *exceptional*, rare-path failures where forcing every caller to check would clutter the common-path code — and where you need to unwind through several call-stack levels to reach a handler that can actually do something about it |
| **Error codes / `errno`-style** | hot-path, performance-critical code where exceptions' (small but real) overhead matters, or interop with C APIs that have no concept of exceptions at all |
| **`std::optional<T>` (C++17)** | a function that can legitimately have "no answer" as a normal outcome (a lookup that might miss) — not an error at all, just an expected possibility; using an exception here would be using exceptions for ordinary control flow, which is a real anti-pattern |

**The precise distinction worth stating:** exceptions should represent *exceptional* circumstances, not expected/routine alternate outcomes — "user not found" in a lookup is routine and belongs in `std::optional` or a return value; "the config file that must exist for the program to run at all is missing" is exceptional and belongs in an exception. Conflating the two (throwing for routine misses, or silently swallowing genuine errors into a sentinel value) is the actual thing interviewers are probing for when they ask "how do you handle errors in C++" — not just "do you know the `try`/`catch` syntax."

## RAII's Real Justification, Restated Precisely (ties files 03 and 08 together)

Given a function can exit through **any** of: normal return, an early `return`, or an exception unwinding through it — manual cleanup code (`free`/`close`/`unlock` written explicitly before every possible exit point) has to be duplicated at every one of those exit points, and it's trivially easy to miss one, especially the exception path, since it's not visible in the normal control-flow reading of the function. RAII collapses all of these exit paths into one mechanism (destructor runs on scope exit, unconditionally, however the scope was exited) — which is precisely why modern C++ treats "manual cleanup with `try`/`finally`-style code" (a pattern common in Java/C#/Python) as a smell in C++ specifically, not a neutral style choice: C++ has a strictly better mechanism available (RAII) that those other languages either lack or added later.

## Practice signal

"How do you handle errors in C++" is a real, common open-ended question — the strong answer distinguishes exceptional-vs-routine outcomes (exceptions vs. `optional`/return values) rather than presenting one mechanism as universally correct. "Why does C++ favor RAII over `try`/`finally`" → C++ doesn't have `finally` at all, specifically *because* RAII covers the same need more reliably (no missed cleanup path, since it's tied to object lifetime rather than manually duplicated at every exit point).

---

## Related Modules

- [C++ 03 — RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md)
