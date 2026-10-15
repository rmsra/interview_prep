# Python 3 Interview Course — Start Here

**Courses:** [DSA](../dsa-course/dsa_00_README_start_here.md) · [Networking](../networking-course/networking_00_README_start_here.md) · [Operating Systems](../os-course/os_00_README_start_here.md) · [C++](../cpp-course/cpp_00_README_start_here.md) · **Python**  
**Python modules:** **Index** · [Core Language Features](python_01_core_language_features.md) · [OOP in Python](python_02_oop_in_python.md) · [Decorators/Generators](python_03_functional_features_and_decorators.md) · [Memory Model/GIL](python_04_memory_model_and_gil.md) · [Concurrency in Python](python_05_concurrency_in_python.md) · [Typing/Modern Python](python_06_typing_and_modern_python.md) · [Interview Trivia](python_07_interview_common_questions.md) · [Testing/Tooling](python_08_testing_and_tooling.md)

---

Python is a **minimum qualification** for this role, listed equally with C++ in the job posting — not a nice-to-have. Your resume already shows heavy practical Python (test automation, log-analysis pipelines, PyTest/CI, NumPy/Pandas), so this course leans toward filling the *language-internals* gaps that don't necessarily surface from using Python as a tool, plus the testing/tooling depth your resume already claims.

## Priority given your background

- **Spend real time:** the GIL and concurrency model (file 04, 05) — this is the single most commonly-probed "do you actually understand Python, or just use it" topic, and it's genuinely different from anything in the C++/OS courses' concurrency material.
- **High-value, moderate depth:** OOP/dunder methods (file 02), decorators/generators/context managers (file 03) — these show up both as trivia questions and as live-coding prompts ("write a decorator that times a function," "implement a context manager").
- **Directly fills a flagged gap:** file 08 (testing/tooling) — this is exactly what was flagged as missing after the resume review a while back. Your PyTest/CI background is real and strong; this file gives you the vocabulary to talk about it precisely.
- **Fast-recall cluster:** file 07, same shape as the C++ course's trivia file.

## Files

1. [`python_01_core_language_features.md`](python_01_core_language_features.md) — data model, mutability, argument passing, comprehensions
2. [`python_02_oop_in_python.md`](python_02_oop_in_python.md) — dunder methods, MRO, duck typing, `@property`
3. [`python_03_functional_features_and_decorators.md`](python_03_functional_features_and_decorators.md) — closures, decorators, generators, context managers
4. [`python_04_memory_model_and_gil.md`](python_04_memory_model_and_gil.md) — refcounting + GC, the GIL precisely, `is` vs `==`
5. [`python_05_concurrency_in_python.md`](python_05_concurrency_in_python.md) — **spend real time here** — threading vs. multiprocessing vs. asyncio
6. [`python_06_typing_and_modern_python.md`](python_06_typing_and_modern_python.md) — type hints, dataclasses, `enum`, `pathlib`
7. [`python_07_interview_common_questions.md`](python_07_interview_common_questions.md) — the trivia cluster
8. [`python_08_testing_and_tooling.md`](python_08_testing_and_tooling.md) — PyTest fixtures/mocking/parametrize, tying directly to your resume

## Format

Same as the rest: **the mechanism**, **why it exists**, **code**, **the gotcha interviewers probe**, **how to answer if asked directly**.
