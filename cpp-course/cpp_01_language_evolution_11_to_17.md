# Language Evolution — C++11 → C++17

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · **Language Evolution 11-17** · [OOP/Class Design](cpp_02_oop_and_class_design.md) · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · [Templates](cpp_05_templates_and_generic_programming.md) · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · [Exceptions](cpp_08_exceptions_and_error_handling.md) · [Interview Trivia](cpp_09_interview_common_questions.md)

---

Knowing *which standard* introduced a feature is a genuine interview signal — it shows you've tracked the language's evolution, not just picked up idioms ambiently. Organized by standard; within each, roughly most-to-least likely to come up.

## C++11 — the big one, treat everything else as building on this

- **`auto`** — compile-time type deduction from the initializer. `auto x = 5;` is `int`. Reduces verbosity, especially with iterators/templates, but can obscure the actual type — a real readability trade-off worth acknowledging if asked "any downsides to `auto`."
- **Range-based for:** `for (auto& x : container)` — sugar over iterator begin/end; use `auto&` (not `auto`) to avoid copying non-trivial elements.
- **Lambdas:** `[capture](params) -> return_type { body }`. Capture by value `[x]`, by reference `[&x]`, or both `[=, &x]`. Compiles to an anonymous functor (a class with `operator()`) — worth knowing that's what's actually happening under the hood, since it explains why lambdas can be stored, copied, and have captured state.
- **Move semantics + rvalue references (`T&&`)** — the single most consequential C++11 feature. Full treatment in file 04; don't shortchange it here.
- **Smart pointers** (`unique_ptr`, `shared_ptr`, `weak_ptr`) — moved into the standard library (had existed as `auto_ptr`, now deprecated/removed, and as Boost extensions before). Full treatment in file 03.
- **`nullptr`** — a real, typed null pointer constant, replacing `NULL`/`0`. Fixes real overload-resolution ambiguity: `f(NULL)` could call `f(int)` instead of `f(char*)` since `NULL` is often just `0`; `f(nullptr)` unambiguously resolves to a pointer overload.
- **`override` and `final`** — `override` on a derived virtual function: compiler verifies it actually overrides a base virtual function (catches signature mismatches — e.g., wrong `const`-ness — that would otherwise silently create an unrelated new virtual function instead of overriding). `final` prevents further overriding (on a method) or further inheritance (on a class). Pure compile-time safety net, zero runtime cost.
- **Uniform initialization / brace init:** `T x{args};` works consistently across aggregates, containers, and custom types (via `std::initializer_list`). Real gotcha: `std::vector<int> v{5};` (one element, value 5) vs. `std::vector<int> v(5);` (five default-constructed elements) — brace-init prefers an `initializer_list` constructor if one exists, which surprises people.
- **Variadic templates:** templates taking an arbitrary number of type parameters (`template<typename... Args>`). The mechanism behind `std::make_unique`, `std::tuple`, and perfect-forwarding wrapper functions. Fuller treatment in file 05.
- **`constexpr`** (basic form): a function/variable that *can* be evaluated at compile time if given compile-time-constant inputs. C++11's version is restrictive (essentially one return statement); C++14 loosens this significantly (see below).
- **`decltype`:** deduces the exact type of an expression, including reference/const qualifiers `auto` would strip — used in generic/template code where the precise type matters, e.g. trailing return types: `auto f(T x) -> decltype(x.member())`.
- **`std::thread`, `std::atomic`, `std::mutex`** — the first standard (portable, non-platform-specific) concurrency primitives. See file 07.

## C++14 — a smaller, "fix and polish" release

- **Generic lambdas:** `[](auto x, auto y) { return x + y; }` — the lambda's parameters can be `auto`, making it effectively a template. Didn't exist in C++11 (lambda parameter types had to be concrete).
- **Return type deduction for normal functions:** `auto f() { return 5; }` — no trailing-return-type gymnastics needed; C++11 required `decltype` tricks or an explicit return type for anything non-trivial.
- **Relaxed `constexpr`:** loops, multiple statements, local variables are now allowed inside a `constexpr` function — C++11's single-return-statement restriction is gone, making `constexpr` genuinely usable for real logic, not just trivial expressions.
- **`std::make_unique`** — the `unique_ptr` equivalent of `make_shared`, oddly missing from C++11 itself. Prefer it over `unique_ptr<T>(new T(...))` for the same exception-safety reason `make_shared` is preferred (see file 03).
- **Variable templates**, **binary literals (`0b101`)**, **digit separators (`1'000'000`)** — minor, occasionally asked as trivia, low priority to memorize deeply.

## C++17 — the one most likely to show up if you claim "modern C++"

- **Structured bindings:** `auto [key, value] = *map_it;` — unpacks a pair/tuple/struct into named variables in one line. Extremely common in real code (range-for over a map: `for (auto& [k, v] : my_map)`), worth having reflexively.
- **`if` with an initializer:** `if (auto it = map.find(key); it != map.end())` — scopes the initialized variable to the `if`/`else` block, avoiding namespace pollution from a variable that's only relevant to one branch. Same pattern exists for `switch`.
- **`std::optional<T>`** — represents "a value, or no value" without the overhead/danger of a sentinel value (like `-1` or `nullptr`) or an out-parameter. Real interview-relevant use: return type for a function that might legitimately fail to produce a value (a lookup that might miss) without throwing an exception or returning a magic constant.
- **`std::variant<Types...>`** — a type-safe tagged union: holds exactly one of a fixed set of alternative types at a time, and access is checked (`std::get` throws `std::bad_variant_access` on a type mismatch, or use `std::visit` for exhaustive, compile-time-checked handling of every alternative). The modern, safe replacement for a C-style `union` + manual type tag.
- **`std::string_view`** — a non-owning, read-only view into a string's character data (pointer + length, no allocation). Avoids unnecessary copies when a function only needs to *read* string data, not own or mutate it — worth knowing precisely: `string_view` doesn't guarantee null-termination and must never outlive the string it views (a classic dangling-view bug if you return one referencing a temporary).
- **Class Template Argument Deduction (CTAD):** `std::pair p(1, 2.0);` instead of `std::pair<int, double> p(1, 2.0);` — the compiler deduces template arguments from constructor arguments, the same way it already did for function templates.
- **Fold expressions:** `(args + ...)` inside a variadic template — collapses a parameter pack with an operator in one expression, replacing what previously needed recursive template instantiation. Fuller treatment in file 05.
- **`constexpr if`:** `if constexpr (condition)` inside a template — the untaken branch isn't even compiled for a given instantiation, enabling compile-time branching in generic code without SFINAE tricks. Fuller treatment in file 05.
- **`std::filesystem`** — standardized, portable path/file operations (finally, without reaching for Boost or platform-specific APIs). Lower interview priority — worth knowing it exists, not worth deep syntax recall.
- **Mandatory copy elision (guaranteed RVO)** for specific cases (returning a prvalue directly) — this became a *language guarantee*, not just a permitted optimization. Real consequence: some code patterns that "worked by accident" pre-17 (e.g., returning a type with no copy/move constructor at all) are now guaranteed to work. Ties directly into file 04.

## Practice signal

"What's new in C++17" or "have you used any C++17 features" is a genuinely common opener — `std::optional`, structured bindings, and `if`-with-initializer are the three most likely to have shown up in your own recent code and are the safest, most concrete answers. If asked to contrast C++11 vs. C++17 in one sentence: **C++11 changed what the language could express (move semantics, lambdas); C++17 mostly made expressing things you already could do more concise and safer** (structured bindings, `optional`/`variant` instead of sentinels/unions).
