# Templates & Generic Programming

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · [OOP/Class Design](cpp_02_oop_and_class_design.md) · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · **Templates** · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · [Exceptions](cpp_08_exceptions_and_error_handling.md) · [Interview Trivia](cpp_09_interview_common_questions.md)

---

## Function & Class Templates — the basics, fast

```cpp
template<typename T>
T max_of(T a, T b) { return a > b ? a : b; }

template<typename T>
class Box {
    T value;
public:
    explicit Box(T v) : value(v) {}
    T get() const { return value; }
};
```
- Templates are compiled via **instantiation**: the compiler generates a real, concrete function/class for each distinct set of template arguments actually used, at compile time. This is why template code that's never instantiated with a given type never has to compile for that type — and why template errors show up at the *instantiation* site (often deep in a call chain), a genuinely famous pain point ("template error message soup") worth acknowledging if it comes up.
- **Template argument deduction:** for function templates, the compiler infers `T` from the arguments (`max_of(3, 5)` deduces `T=int`) — you don't have to write `max_of<int>(3, 5)` explicitly, though you can when deduction would be ambiguous.

## Template Specialization

```cpp
template<typename T>
struct Describe { static const char* name() { return "unknown"; } };

template<>                                    // full specialization
struct Describe<int> { static const char* name() { return "int"; } };

template<typename T>                          // partial specialization (class templates only)
struct Describe<T*> { static const char* name() { return "pointer"; } };
```
- **Full specialization:** a completely separate implementation for one specific set of template arguments.
- **Partial specialization:** a separate implementation for a *pattern* of arguments (any pointer type, any `std::vector<T>`, etc.) — **only available for class templates**, not function templates (function templates can only be *overloaded*, which achieves a similar practical effect but is a genuinely different mechanism, worth knowing as a precise distinction if asked).

## SFINAE — "Substitution Failure Is Not An Error"

**The core idea:** when the compiler substitutes template arguments to check if a template is viable for a given call, if that substitution produces an invalid type/expression, the compiler doesn't error out — it just **silently removes that template from the overload-resolution candidate set**, as if it had never been written. This is a real language rule, not just a technique, and it's what makes conditional template compilation possible without `if constexpr` (which is the C++17 improvement over this).

```cpp
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
process(T x) { return x * 2; }   // only viable when T is an integral type

template<typename T>
typename std::enable_if<!std::is_integral<T>::value, T>::type
process(T x) { return x; }       // only viable when T is NOT integral
```
`std::enable_if<condition, T>::type` is only a valid type (equal to `T`) when `condition` is true; otherwise it has no `::type` member at all, which makes substitution fail for that overload, which SFINAE then quietly drops rather than erroring. This is genuinely one of the more arcane corners of pre-17 C++ — **worth knowing the concept and being able to recognize the pattern, but `if constexpr` (below) has made hand-written SFINAE much rarer in code written after C++17.**

## `constexpr if` (C++17) — the modern, readable alternative to SFINAE for many cases

```cpp
template<typename T>
auto process(T x) {
    if constexpr (std::is_integral<T>::value) {
        return x * 2;
    } else {
        return x;
    }
}
```
The untaken branch **isn't even compiled** for a given instantiation of `T` — not just skipped at runtime, genuinely not type-checked or compiled for that instantiation. This means code that would be ill-formed for one branch (e.g., calling a method that only exists on some types) is fine, as long as it's inside the branch that's never selected for that particular `T`. This single feature replaced a large fraction of what SFINAE was previously needed for, with vastly more readable code — a good, concrete example of "C++17 made existing things easier," matching file 01's framing.

## Variadic Templates & Fold Expressions

```cpp
template<typename... Args>              // C++11: parameter pack
void log_all(Args... args) {
    (std::cout << ... << args) << '\n'; // C++17: fold expression — collapses the pack with <<
}

// Recursive unpacking — the C++11-only way, before fold expressions existed
template<typename T>
void print(T t) { std::cout << t; }
template<typename T, typename... Rest>
void print(T first, Rest... rest) {
    std::cout << first << ", ";
    print(rest...);                     // recurse on the remaining pack
}
```
- **Parameter pack (C++11):** `Args...` represents zero or more template type parameters; `args...` (lowercase) represents the corresponding function parameters. Before C++17, processing a pack required recursive template instantiation (peel off one argument, recurse on the rest, need a base-case overload to stop) — genuinely verbose.
- **Fold expression (C++17):** `(args + ...)`, `(args && ...)`, etc. — collapses an entire pack with a binary operator in one expression, eliminating most of that recursive boilerplate. This is the mechanism behind `std::make_unique`'s ability to forward an arbitrary number of constructor arguments (combined with `std::forward` from file 04 — variadic templates + perfect forwarding together is exactly how a generic factory function is built).

## CTAD — Class Template Argument Deduction (C++17)

```cpp
std::pair p(1, 2.5);            // deduces std::pair<int, double> — no <> needed
std::vector v{1, 2, 3};         // deduces std::vector<int>
```
Extends the deduction function templates always had to class template constructors. For custom classes, the compiler generates **deduction guides** automatically in the common case; you can also write one explicitly for cases the automatic generation gets wrong (e.g., a constructor taking an iterator pair, where you want the element type deduced from what the iterators point to, not from the iterator type itself).

## Practice signal

"What is SFINAE" and "what's the difference between SFINAE and `constexpr if`" are both fair, common questions — the precise answer to the second is that SFINAE operates by *removing a candidate from overload resolution* when substitution fails (a function-template-level, compile-time-but-indirect mechanism), while `constexpr if` operates by *not compiling a branch at all* within a single function body (more direct, more readable, C++17+). If templates come up as a live-coding topic, a generic `max_of`/`Box`-style function or class template (not full SFINAE) is the realistic bar — full hand-rolled SFINAE is much more likely to be a "have you seen this" discussion than something you're asked to write from scratch live.
