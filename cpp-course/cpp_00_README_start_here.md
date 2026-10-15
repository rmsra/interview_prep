# C++ Language Course (C++11 → C++17) — Start Here

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · **C++** · [Python](../python-course/python_00_README_start_here.md)  
**C++ modules:** **Index** · [Language Evolution 11-17](cpp_01_language_evolution_11_to_17.md) · [OOP/Class Design](cpp_02_oop_and_class_design.md) · [RAII/Smart Pointers](cpp_03_raii_and_smart_pointers.md) · [Move Semantics](cpp_04_move_semantics_and_value_categories.md) · [Templates](cpp_05_templates_and_generic_programming.md) · [STL Containers/Iterators](cpp_06_stl_containers_and_iterators.md) · [Concurrency in C++](cpp_07_concurrency_in_cpp.md) · [Exceptions](cpp_08_exceptions_and_error_handling.md) · [Interview Trivia](cpp_09_interview_common_questions.md)

---

This is different from `dsa-course/dsa_01_cpp_setup_and_best_practices.md`, which is a fast interview-speed STL cheat sheet. This course is the language itself — the semantics, the "why," and the standard-by-standard evolution — for the dedicated C++ coding assessment Tanique flagged, and for any "explain this language feature" question in the Development/Debugging rounds.

## Why C++11 → C++17, not C++20+

Your production background (DDDS/CU-UP, DPDK) sits in this range — C++11 is the floor, and C++17 is a reasonable modern ceiling most telecom/systems codebases have actually adopted. C++20 (concepts, coroutines, ranges) is a real risk to bring up unprompted: it signals textbook knowledge disconnected from what you've actually shipped. If asked directly, it's fine to say so plainly and note you're less deep there.

## Priority given your background

You're not learning this cold — you've written production C++ for 7+ years. The risk isn't "do you know C++," it's **precision under a live question**: correctly naming *why* a rule exists (not just that it exists), and being fast with the syntax that doesn't come up daily even when you use the underlying concept constantly.

- **Spend real time:** move semantics (file 04) and RAII/smart pointers (file 03) — these are where "I use `unique_ptr`" and "I can explain why the move constructor didn't fire here" diverge, and it's the single most common deep-dive pairing in a C++ interview.
- **Calibration pass, you know this:** OOP/class design (file 02) — refresh vocabulary (Rule of 5, vtables, slicing) rather than relearn.
- **Genuinely useful, moderate depth:** templates (file 05), STL internals (file 06) — beyond the cheat sheet's "which container," this covers iterator invalidation and complexity guarantees precisely.
- **Keep light, cross-reference instead of duplicating:** concurrency (file 07) — the conceptual depth (memory ordering, atomics) already lives in `os-course/os_02_synchronization_and_concurrency.md`; this file is just the C++-specific API surface on top of that.
- **High-yield for trivia-style questions:** file 09 — the "greatest hits" list interviewers reach for when they want a fast calibration read on your C++ fluency.

## Files

1. [`cpp_01_language_evolution_11_to_17.md`](cpp_01_language_evolution_11_to_17.md) — what each standard actually added, and why it mattered
2. [`cpp_02_oop_and_class_design.md`](cpp_02_oop_and_class_design.md) — Rule of 5/0, virtual functions, vtables, slicing, multiple inheritance
3. [`cpp_03_raii_and_smart_pointers.md`](cpp_03_raii_and_smart_pointers.md) — `unique_ptr`/`shared_ptr`/`weak_ptr`, exception safety
4. [`cpp_04_move_semantics_and_value_categories.md`](cpp_04_move_semantics_and_value_categories.md) — **spend the most time here**
5. [`cpp_05_templates_and_generic_programming.md`](cpp_05_templates_and_generic_programming.md) — SFINAE, variadics, CTAD, `constexpr if`
6. [`cpp_06_stl_containers_and_iterators.md`](cpp_06_stl_containers_and_iterators.md) — invalidation rules, complexity guarantees
7. [`cpp_07_concurrency_in_cpp.md`](cpp_07_concurrency_in_cpp.md) — `std::thread`/`async`/`atomic` API surface, cross-referenced to the OS course
8. [`cpp_08_exceptions_and_error_handling.md`](cpp_08_exceptions_and_error_handling.md) — exception safety guarantees, `noexcept`, RAII's real justification
9. [`cpp_09_interview_common_questions.md`](cpp_09_interview_common_questions.md) — the trivia-cluster questions, answered precisely

## Format

Same as the other courses: **the mechanism**, **why it exists**, **code**, **the gotcha interviewers probe**, **how to answer if asked directly**.
