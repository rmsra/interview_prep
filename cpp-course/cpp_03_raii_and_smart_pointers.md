# RAII & Smart Pointers

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · [OOP/Class Design](cpp_02_oop_and_class_design.md) · **RAII/Smart Pointers** · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · [Templates](cpp_05_templates_and_generic_programming.md) · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · [Exceptions](cpp_08_exceptions_and_error_handling.md) · [Interview Trivia](cpp_09_interview_common_questions.md)

---

## RAII — the idea underneath almost everything else in this file

**Resource Acquisition Is Initialization:** tie a resource's lifetime to an object's scope. Acquire the resource in the constructor, release it in the destructor. Because C++ guarantees destructors run when an object goes out of scope — **including when an exception unwinds the stack past it** — RAII makes resource cleanup automatic and exception-safe without a single `try`/`catch`/`finally`. This is *the* justification for preferring RAII wrappers over manual acquire/release pairs, and it's worth stating exactly this way if asked "why RAII" rather than just "it's convenient."

## `unique_ptr` — exclusive ownership, (almost) zero overhead

```cpp
std::unique_ptr<Widget> w = std::make_unique<Widget>(args);
// automatically deleted when w goes out of scope
std::unique_ptr<Widget> w2 = std::move(w);   // ownership transferred; w is now null
```
- **Cannot be copied** (copy constructor/assignment are deleted) — only moved. This is the type system enforcing "exactly one owner" at compile time, not a runtime check.
- **Zero runtime overhead over a raw pointer** in the default case — no reference count, no control block. The only cost is the destructor call and (by default) a `delete` on destruction.
- **Custom deleters:** `unique_ptr<FILE, decltype(&fclose)> f(fopen(...), &fclose);` — the deleter type is part of `unique_ptr`'s template type, which is why it can be zero-overhead (known at compile time) but also means two `unique_ptr`s with different deleter types aren't the same type.
- **Prefer `make_unique` over `unique_ptr<T>(new T(...))`** — same exception-safety reasoning as `make_shared` below.

## `shared_ptr` — shared ownership via reference counting

```cpp
std::shared_ptr<Widget> s1 = std::make_shared<Widget>(args);
std::shared_ptr<Widget> s2 = s1;   // both share ownership, refcount now 2
// Widget destroyed when the LAST shared_ptr referencing it is destroyed
```
- **Real overhead, worth being able to state precisely:** a control block (heap-allocated) holding the reference count, plus **atomic increment/decrement** on copy/destroy (atomic specifically so `shared_ptr` is safe to copy across threads) — this atomic overhead is the actual cost, not "it's just a pointer with a number attached."
- **`make_shared<T>(args)` vs. `shared_ptr<T>(new T(args))` — a real, concrete difference, not just style:**
  1. **One allocation instead of two** — `make_shared` allocates the control block and the object together in a single block; the raw-`new` form allocates the object, then a separate control block inside the `shared_ptr` constructor.
  2. **Exception safety, precisely:** `foo(shared_ptr<T>(new T()), bar())` has an ordering problem pre-C++17 — if `new T()` succeeds but `bar()` throws before the `shared_ptr` constructor runs, `T` leaks, because the raw pointer isn't yet owned by anything exception-safe. `make_shared` avoids the intermediate raw-pointer state entirely. (C++17 tightened function-argument evaluation order rules, which reduces but doesn't fully eliminate reasons to prefer explicit `new`.)
  - **The one real reason to still use `shared_ptr<T>(new T())` directly:** when you need a custom deleter, or when the combined allocation would keep the object's memory alive longer than necessary (a `weak_ptr` keeps the *control block* alive even after the object itself is destroyed — with `make_shared`'s combined allocation, that means the object's memory isn't actually freed until every `weak_ptr` is gone too, which can matter for a large object with long-lived weak references).

## `weak_ptr` — observing without owning, and breaking cycles

```cpp
std::shared_ptr<Node> parent = std::make_shared<Node>();
std::shared_ptr<Node> child = std::make_shared<Node>();
parent->child = child;
child->parent = parent;   // if this were shared_ptr: CYCLE — neither refcount ever hits 0, both leak forever
// fix: child->parent should be std::weak_ptr<Node>
```
- `weak_ptr` observes an object managed by `shared_ptr` **without** contributing to the reference count — doesn't keep the object alive on its own.
- **Doesn't dereference directly** — must call `.lock()`, which returns a `shared_ptr` (temporarily bumping the refcount, safely) if the object still exists, or an empty/null `shared_ptr` if it's already been destroyed. This is the mechanism that makes it safe: you can't accidentally dereference a dangling reference, because `lock()` forces you to check.
- **The canonical use case, worth having ready:** any parent/child or observer pattern where a cycle would otherwise form — a child holding `shared_ptr` back to its parent is the classic accidental-leak bug, and `weak_ptr` is the textbook fix.

## Exception Safety Guarantees — the vocabulary, precisely

Interviewers sometimes ask you to characterize a function's exception safety, not just "is it safe" — there are three named levels:

| Guarantee | Meaning |
|---|---|
| **Basic guarantee** | if an exception is thrown, no resources leak and the program remains in a *valid* (but possibly changed/unspecified) state — invariants hold, but you don't know exactly what the state is |
| **Strong guarantee** | if an exception is thrown, the operation has **no effect at all** — state is exactly as if the call never happened (commonly implemented via copy-and-swap: build the new state fully in a temporary, then swap in — the swap itself is `noexcept`, so nothing can fail partway through the visible change) |
| **Nothrow guarantee** (`noexcept`) | the operation is guaranteed never to throw at all |

RAII is what makes the **basic guarantee achievable almost for free** in idiomatic C++ — since destructors run during unwinding automatically, resources don't leak even when you haven't specifically engineered for the strong guarantee.

## `noexcept`

- Marks a function as guaranteed not to throw. If it throws anyway, `std::terminate` is called immediately — no stack unwinding, no destructors run past that point. This is a hard contract, not a suggestion.
- **Why it matters for performance, concretely:** `std::vector`'s reallocation logic checks whether a type's move constructor is `noexcept`. If it is, `vector` uses **move** to transfer elements to the new buffer during growth (fast, and safe — a move can't leave the vector in a state where an exception mid-transfer would corrupt it, since nothing can throw). If the move constructor is *not* marked `noexcept`, `vector` conservatively falls back to **copying** elements instead, even though a move constructor exists — because a throwing move partway through reallocation could leave the vector in a corrupted, half-moved state with no way to safely recover. **Concretely: forgetting `noexcept` on a move constructor can silently make your type slower inside `vector` without any compiler error** — a genuinely good, specific gotcha to know cold.

## Practice signal

"When would you use `shared_ptr` over `unique_ptr`" → default to `unique_ptr` unless you *specifically* need shared ownership (genuinely shared lifetime, not just "I don't want to think about who owns this") — `unique_ptr` should be the default reach, `shared_ptr` the deliberate exception, and saying so signals good judgment, not just API knowledge. "Why might two objects referencing each other leak" → reference cycle, `weak_ptr` as the fix. "Why mark a move constructor `noexcept`" → the `vector` reallocation behavior above is the single best concrete answer.

---

## Related Modules

- [Python 04 — Memory Model (refcounting + cycle GC vs. shared_ptr/weak_ptr)](../python-course/python_04_memory_model_and_gil.md)
- [OS 04 — Memory Management](../os-course/os_04_memory_management.md)
