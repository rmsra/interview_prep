# OOP & Class Design

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** [Index](cpp_00_README_start_here.md) · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · **OOP/Class Design** · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · [Templates](cpp_05_templates_and_generic_programming.md) · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · [Exceptions](cpp_08_exceptions_and_error_handling.md) · [Interview Trivia](cpp_09_interview_common_questions.md)

---

## The Rule of 5 (and Rule of 0) — precisely, not just the list

If a class manages a resource (raw pointer, file handle, etc.), the compiler-generated special member functions are almost never correct, so C++ gives you a rule for which ones to define **together**:

1. Destructor
2. Copy constructor
3. Copy assignment operator
4. Move constructor (C++11)
5. Move assignment operator (C++11)

**Why "together," precisely:** if you define *any* of the resource-managing ones (say, the destructor, to free a raw pointer), the compiler will still generate a copy constructor for you — a shallow, member-wise copy — which now double-frees the same resource when both copies are destroyed. Defining one and leaving the rest to the compiler's defaults is the actual source of most "Rule of 3/5" bugs, not the absence of any special member function at all.

```cpp
class Buffer {
    int* data;
    size_t size;
public:
    Buffer(size_t n) : data(new int[n]), size(n) {}
    ~Buffer() { delete[] data; }                                    // 1
    Buffer(const Buffer& o) : data(new int[o.size]), size(o.size) { // 2
        std::copy(o.data, o.data + size, data);
    }
    Buffer& operator=(const Buffer& o) {                            // 3
        if (this == &o) return *this;
        delete[] data;
        data = new int[o.size]; size = o.size;
        std::copy(o.data, o.data + size, data);
        return *this;
    }
    Buffer(Buffer&& o) noexcept : data(o.data), size(o.size) {      // 4
        o.data = nullptr; o.size = 0;
    }
    Buffer& operator=(Buffer&& o) noexcept {                        // 5
        if (this == &o) return *this;
        delete[] data;
        data = o.data; size = o.size;
        o.data = nullptr; o.size = 0;
        return *this;
    }
};
```

**The Rule of 0 — the modern, preferred answer:** don't manage raw resources yourself at all. Use `std::vector`/`std::unique_ptr`/`std::string` as members instead of raw pointers, and let the compiler-generated special members work correctly by composition (each member's own correct special functions get called automatically). **This is genuinely the answer a senior interviewer wants to hear** — "I'd reach for Rule of 0 by using RAII-wrapping members, and only hand-write the Rule of 5 when I'm implementing the low-level RAII wrapper itself" is a stronger answer than reciting the five functions.

## Virtual Functions & the Vtable

- A **virtual function** enables **dynamic dispatch**: the function actually called is resolved at runtime based on the object's *actual* type, not the static type of the pointer/reference used to call it.
- **Mechanism:** a class with any virtual functions gets a hidden **vtable pointer** (vptr) added to each instance, pointing to a per-class **vtable** — an array of function pointers, one per virtual function. A virtual call is an indirect call through this table (`obj->vptr[i](...)`), not a direct call — this is the real (small, usually negligible) runtime cost virtual functions have over non-virtual ones, worth naming precisely if asked "what's the cost of virtual functions."
- **Virtual destructor — the single most commonly-tested point in this section:** if a class might be deleted through a base-class pointer (`Base* b = new Derived(); delete b;`) and the base destructor is *not* virtual, only `~Base()` runs — `~Derived()` never executes, silently leaking any resources `Derived` owns. **Any class intended to be used polymorphically through a base pointer must have a virtual destructor.** This is a real, common bug class, not just trivia.
- **Pure virtual functions / abstract classes:** `virtual void f() = 0;` — no implementation in this class, and the class becomes abstract (cannot be instantiated). A derived class must override every pure virtual function to become concrete. This is how C++ expresses an interface.

## Object Slicing — the gotcha that trips people who understand virtual functions but forget this

```cpp
void process(Base b) { b.speak(); }   // takes Base BY VALUE
Derived d;
process(d);   // SLICED — only the Base portion of d is copied into b
```

Passing/assigning a derived object **by value** into a base-typed variable copies only the base-class portion — the derived-specific data (and effectively, the dynamic type for dispatch purposes) is silently discarded ("sliced off"). Virtual dispatch doesn't save you here — `b` is genuinely, statically a `Base` object at this point, not a `Derived` wearing a mask. **Fix: pass/store polymorphic types by reference or pointer, never by value.** This single rule (polymorphism requires indirection) is worth stating explicitly if slicing comes up.

## Multiple Inheritance & the Diamond Problem

```cpp
class Animal { public: int legs; };
class Bird : public Animal {};
class Horse : public Animal {};
class Pegasus : public Bird, public Horse {};  // two Animal sub-objects!
```

- Without intervention, `Pegasus` contains **two separate copies** of `Animal` (one via `Bird`, one via `Horse`) — `pegasus.legs` is ambiguous, and must be disambiguated (`pegasus.Bird::legs`).
- **`virtual` inheritance fixes this:** `class Bird : public virtual Animal {}; class Horse : public virtual Animal {};` — now `Pegasus` shares a single `Animal` sub-object. Worth knowing the mechanism exists and the keyword, even though real-world diamond inheritance is rare — most codebases (correctly) avoid multiple inheritance of *state*, reserving multiple inheritance for combining pure-interface (abstract) base classes, which have no data to duplicate and thus no diamond problem in practice.

## `const` Correctness

- `const` on a member function (`void f() const`) promises the function doesn't modify the object's observable state — enables calling it on `const` objects/references, and is a real compiler-enforced contract, not just documentation.
- `mutable` on a member allows it to be modified even inside a `const` member function — the standard escape hatch for things like a cache or a mutex that need to change even during a logically-const operation (e.g., lazy-computing and caching a value inside an otherwise-const getter).
- **Overload resolution on const:** a class can have both a `const` and non-`const` overload of the same-named member function (differing only in `const`-ness) — the compiler picks based on whether the calling object is `const`. Classic example: `operator[]` on `std::vector` has both, so a `const vector&` gives you read-only element access.

## Practice signal

Virtual destructors and object slicing are the two highest-yield "gotcha" questions in this file — both are real bugs people actually write, not just trivia. If asked to design a class hierarchy live, stating "Rule of 0 by composition, virtual destructor since this is meant to be used polymorphically" up front, before writing any code, is a strong opening signal.

---

## Related Modules

- [Python 02 — OOP in Python (contrast: MRO/C3 vs. virtual inheritance)](../python-course/python_02_oop_in_python.md)
